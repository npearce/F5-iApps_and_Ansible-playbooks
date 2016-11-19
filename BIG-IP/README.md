For the BIG-IP examples we are going to perform the following tasks. Make sure your environment is up and running first as per the instructions in ../README.md.

#Exercise 1 - First contact
Ansible likes to reach out to devices. It wants to SSH to everything. However, there's this cool new thing called REST API's. So, what we are going to do it tell Ansible that the host is 'localhost', and then get localhost to make REST calls for us. So, the first exercise is  

##Part 1
Lets start with a simple exercise to explain all the parts. In Exercise 1 - Part 1, we're going to perform a basic REST API 'GET'.

Take a look at "BIG-IP/01-bigip-list_service_templates-part1.yml"

This part performs the request:
`    uri:
      url: https://{{inventory_hostname}}/mgmt/tm/cloud/templates/iapp/
      validate_certs: no
      user: admin
      password: password
      return_content: yes
`
This part stores the response in 'iapps_list':
`    register: iapps_list
`

And this part tells Ansible its looking at JSON, and to print out the contents of the JSON array items:
`    debug: msg="{{(iapps_list.content|from_json)["items"]}}"
`

###NOTES
`url:` This is the BIG-IP REST resource that Ansible is going to communicate with. `inventory_hostname` just inserts the address for the 'current device' that Ansible is working on. Its one of Ansible's built-in variables.

`validate_certs` refers to whether it should get upset about our default, self-signed certs we're using in the lab BIG-IP.

`[User/Password]` for the BIG-IP REST access.

`return_content` means it should care about the response (versus throwing the request into a black hole and walking away).

`register` collects the response and stores it in `iapps_list`


##Part 2
Part 2 perfoms the same action as Part1. However, it introduces some new themes along the lines of 'doing thing better'.
Take a look at "BIG-IP/01-bigip-list_service_templates-part2.yml"

Two new themes here:
1) Username and Password and now variables passed in to the playbook at the time of execution:
`user: "{{ username }}"
password: "{{ password }}"`

2) Instead of just printing out the entire response, as we did in Part 1, we are now checking the response for a specific item. Saving the admin a bit of effort.

2a) First we use a `when: [something] in` which says that if you find a 'something' in the response, perform this action. In this case, the 'something' i s'f5.http', and the action to perform if its found is to print a debug message.
`debug: msg="'f5.http' iApp found"
when: '"f5.http" in (iapps_list.content|from_json)["items"]'
`
2b) The second evaluation we are performing against the response it to check if the response 'DOES NOT' include something. Notice the `not` in the `when:` statement below.  
`debug: msg="'f5.http' iApp not found"
when: '"f5.http" not in (iapps_list.content|from_json)["items"]'
`

Both the 2a) and 2b) examples can come in use depending on what you want to achieve. More on these later.

##Part 3
In "BIG-IP/01-bigip-list_service_templates-part3.yml" we're adding one more variable, `{{ appsvcs_ver }}`. By doing this, we no longer have the [something] hard coded into the playbook. Instead we can pass the `{{ appsvcs_ver }}` in at execution time. As you playbooks get longer, and more complex, this becomes increasibgly important. Note how many times `{{ appsvcs_ver }}` is referenced in this single playbook task:

`  - name: Check {{ appsvcs_ver }} IS in response
    debug: msg="{{ appsvcs_ver }} found"
    when: '"{{ appsvcs_ver }}" in (iapps_list.content|from_json)["items"]'
`

#Exercise 2 - Request an auth token
Maybe you want to use auth tokens, maybe you don't. Either way they are great for demonstrating a simple REST API POST. Taking a look at 'BIG-IP/02-bigip-get_auth_token.yml' you will notice a few new options:

`      method: POST
      body: { "username":"{{ username }}", "password":"{{ password }}", "loginProviderName":"tmos" }
      body_format: json
`

`method:` we need to say its a HTTP POST

`body:` because a POST requires data (we are 'POSTing' to a resource after all).

`body_format:` because we need to tell it 'what' we are sending. For the BIG-IP API we are posting JSON

Another change you may have noticed, we are being more specific with the object in the JSON response. First we print out the 'entire' response with:

`  - name: Full output
    debug: msg={{ (auth_token.content|from_json) }}
`

Next, we provide an example of just printing the 'token' using:

`  - name: Return the token
    debug: msg={{(auth_token.content|from_json)["token"]["token"]}}
`

Note the reference to `["token"]["token"]`. I fyou look at the structure of the JSON payload from the 'Full output', you will see how the JSON structure reflects this.


#Exercise 3 - Install an iApp onto a BIG-IP
In this playbook we are going to install and iApp (service template) onto BIG-IP, after first verifying it is not there, and then checking that the install was successful, if required, afterwards. Take a look at: 'BIG-IP/03-bigip-install_iapp.yml'

We've already covered the retrieving an iApp list from the BIG-IP and checking the response in Exercise 1.

Moving on to the `- name: Perform the installation!`, we introduce a new 'core module' named `get_url:`. Not the same as the previously used `url:`!!

`get_url:` has a source and destination allowing us to fetch something from a remote HTTP URL and push that object to our BIG-IP:
`url: "{{ appsvcs_source }}"
dest: /var/tmp/
`

If you read the notes at the beginning of 'BIG-IP/03-bigip-install_iapp.yml' you will see that we provide an example for `{{ appsvcs_source }}` which is an iApp on GitHub.

Immediatley following this we execute the `tmsh` (BIG-IPs TMOS Shell) command: `shell: "tmsh load /sys application template /var/tmp/{{ appsvcs_file }}"`

Following the `tmsh` installation command, we then verify that the newly installed iApp is present, which should be familiar to you.

Finally, we perform a cleanup by removing the iApp that we placed into `/var/tmp` using the `get_url:` command earlier.

#Exercise 4 - Deploying an L4 - L7 Service - Part 1
In 'BIG-IP/02-bigip-get_auth_token.yml' we introduced how to perform a REST POST. In that example we had a very simple 'POST body:' to deal with. But deploy an entire L4 - L7 service policy requires quite a bit more data. Take a look at `BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml`.

In exercise 02, we made the short JSON body into a string and escaped the necessary characters to keep Ansible happy. Now, imagine trying to do that to `example_data/minimum_json_post.json`. NO THANKS!!! So, here's tip to save you a lot of time! First, you advertise for an intern... Just kidding. No, you take the JSON payload and you navigate to http://www.json2yaml.com. Paste it in and 'hey presto' you've got the YAML version of the same body.

Why convert it to YAML? A couple of reasons:
* You don't have to escape a billion characters in your playbook to preserve the JSON formatting.
* Ansible is clever enough to convert it to JSON when it performs the POST because we specified `body_format: json`. Thank you, Ansible!

*NOTE: Before running this playbook, take a look at the comments at the top. I listed the variables that his template expects at run-time (with examples). If you don't provide these it will not work.*

Now, if you've being paying attention, you will have noticed that we have a problem with the pool members in the playbook. It has support for two pool members by default, and a third if you uncomment it from the playbook. Well, this isn't good enough. We need to be a little more dynamic in how many pool members can be entered. We solve this problem in Part 2, below!

#Exercise 4 - Deploying an L4 - L7 Service - Part 2
You've probably noticed that we've run into a problem with the pool members. In the Playbook 'BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml' we have room for two pool members which are fed in using the `pool_member_1:` and `pool_member_2:` variables. But what if you have 3 pool members, or more? We need the quantity of pool members to be dynamic and not determined by the playbook.

Enter, templates! Not to be confused with the Ansible Tower 'Job Templates', Ansible templates support building configuratoins using the Jinja2.

So, lets move the `body:` from the playbook and put that into a template named `deploy.j2`.

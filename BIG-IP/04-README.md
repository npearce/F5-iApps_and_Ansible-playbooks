
#Exercise 4 - Part 1 - Deploying an L4 - L7 Service
In 'BIG-IP/02-bigip-get_auth_token.yml' we introduced how to perform a REST POST. In that example we had a very simple 'POST body:' to deal with, which only included three values. However, when deploying an entire L4 - L7 service policy, this requires quite a bit more data. Take a look at `BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml` - https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml

In exercise 02, we made the short JSON body into a string and escaped the necessary characters to keep Ansible happy. This is what it looked like:
`{ "username":"{{ username }}", "password":"{{ password }}", "loginProviderName":"tmos" }`

Now, imagine working through the formatting for `example_data/minimum_json_post.json` - https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/example_data/minimum_json_post.json. NO THANKS!!! So, here's tip to save you a lot of time!

1. Advertise for an intern... Just kidding.

No, you take the JSON payload and you navigate to http://www.json2yaml.com. Paste it in and 'hey presto' you've got a YAML formatted version of the same JSON data. This looks like `example_data/minimum_yaml_post.yml` - https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/example_data/minimum_yaml_post.yml

Why convert it to YAML? A couple of reasons:
* You don't have to escape a billion characters in your playbook to preserve the JSON formatting.
* Ansible is clever enough to convert it to JSON when it performs the POST when it sees `body_format: json`. Thank you, Ansible!!

*NOTE: Before running this playbook, take a look at the comments at the top. I listed the variables that his template expects at run-time (with examples). If you don't provide these it will not work.*

##Create a new Job Template
1. Navigate to the 'Job Templates' section.
2. Click the green '+ADD' button.
3. Enter the name *myTemplate-04*.
4. Leave the default job type as 'Run'.
5. For 'Inventory', click the magnifying glass. From the window that appears, select 'myInventory' and click 'save'.
6. For 'Project', click the magnifying glass and select 'myProject'.
7. For 'Playbook', select 'BIG-IP/03-bigip-install_iapp.yml'
8. For 'Machine Credential' click the magnifying glass and select 'bigips'.
9. Leave verbosity set as '0 (Normal)'.
10. Remember to add the required variables in again, referenced in the comments at the top of the playbook (for example):
`username: admin
password: admin
service_name: myHTTP_Service_1
virtual_ip_address: 10.128.10.21
pool_member_1: 10.128.20.1
pool_member_2: 10.128.20.2`

11. Click 'Save'.
12. Scroll to the bottom of the page and click the 'Rocket Ship' icon next to *myTemplate-04*.

Now, if you've being paying attention, you will have noticed that we have a problem with the pool members in the playbook. It has support for two pool members by default, and a third if you uncomment it from the playbook. This is not automated enough, having to manually comment/uncomment playbooks. We need to be a little more dynamic in how many pool members can be entered. We solve this problem in Part 2, below!

#Exercise 4 - Part 2 - Deploying an L4 - L7 Service
The pool members problem. In the Playbook `BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml` we have room for two pool members which are fed in using the `pool_member_1:` and `pool_member_2:` variables. But what if you have 3 pool members, or more? We need the quantity of pool members to be dynamic and not determined by the playbook.

Enter, templates! Not to be confused with the Ansible Tower 'Job Templates', Ansible templates support building configurations using Jinja2. Take a look at `BIG-IP/template_deploy.j2`. For the most part, it looks like the YAML formatted payload from the previous exercise... until you scroll down to the pool member section. Take a look at the neat Jinja2 loop:

`{% for host in groups.Web_Servers %}
    - - '0'
      - '{{ host }}'
      - '80'
      - '0'
      - '1'
      - '0'
      - enabled
      - ''
{% endfor %}`

This is taking a look at the Ansible Tower 'Inventory', where I created an Invetory Group named `Web_Servers` in which I entered the IP addresses of some web servers. The Jinja2 loop above will iterate through the list for each web server.

So, how to we execute this nifty template feature? There's a new command in `BIG-IP/04-bigip-deploy_L4-L7_service-part2.yml` that we haven't used before:
`    - name: build the JSON payload
      template: src={{ playbook_dir }}/BIG-IP/template_deploy.j2 dest=/var/lib/awx/projects/_6__myproject/BIG-IP/deploy_payload.yml
`
The template si executed which then creates the `deploy_payload.yml` at the time the playbook is executed. Yay, dynamic JSON body creation!

Now, take a look at `body:` in `BIG-IP/04-bigip-deploy_L4-L7_service-part2.yml`. We've replaced many lines of YAML with:
`body: "{{ (lookup('template','{{ playbook_dir }}/BIG-IP/deploy_payload.yml') | from_yaml) }}"`

The `from_yaml` is just telling the playbook its YAML. Otherwise it just see a large string of nonesense. By telling Anislbe its YAML, then it knows it can convert it to JSON (because of `body_format: json`).


##Create a new Job Template
1. Navigate to the 'Job Templates' section.
2. Click on our existing template *myTemplate-04*.
3. For 'Playbook', select 'BIG-IP/04-bigip-deploy_L4-L7_service-part2.yml'
4. As per the notes at the beginning of the playbook, you must provide some variables, for example:
`username: admin
password: admin
service_name: myHTTP_Service_2
virtual_ip_address: 10.128.10.22
`
5. Click 'Save'.
6. *IMPORTANT* The pool members are now sourced via an Inventory Group. So, navigate to 'Inventories'
7. Click 'myInventory'.
8. Click the green 'Add Group' button.
9. Enter 'Web_Servers' as the group name (this is used by the template).
10. For 'source' select 'Manual'.
11. Click 'Save'
12. Click on the newly created 'Web_Servers' group.
13. Click 'Add Host'.
14. Enter the IP Address of the first host, for example, '10.128.20.1', and click 'Save'.
15. Repeat step 14. for each of the required pool members.
16. Navigate to 'Job Templates' and select *myTemplate-04* from the list.
17. The pool_member_x variables are no longer required. You can delete them.
18. This is a good playbook for enabling 'Prompt on Launch' under 'Extra Variables'.
19. Click 'Save'.
20. Scroll to the bottom of the page and click the 'Rocket Ship' icon next to *myTemplate-04*.

#Exercise 5 - Start experimenting with templates
We're done. Hopefully that's a good intro for you. Let me know if you want to understand more.

I recommend you start experimenting with 'template_deploy.j2' - located here:  https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/template_deploy.j2 - if you want to start customizing more of the service templates options. As you can see, there are many more options to tweak!

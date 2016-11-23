
#Exercise 4 - Deploying an L4 - L7 Service - Part 1
In 'BIG-IP/02-bigip-get_auth_token.yml' we introduced how to perform a REST POST. In that example we had a very simple 'POST body:' to deal with. But deploy an entire L4 - L7 service policy requires quite a bit more data. Take a look at `BIG-IP/04-bigip-deploy_L4-L7_service-part1.yml`.

In exercise 02, we made the short JSON body into a string and escaped the necessary characters to keep Ansible happy. Now, imagine trying to do that to `example_data/minimum_json_post.json`. NO THANKS!!! So, here's tip to save you a lot of time! First, you advertise for an intern... Just kidding. No, you take the JSON payload and you navigate to http://www.json2yaml.com. Paste it in and 'hey presto' you've got the YAML version of the same body.

Why convert it to YAML? A couple of reasons:
* You don't have to escape a billion characters in your playbook to preserve the JSON formatting.
* Ansible is clever enough to convert it to JSON when it performs the POST because we specified `body_format: json`. Thank you, Ansible!

*NOTE: Before running this playbook, take a look at the comments at the top. I listed the variables that his template expects at run-time (with examples). If you don't provide these it will not work.*

Now, if you've being paying attention, you will have noticed that we have a problem with the pool members in the playbook. It has support for two pool members by default, and a third if you uncomment it from the playbook. Well, this isn't good enough. We need to be a little more dynamic in how many pool members can be entered. We solve this problem in Part 2, below!

#Exercise 4 - Deploying an L4 - L7 Service - Part 2
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
      template: src=/var/lib/awx/projects/_6__myproject/BIG-IP/template_deploy.j2 dest=/var/lib/awx/projects/_6__myproject/BIG-IP/deploy_payload.yml
`
The template si executed which then creates the `deploy_payload.yml` at the time the playbook is executed. Yay, dynamic JSON body creation!

Now, take a look at `body:` in `BIG-IP/04-bigip-deploy_L4-L7_service-part2.yml`. We've replaced many lines of YAML with:
`body: "{{ (lookup('template','/var/lib/awx/projects/_6__myproject/BIG-IP/deploy_payload.yml') | from_yaml) }}"`

The `from_yaml` is just telling the playbook its YAML. Otherwise it just see a large string of nonesense. By telling Anislbe its YAML, then it knows it can convert it to JSON (because of `body_format: json`).

#Exercise 4 - Put the kettle on
We're done. Hopefully that's a good intro for you. Let me know if you want to understand more.

Interest in further reading? Taking a look at the awesome work my colleague, Tim Rupp, is doing here:
https://github.com/F5Networks/f5-ansible

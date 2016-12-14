#Exercise 2 - Part 1 - Request an auth token
Maybe you want to use auth tokens, maybe you don't. Either way we're going to use the Auth Token to demonstrate a simple REST API POST. Take a look at 'BIG-IP/02-bigip-get_auth_token.yml': https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/02-bigip-get_auth_token.yml

You will notice a few new options:

```
  method: POST
  body: { "username":"{{ username }}", "password":"{{ password }}", "loginProviderName":"tmos" }
  body_format: json
```

`method:` we need to say its a HTTP POST

`body:` because a POST requires data (we are 'POSTing' to a resource after all).

`body_format:` because we need to tell it 'what' we are sending.

Another change you may have noticed, we are being more specific with the object in the JSON response. First we print out the 'entire' response with:

```  
- name: Full output
  debug: msg={{ (auth_token.content|from_json) }}
```

Next, we provide an example of just selecting the 'token' object from the JSON response using:

```
- name: Return the token
  debug: msg={{(auth_token.content|from_json)["token"]["token"]}}
```

Note the reference to '["token"]["token"]'. If you look at the structure of the JSON payload from the 'Full output', you will see how the JSON structure reflects this, and how the reference to '["token"]["token"]' gets the correct data.


##Create a new Job Template
1. Navigate to the 'Job Templates' section.
2. Click the green '+ADD' button.
3. Enter the name **myTemplate-02**.
4. Leave the default job type as 'Run'.
5. For 'Inventory', click the magnifying glass. From the window that appears, select 'myInventory' and click 'save'.
6. For 'Project', click the magnifying glass and select 'myProject'.
7. For 'Playbook', select 'BIG-IP/02-bigip-get_auth_token-part1.yml'
8. For 'Machine Credential' click the magnifying glass and select 'bigips'.
9. Leave verbosity set as '0 (Normal)'.
10. Remember to add the username and password variables in again, as per the notes at the top of the playbook.
11. Click 'Save'.
12. Scroll to the bottom of the page and click the 'Rocket Ship' icon next to **myTemplate-02**.

##Review
Note the difference between the 'TASK [Full output]' and 'TASK [Return the token]'.

#Exercise 2 - Part 2 - Using an auth token
Well, now we asked for an Auth Token we may as well use it!
In this playbook we are going to use the 'request auth token' taks and then perform a second action that uses the Auth Token (instead of username and password) to extend the default auth token timeout.

##Switch to the 'part2' playbook:
1. Navigate to 'Job Templates'.
2. Click on our existing 'myTemplate-02'.
3. For playbook, select 'BIG-IP/02-bigip-get_auth_token-part2.yml'
4. Scroll down to 'Extra Variables'.
5. After the 'username: and password:' variables added earlier, on a new line, add `token_timeout: 36000`. It should now look like:

```
---
username: admin
password: admin
token_timeout: 36000
```

6. Click 'Save'
7. Scroll down and execute the Job Template (Rocket Ship icon) next to: 'myTemplate-01'

Note in the playbook 'BIG-IP/02-bigip-get_auth_token-part2.yml' that the 'Increase token timeout' play does NOT have username or password. This is because the Auth Token is providing our authentication. This is achieved using:

`HEADER_X-F5-Auth-Token: "{{ (auth_token.content|from_json)['token']['token'] }}"`

**Optional exercise** You can remove the 'Full Output' play from the playbook. I just left that in so you could see where the token comes from.

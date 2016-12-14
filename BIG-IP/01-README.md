Before attempting the BIG-IP Exercises below make sure your environment is up and running first as per the instructions in the Setup README.md:

https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/README.md

#Exercise 1 - Part 1 - First contact
For 'Exercise 1' we will create a basic Ansible Tower Job that will perform a simple REST 'GET'.

Ansible likes to reach out to devices. It wants to SSH to everything. However, there's this cool new thing called REST API's. So, what we are going to do is tell Ansible that the host is 'localhost', and then get localhost to make REST calls for us. So, the first exercise is  

##Create a Job Template
1. Navigate to the 'Job Templates' section.
2. Click the green '+ADD' button.
3. Enter the name **myTemplate-01**.
4. Leave the default job type as 'Run'.
5. For 'Inventory', click the magnifying glass. From the window that appears, select 'myInventory' and click 'save'.
6. For 'Project', click the magnifying glass and select 'myProject'.
7. For 'Playbook', select 'BIG-IP/01-bigip-list_service_templates-part1.yml'
8. For 'Machine Credential' click the magnifying glass and select 'bigips'.
9. Leave verbosity set as '0 (Normal)'.
10. Click 'Save'.
11. Scroll to the bottom of the page and click the 'Rocket Ship' icon next to **myTemplate-01**.

NOTE: If you re-load the page and it still says 'Running' in the top right-hand corner, then wait a moment longer before re-loading again.

What happened? Failed, right? unless your BIG-IP had the password 'password', then this playbook will fail. Lets work through fixing this.

##Review/Fix the playbook
###Review the playbook
First lets review the working parts of the playbook:

https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/01-bigip-list_service_templates-part1.yml

This part performs the request:
```
uri:
   url: https://{{inventory_hostname}}/mgmt/tm/cloud/templates/iapp/
   validate_certs: no
   user: admin
   password: password
   return_content: yes
```
This part stores the response in 'iapps_list':

`  register: iapps_list`

And this part tells Ansible its looking at JSON, and to print out the contents of the JSON array items:

`  debug: msg="{{(iapps_list.content|from_json)["items"]}}"`

*NOTES*

`url:` This is the BIG-IP REST resource that Ansible is going to communicate with. `inventory_hostname` just inserts the address for the 'current device' that Ansible is working on. Its one of Ansible's built-in variables.

`validate_certs` refers to whether it should get upset about our default, self-signed certs we're using in the lab BIG-IP.

`[User/Password]` for the BIG-IP REST access.

`return_content` means it should care about the response (versus throwing the request into a black hole and walking away).

`register` collects the response and stores it in `iapps_list`

###Fix the playbook
To fix the playbook you will need a console login to the Ansible Tower host.
1. Navigate to '/var/lib/awx/projects/'
2. Find your project (based on its directory name). Ansible will pre-pend a number to the project name. For example, `_22_myproject`. Note also that it is all lowercase.
3. Navigate to the BIG-IP subdirectory within your project directory.
4. From here you can open/edit '01-bigip-list_service_templates-part1.yml'. NOTE: the default Centos 7 install has both 'vi' and 'nano' for editing.
5. Change the value for 'password' to your BIG-IP password. This will be the same credentials you use to login to the BIG-IP GUI.
6. Back to the Ansible Tower UI, re-run the playbook (click the 'Rocket Ship' again).

Success will result in a list of the services templates (F5 iApps) installed on the BIG-IP.

**Optional exercise:**
In the Job Template, change the 'verbosity' level from "0 (normal)" to "3 (debug)" and run the playbook again. Review the additional data.

#Exercise 1 – Part 2 - Using variables to avoid playbook edits
Part 2 performs the same action as Part1. However, it introduces some new themes.

Take a look at 'BIG-IP/01-bigip-list_service_templates-part2.yml': https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/01-bigip-list_service_templates-part2.yml

NOTE: pay attention to the comments at the top of the playbook file. It says you need to use two variables in order to execute this playbook.

Reading the playbook you will see that the Username and Password and now variables passed in to the playbook at the time of execution:

```
user: "{{ username }}"
password: "{{ password }}"
```

In addition to using variables for the username and password, we have introduced some intelligence into how we handle the response. In Part 1 we printed out the entire response (the whole list of iApps). In Part 2, instead of just printing out everything we are now checking the response for a specific item.

In the playbook we have two examples of doing this. A 'positive' check (see if it IS there) and a 'negative' check (see if it is NOT there). These will come in handy later.

**Positive check:** First we use a 'when: [something] in' which says that if you find a 'something' in the response, perform this action. In this case, the 'something' is 'f5.http', and the action to perform, if it is found, is to print a debug message.

```
debug: msg="'f5.http' iApp found"
when: '"f5.http" in (iapps_list.content|from_json)["items"]'
```

**Negative check:** The second evaluation we are performing against the response it to check if the response 'DOES NOT' include something. Notice the 'not' in the 'when:' statement below.  

```
debug: msg="'f5.http' iApp not found"
when: '"f5.http" not in (iapps_list.content|from_json)["items"]'
```

More on these later...

##Using the variable playbook:
1. Navigate to 'Job Templates'.
2. Click on our existing 'myTemplate-01'.
3. For playbook, select 'BIG-IP/01-bigip-list_service_templates-part2.yml'
4. Scroll down to 'Extra Variables'.
5. After the `---`, on new lines, enter the "key: value" pairs (assuming your username is 'admin' and password is 'admin'):

```
username: admin
password: admin
```

6. Click 'Save'
7. Scroll down and execute the Job Template (Rocket Ship icon) next to: 'myTemplate-01'

You should see a far more concise response.


#Exercise 1 – Part 3 - More variables
In "BIG-IP/01-bigip-list_service_templates-part3.yml" we're combining a variable to the BIG-IP response handling, '{{ appsvcs_ver }}'. By doing this, we no longer have the [something] of 'Part 2' hard-coded into the playbook. Instead we can pass the '{{ appsvcs_ver }}' in at execution time. As your playbooks get longer, and more complex, these shortcuts become increasingly important. Note how many times '{{ appsvcs_ver }}' is referenced in this single playbook task:

```
- name: Check {{ appsvcs_ver }} IS in response
  debug: msg="{{ appsvcs_ver }} found"
  when: '"{{ appsvcs_ver }}" in (iapps_list.content|from_json)["items"]'
```

You will note, once again, the comments at the top of this playbook informing you which variables are expected for execution:
https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/01-bigip-list_service_templates-part3.yml

We must now add the variable: 'appsvcs_ver'

##Switch playbooks and add the variable
1. Navigate to the 'myTemplate-01' Job Template
2. From the playbook list, select: "BIG-IP/01-bigip-list_service_templates-part3.yml"
3. Scroll down to 'Extra Variables' and, after username and password, enter: "appsvcs_ver: appsvcs_integration_v2.0_001".
4. Check "Prompt on launch" under the "Extra Variables" box.
5. Click 'Save'
6. Execute the Job (Rocket Ship).

Note how 'Prompt on launch' enables the administrator to edit the variables just prior to execution.

Unless you had already manually installed the 'appsvcs_integration_v2.0_001-001.tmpl' from GitHub, it will report 'Note found'. Installing this template is covered in "Exercise 4".

#Exercise 1 - Part 4 - Handling appsvcs_integration versions
Thus far we have been specifying an exact iApp version, which is fine as a one-off but, now its time to handle versions better. Handling this now will make life easier later.

##Switch playbooks and edit the variables
1. Navigate to the 'myTemplate-01' Job Template
2. From the playbook list, select: "BIG-IP/01-bigip-list_service_templates-part4.yml"
3. Scroll down to 'Extra Variables' and replace the current entries with the following:

```
username: admin
password: admin
appsvcs_major: "2.0"
appsvcs_pres: "001"
appsvcs_ver: appsvcs_integration_v{{ appsvcs_major }}_{{ appsvcs_pres }}
```

4. Click 'Save'
5. Execute the Job (Rocket Ship).

**Important**
An iApp name contains version numbers for both the implementation layer (major and minor version) and the presentation layer. When we retrieve a list of iApps from the BIG-IP the implementation layer minor version is omitted. For example, if you installed the iApp "appsvcs_integration_v2.0-002_001.tmpl", the JSON response when retrieving a list of iApps would show "appsvcs_integration_v2.0_001".
For more on appsvcs_integration versions please read: http://appsvcs-integration-iapp.readthedocs.io/en/latest/design.html#versioning

Unless you had already installed the iApp manually, the playbook will come back `"msg": "'appsvcs_integration_v2.0_001' not found"` but note that it correctly assembled the version number variables to produce the correct name 'appsvcs_integration_v2.0_001'. We'll use this more later!

*Optional exercise:*
Run the same playbook again but providing an iApp we know exists. For example, "f5.http". This should will report 'f5.http found'. Note that we did no re-write any of the playbook to achieve this.

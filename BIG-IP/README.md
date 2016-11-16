For the BIG-IP examples we are going to perform the following tasks. Make sure your environment is up and running first as per the instructions in ../README.md.

#Exercise 1 - First contact
Ansible likes to reach out to devices. It wants to SSH to everything. However, there's this cool new thing called REST API's. So, what we are going to do it tell Ansible that the host is 'localhost', and then get localhost to make REST calls for us. So, the first exercise is  

#Exercise 2 - Import the L4-L7 iApp playbook from Github
Import the F5 L4-L7 Playbook from Github.com into Ansible:

1. In Ansible Tower, navigate to the 'Project' tab.
2. Create a new project named 'F5 iApp Playbook'.
3. Select the SCM type 'Git'
4. Enter the SCM URL `http://github.com/`
5. Click 'Save'
6. Click the download (icon with a cloud) button next to the 'F5 Playbooks' project.

#Exercise 3
In the playbook,
https://raw.githubusercontent.com/npearce/F5-iWorkflow_Ansible-playbooks/master/appsvcs_integration_v2.0-001_001.tmpl




#NOTE:

https://10.0.0.167/mgmt/tm/cloud/templates/iapp/?$select='name'&$filter='appsvcs_integration'


Parsing JSON output:
http://justin-hennessy.blogspot.com/2014/11/parsing-json-with-ansible.html

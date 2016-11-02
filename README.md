#About
Adding L4-L7 Services to Ansible playbooks.

Many third party integrations with F5 BIG-IPs deliver only basic L3-L4 functionality, sush as adding a pool member, or disabling a node. What they lack are the advnaced L7 services that the majority of F5 customers deploy. This Github repository will guide Ansible users through the steps involved to automate the deliver of F5's advanced L4-L7 services.

Themes introduced in the following exercised.

1. Service Templates: F5 Service Templates, branded F5 iApps, automate the configuration of advanced L4-L7 functionality. Before you ask, migrating the complexity of an F5 iApp into an Ansible playbook would take years. However, deploying an iApp from an Ansible playbook means the Ansible admin can deliver advanced L4-L7 services without the requirement for F5 domain-specific knowledge. This is the power of abstraction when Ansible playbooks and F5 iApps are combined.


#Environment
The Ansible Playbooks in this repository were developed using the following environment:

* Ansible Tower 3.0.2 (running on CentOS 7)
* iWorkflow v2.0.1
* BIG-IP 12.1.1

NOTE: Default install of CentOS 7 (VM w/ 4GB RAM) using the CentOS 7 DVD ISO.

For install instructions refer to the 'Ansible Tower Quick Installation Guide v3.0.3': http://docs.ansible.com/ansible-tower/latest/html/quickinstall/index.html

#Getting started
##Ansible Tower
Once up and running, unless you're an Ansible Tower pro already, I *highly* recommend you go through the "Ansible Tower Quick Setup Guide v3.0.3" before attempting Exercise 1:
http://docs.ansible.com/ansible-tower/3.0.2/html/quickstart/index.html

#Exercise 1
Import the F5 L4-L7 Playbook from Github.com into Ansible:

1. In Ansible Tower, navigate to the 'Project' tab.
2. Create a new project named 'F5 iApp Playbook'.
3. Select the SCM type 'Git'
4. Enter the SCM URL `http://github.com/`
5. Click 'Save'
6. Click the download (icon with a cloud) button next to the 'F5 Playbooks' project.

#Exercise 2
In the playbook,
https://raw.githubusercontent.com/npearce/F5-iWorkflow_Ansible-playbooks/master/appsvcs_integration_v2.0-001_001.tmpl



#Exercise 1


#Exercise 1

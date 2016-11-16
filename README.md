#About
Adding L4-L7 Services to Ansible playbooks.

Many third party integrations with F5 BIG-IPs deliver only basic L3-L4 functionality, sush as adding a pool member, or disabling a node. What they lack are the advnaced L7 services that the majority of F5 customers deploy. This Github repository will guide Ansible users through the steps involved to automate the deliver of F5's advanced L4-L7 services.

Themes introduced in the following exercised.

*Service Templates*: F5 Service Templates, branded F5 iApps, automate the configuration of advanced L4-L7 functionality. Before you ask, migrating the complexity of an F5 iApp into an Ansible playbook would take years. However, deploying an iApp from an Ansible playbook means the Ansible admin can deliver advanced L4-L7 services without the requirement for F5 domain-specific knowledge. This is the power of abstraction when Ansible playbooks and F5 iApps are combined.

*Strict Updates*: By default, F5 Service Templates (iApps) enable 'strict updates'. Strict updates prevent modification to BIG-IP objects outside of the service template. For example, if an administrator deployed a configuration using the template `Front-end_HTML5_App_Type_1c`, the objects created by the template–the virtual servers, pools, profiles, health-check monitors, etc–would only be editable via the template. The administrator cannot directly modify the template-created BIG-IP objects outside of the template. This is important for preserving the source-of-truth in a declarative model. For more on that topic refer to: [link to REDtalks episode w/ H. Patel]

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

##Where do things go?
Ansible's getting started documentation is very minimal (it assumes you already know their file/directory structure).

##Get the core modules
The install instructions gave me an outdated EPEL library.

1. Create a new Project using the SCM 'git': https://github.com/ansible/ansible-modules-core


##Create a project
1. Create a new directory under `/var/lib/awx/projects/`  (in this project we will use 'myProject')


#Exercise 1 - Add a BIG-IP to the inventory
1. Navigate to the Ansible Tower 'Inventories' section.
2. Select the 'Demo' inventory.
3. On the right-hand side, click the 'Add Host' button.
4. Enter the hostname or ip address of the BIG-IP management interface. For my lab this is: `10.0.0.167`)
5. Click 'Save'


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


#Exercise 1

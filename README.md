#About
Deploying L4-L7 Services policies from Ansible playbooks.

Many third party integrations with F5 BIG-IPs deliver only basic L3-L4 functionality, sush as adding a pool member, or disabling a node. What they lack are the advanced L4 - L7 Services that the majority of F5 customers deploy. This Github repository will guide you through an introduction to Ansible Tower and how the steps involved to automate the delivery of F5 BIG-IP advanced L4-L7 services.

Themes introduced in the following exercises:

**Service Templates**:
F5 Service Templates, branded F5 iApps, automate the configuration of advanced L4-L7 functionality. Before you ask, migrating the complexity of an F5 iApp into an Ansible playbook would take years. However, deploying an iApp from an Ansible playbook means the Ansible admin can deliver advanced L4-L7 services without the requirement for F5 domain-specific knowledge. This is the power of abstraction when Ansible playbooks and F5 iApps are combined.

**Strict Updates**:
By default, F5 Service Templates (iApps) enable 'strict updates'. Strict updates prevent modification to BIG-IP objects outside of the service template. For example, if an administrator deployed a configuration using the template `Front-end_HTML5_App_Type_1c`, the objects created by the template–the virtual servers, pools, profiles, health-check monitors, etc–would only be editable via the template. The administrator cannot directly modify the template-created BIG-IP objects outside of the template. This is important for preserving the source-of-truth in a declarative model.

For more on that topic (Imperative vs Declarative models) refer to:

https://redtalks.live/2016/11/10/redtalks-08-hitesh-on-imperative-vs-declarative-and-sandwiches/

#Environment
The Ansible playbooks in this repository were developed using the following environment:

* Ansible Tower 3.0.3 (running on CentOS 7)
* iWorkflow v2.1 (to be released Feb 2017 - playbooks will be made available then)
* BIG-IP 12.1.0

NOTE: Default install of CentOS 7 (w/ 4GB RAM) using the CentOS 7 DVD ISO.

For install instructions refer to the 'Ansible Tower Quick Installation Guide v3.0.3':

http://docs.ansible.com/ansible-tower/latest/html/quickinstall/index.html

#Getting started
**IMPORTANT** install the 'Ansible Bundle'. When I registered for a trail license, I was sent a link to the Ansible Tower download. This assumes you know Ansible well enough to get it working with a pre-installed Ansible environment. Its not straight forward if you are new to Ansible. However, if you install the Ansible *Bundle* it will install both Ansible and Ansible Tower together and they will work immediately.

This is the Bundle that works straight away:

https://releases.ansible.com/awx/setup-bundle/ansible-tower-setup-bundle-latest.el7.tar.gz

##Ansible Tower
Once up and running, unless you're an Ansible Tower pro already, I *highly* recommend you go through the "Ansible Tower Quick Setup Guide v3.0.3" before attempting Exercise 1. This guide will walk you through all the parts of Ansible Tower:

http://docs.ansible.com/ansible-tower/3.0.3/html/quickstart/index.html

##Where do things go?
Ansible's getting started documentation is minimal (it assumes you already know their file/directory structure). For this exercise, everything of importance goes in:
`/var/lib/awx/projects`

##Setup the Ansible environment.
Login to Ansible Tower using the credentials you provided during installation and work through the following steps. These will be required to continue through the lab exercises.

##Step 1 - Add a BIG-IP to the inventory
1. Navigate to the Ansible Tower 'Inventories' section.
2. Click the green '+ADD' to create a new inventory.
3. Name your inventory, e.g. 'myInventory', and click 'Save'.
4. In the new window that appears, click '+ADD GROUP'
5. Name your group 'bigips' (this will be references by the playbooks), and select the source 'Manual'. Click 'Save'.
6. On the left-hand side, click on the newly created 'bigips' group, then click the '+ADD HOST' button to the right.
7. Enter the hostname (assuming your Ansible-Tower serup has DNS configured) or IP Address of the BIG-IP management interface. For bigip1.n8lab.local we are using its management IP Address: `10.128.1.128`.
8. Click 'Save'
9. At the top of the window you should now see the inventory hierarchy as: "INVENTORIES \ MYINVENTORY \ BIGIPS \ 10.128.1.128"
10. [optional] - you can add more BIG-IP hosts to the 'bigips' group.

##Step 2 - Add some credentials for accessing the BIG-IP
1. Navigate to 'settings' (small cog at the top right).
2. Click on 'Credentials'
3. Click the green '+ADD' button.
4. Enter a name, e.g. 'bigips'
5. Select the type 'Machine'
6. For this lab we enter the username 'root' and password 'admin'.
7. Click 'save'.

NOTE: You can leave the password blank and check 'Ask at runtime?', if you prefer. For details on how Ansible handles passwords refer to:

http://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html

##Step 3 - Import the L4-L7 iApp playbooks from Github
1. In Ansible Tower, navigate to the 'Project' tab.
2. Click the green '+ADD' button.
3. Enter the name 'myProject'.
4. Select the SCM type 'Git'
5. Enter the SCM URL `https://github.com/npearce/F5-iApps_and_Ansible-playbooks`. Leave the 'SCM update options' unchecked for now.
6. Click 'Save'
7. Scroll down to the 'Projects' pane at the bottom of the 'Projects' page.
8. Next to 'myProject', click the download icon (cloud with a downward pointing arrow).
9. Navigate to the 'Jobs' section and you should see 'myProject' at the top of the list.
10. Click on 'myProject' to verify its status reads "Successful". NOTE: you may have to reload the page if its not yet complete.

##Step 4 - Create a Job Template
1. Navigate to the 'Job Templates' section.
2. Click the green '+ADD' button.
3. Enter the name 'myTestTemplate'.
4. Leave the default job type as 'Run'.
5. For 'Inventory', click the magnifying glass. From the window that appears, select 'myInventory' and click 'save'.
6. For 'Project', click the magnifying glass and select 'myProject'. This will allow you to use the playbooks downloaded from GitHub in 'Step 3'.
7. For 'Playbook', select the first entry '00-hello_world.yml' NOTE: the other playbooks that were downloaded. We'll get to them later.
8. For 'Machine Credential' click the magnifying glass and select 'bigips'.
9. Leave verbosity set as '0 (Normal)'.
10. Click 'Save'.

##Step 5 - Run your first Ansible Tower Job
1. Navigate to 'Job Templates'.
2. Scroll down to the 'Job Templates' list.
3. Next to 'myTestTemplate', click the Rocket Ship icon (to execute the playbook).
4. You will be presented a new window showing 'Results' and 'Standard Out'.
5. If its not already finished, wait a few seconds and then reload the page.
6. If everything worked you should see "Successful" in the top right, and out of the "Hello World!" playbook on the right.

If 'Step 5' failed then something must be wrong with your Ansible config. Maybe try a re-install?
If successful, navigate to the 01-README.md in the ./BIG-IP subdirectory and continue to "Exercise 1"

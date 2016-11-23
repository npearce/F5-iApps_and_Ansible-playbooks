
#Exercise 3 - Install an iApp onto a BIG-IP
In this playbook we are going to install and iApp (service template) onto BIG-IP, after first verifying it is not there, and then checking that the install was successful, if required, afterwards. Take a look at: 'BIG-IP/03-bigip-install_iapp.yml' - https://github.com/npearce/F5-iApps_and_Ansible-playbooks/blob/master/BIG-IP/03-bigip-install_iapp.yml

We've already covered retrieving an iApp list from the BIG-IP and checking the response in Exercise 1.

Reviewing the playbook, in the section '- name: Perform the installation!', we introduce a new 'core module' 'named get_url:'. Not the same as the previously used 'url:'!! 'url:' supports a destination, which is great for REST API actions. 'get_url:' supports both a source and destination, which is great for transferring files.

`get_url:` has a source and destination allowing us to fetch something from a remote HTTP URL and push that object to our BIG-IP:

```
url: "{{ appsvcs_source }}"
dest: /var/tmp/
```

If you read the notes at the beginning of 'BIG-IP/03-bigip-install_iapp.yml' you will see that we provide an example for '{{ appsvcs_source }}' which is an iApp on GitHub.

Immediately following this we execute the 'tmsh' (BIG-IPs TMOS Shell) command:

`shell: "tmsh load /sys application template /var/tmp/{{ appsvcs_file }}"`

Following the 'tmsh' installation command, we then verify that the newly installed iApp is present, which should be familiar to you. *NOTE:* previous interactions have been via HTTP. However, the Ansible 'shell:' command drops into SSH. This is when the Ansible 'Credentials' are used, which we set at the beginning before Exercise 1.

Finally, we perform a cleanup by removing the iApp that we placed into '/var/tmp' using the 'get_url:' command earlier.

##Create a new Job Template
1. Navigate to the 'Job Templates' section.
2. Click the green '+ADD' button.
3. Enter the name **myTemplate-03**.
4. Leave the default job type as 'Run'.
5. For 'Inventory', click the magnifying glass. From the window that appears, select 'myInventory' and click 'save'.
6. For 'Project', click the magnifying glass and select 'myProject'.
7. For 'Playbook', select 'BIG-IP/03-bigip-install_iapp.yml'
8. For 'Machine Credential' click the magnifying glass and select 'bigips'.
9. Leave verbosity set as '0 (Normal)'.
10. Remember to add the required variables in again, referenced in the comments at the top of the playbook:

```
username:
password:
appsvcs_file:
appsvcs_source:
appsvcs_ver:
```


11. Click 'Save'.
12. Scroll to the bottom of the page and click the 'Rocket Ship' icon next to **myTemplate-03**.

##Review
If you now login to the BIG-IP you will see a new iApp installed. This particular Ansible playbook has a lot of interesting debug information. I suggest you edit the Job Template and run it again, for reference.

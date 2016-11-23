
#Exercise 3 - Install an iApp onto a BIG-IP
In this playbook we are going to install and iApp (service template) onto BIG-IP, after first verifying it is not there, and then checking that the install was successful, if required, afterwards. Take a look at: 'BIG-IP/03-bigip-install_iapp.yml'

We've already covered the retrieving an iApp list from the BIG-IP and checking the response in Exercise 1.

Moving on to the `- name: Perform the installation!`, we introduce a new 'core module' named `get_url:`. Not the same as the previously used `url:`!!

`get_url:` has a source and destination allowing us to fetch something from a remote HTTP URL and push that object to our BIG-IP:
`url: "{{ appsvcs_source }}"
dest: /var/tmp/
`

If you read the notes at the beginning of 'BIG-IP/03-bigip-install_iapp.yml' you will see that we provide an example for `{{ appsvcs_source }}` which is an iApp on GitHub.

Immediately following this we execute the `tmsh` (BIG-IPs TMOS Shell) command: `shell: "tmsh load /sys application template /var/tmp/{{ appsvcs_file }}"`

Following the `tmsh` installation command, we then verify that the newly installed iApp is present, which should be familiar to you.

Finally, we perform a cleanup by removing the iApp that we placed into `/var/tmp` using the `get_url:` command earlier.

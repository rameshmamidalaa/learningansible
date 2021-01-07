# learningansible
References:
https://www.linkedin.com/learning/red-hat-certified-engineer-ex294-cert-prep-1-foundations-of-ansible
https://www.linkedin.com/learning/red-hat-certified-engineer-ex294-cert-prep-2-using-ansible-playbooks
https://www.linkedin.com/learning/red-hat-certified-engineer-ex294-cert-prep-3-managing-systems-with-ansible

# Some example Ad-hoc commands.
  Ansible Syntax: ansible [pattern] -m [module] -a "[module options]"
  For Example:
# Testing Connectivity to Nodes: ansible all -m ping -i inventory/hosts_vm
  ansible webservers -m shell -a 'echo $DISPLAY'
  ansible webservers -a "/sbin/reboot"  please note that here -a - arguments are in double quotes (""), why because it is not a variable, if it's a variable use in single quotes ('') as mentioned in the first example.
# Creating an empty file in the remote host.
  ansible rhhost2* -m file -a "path=/home/user1/tmp/file1.txt state=touch mode=700"
# Copying a file in the remote host if required with the content. In the following command used "force=no" means it always create the new file instead of overriding the existing file in case it it exists.
  ansible rhhost2* -m copy -a "dest=/home/user1/tmp/file2.txt content='stuff' force=no mode=700"

  ansible rhhost2* -m file -a "dest=/home/user1/tmp/file2.txt mode=600"
  ansible rhhost2* -m file -a "dest=/home/user1/tmp/file2.txt mode=600 owner=root group=root" -b -K (Where -b means becomes: to elevate the privileges & -K means --ask-become-pass to prompt for the password).
#  "--ask-become-pass": If the remote user needs to provide a password in order to run sudo commands, you can include the option --ask-become-pass to your Ansible command. This will prompt you to provide the remote user sudo password:
# To create a directory.
  ansible rhhost2* -m file -a "dest=/home/user1/tmp/newdir mode=755 state=directory"
# To delete a directory.
  ansible rhhost2* -m file -a "dest=/home/user1/tmp/newdir mode=755 state=directory state=absent"
# To manage the software packages (For Example: in RHEL OS family).
  ansible rhhost2* -m yum -a "name=httpd state=present" -b -K (Where -b means becomes: to elevate the privileges & -K means --ask-become-pass to prompt for the password).
# If it is Ubuntu: ansible rhhost2* -m apt -a "name=apahe2 state=present" -b -K
# To install the latest required software package:
  ansible rhhost2* -m yum -a "name=httpd state=latest" -b -K
# To install the nginx web server
  ansible rhhost2* -m apt -a "name=nginx state=present" -b -K
# To remove the installed package:
  ansible rhhost2* -m yum -a "name=httpd state=absent" -b -K
# Restart servers and services:
  ansible localhost -m service -a "name=nginx state=started" -b -K
  ansible localhost -m service -a "name=nginx state=restarted" -b -K
  ansible localhost -m service -a "name=nginx state=reloaded" -b -K
  ansible localhost -m service -a "name=nginx state=stopped" -b -K
  ansible localhost -m service -a "name=nginx enabled=yes" -b -K
  ansible localhost -m service -a "name=nginx enabled=yes" --check  To check if the service enabled or not.
  ansible localhost -m shell -a "systemctl status nginx" To check the status of nginx server using 'shell' module.
# To add the new user and add it to the group
  ansible rhhost2* -m user -a "name=user2 state=present home=/home/user2 shell=/bin/bash" -b -K
  ansible rhhost2* -m user -a "name=user2 group=wheel" -b -K
  ansible rhhost2* -m user -a "name=user2 group=user2 groups=wheel" -b -K
  Create the encrypted password use following cmd: doveadm pw -s SHA512-CRYPT  Then copy the pwd use it in the below cmd:
  ansible rhhost2* -m user -a 'name=user3 state=present home=/home/user3 shell=/bin/bash password=xxxxxxxxxxxxx' -b -K
# To remove the user:
  ansible rhhost2* -m user -a "name=user2 state=absent" -b -K
  ansible rhhost2* -m user -a "name=user2 state=absent group=user2 groups=wheel" -b -K
# To create & delete the group:
  ansible rhhost2* -m group -a "name=user2 state=present" -b -K
  ansible rhhost2* -m group -a "name=user2 state=absent" -b -K
# Gather Data: Gathering information from the host in our inventory is best left to playbooks, but some of the power can also be had using ad hoc commands.
  ansible rhhost2* -m setup
  ansible rhhost2* -m setup -a 'gather_subset=network'  Gather specific information.
  ansible rhhost2* -m setup -a 'gather_subset=!network,hardware'
  ansible rhhost2* -m setup -a 'gather_subset=!all,min' To get minimum information.
  ansible rhhost2* -m setup -a 'gather_subset=!all,!min'
  ansible rhhost2* -m setup -a 'gather_subset=!all,!min,hardware'
  ansible rhhost2* -m setup -a 'gather_subset=!all,!min,hardware filter=ansible_mounts' filter it for more specific information.
# Dump the information into the specific location.
  ansible rhhost2* -m setup --tree /tmp/facts
# Verify playbooks using ansible-lint.
  ansible-lint apache.yml
  ansible-playbook --syntax-check webservices.yml
  ansible-playbook --check webservices.yml
# Structure of the project: It's a good time to start planning for the future. To get our Ansible configuration scheme to scale, we need to create a structure that separates group variables, roles, tasks, and templates. This structure will allow us to grow our configuration setup beyond where we are now (simple playbook in one directory).Each directory included must have a main.yml file in it.
├── apache.yml
├── group_vars
├── inventory
│   └── hosts_vm
├── roles
│   ├── common
│   │   ├── defaults          - Contains defualt variables for the role.
│   │   ├── files             - Contains files that can be deployed by the role.
│   │   ├── handlers          - Contains change handlers used by this role.
│   │   ├── meta              - Contains meta data (role dependencies) for the role.
│   │   ├── tasks             - Contains the main list of tasks to be executed by the role.
│   │   ├── templates         - Contains templates for the role.
│   │   └── vars              - Contains non-defualt variables for the role.
│   ├── dbservers
│   │   ├── defaults
│   │   ├── files
│   │   ├── handlers
│   │   ├── meta
│   │   ├── tasks
│   │   ├── templates
│   │   └── vars
│   └── webservers
│       ├── defaults
│       ├── files
│       ├── handlers
│       ├── meta
│       ├── tasks
│       ├── templates
│       └── vars

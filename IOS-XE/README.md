# Cisco IOS XE Ansible playbooks examples

A collection of simple Ansible playbooks to configure a Cisco Catalyst device running IOS-XE

Playbooks have been tested with Ansible 2.3 on a Catalyst 3850 running IOS XE 16.5.1 and Catalyst 9300 running ISO XE 16.6.2 

# Installation

To install latest version of Ansible on servers running the most popular Linux distributions like Red Hat, CentOS, Fedora, Debian, or Ubuntu, the OS package manager can be used.

Installation via the Python package manager (pip), is available as well.

All the Cisco IOS XE modules are included in Ansible Core so no aditional effort is required to begin automating your Cisco IOS XE devices. As Ansible has an agentless architecture, once username and password are configured/provided, then the devices can be managed through Ansible. The username provided with the playbook must have the requisite role privilege to allow device configuration changes.

Fedora, CentOS, RedHat:
Installation using yum package manager:

~~~~
$sudo yum install ansible
~~~~

## Installation using pip:

~~~~
$sudo pip install ansible
~~~~

In case pip is not installed on the server yet, it can be installed using the yum package manager again:

~~~~
$ sudo yum install python-pip
~~~~

Debian, Ubuntu:
Installation using apt-get package manager:

~~~~
$ sudo apt-get install ansible
~~~~

## Installation using pip:

~~~~
$ sudo pip install ansible
~~~~
In case pip is not installed on the server yet, it can be installed using the apt-get package manager again:

~~~~
$ sudo apt-get install python-pip
~~~~

OSX:
pip is the recommended installation method on OSX.

~~~~
$ sudo pip install ansible
~~~~

In case pip is not installed on the Mac yet, it can be installed using the easy_install Python Tool:
~~~~
$ sudo easy_install pip
~~~~

# Ansible version
After installation, on any Linux distro, you can check the Ansible version installed.

~~~~
$ansible --version
ansible 2.3.0.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.5 (default, Nov  6 2016, 00:28:07) [GCC 4.8.5 20150623 (Red Hat 4.8.5-11)]
~~~~

# Hosts File
The host file is where the devices under management are listed. A single device can be in a single group or included in multiple groups. In the below hosts file we have a single group called ios-xe, which has 3 devices, cat9k1, cat9k2 and cat9k3. The connection is set to local as we will be connecting SSH to manage the devices.

~~~~
$ cat /etc/ansible/hosts
[all:vars]
ansible_connection = local
ansible_python_interpreter=python

[ios-xe]
cat9k1
cat9k2
cat9k3
~~~~

# Authentication
When speaking with remote machines, SSH keys are encouraged but username/passwords can be used as well. SSH Keys are a little more complicated to confiure and their setup is not included in this Quick Start Guide.

There are several ways to provide username/password in Ansible:

## hosts File
Username can be defined in the hosts file and used to run any playbook:
~~~~
$ cat /etc/ansible/hosts
[all:vars]
ansible_connection = local
username=cisco
password=cisco
~~~~

NOTE: this method has been deprecated and will be removed in a future version

## Variables File

Username and password can also be defined in an external file and imported in the playbook:
~~~~
$ cat external_vars.yml
---
username: cisco
password: cisco
auth_pass: cisco
transport: cli
~~~~
~~~~
$ cat ios_command.yaml
- name: test Ansible ios_command on Cisco IOS XE
  hosts: ios-xe
  connection: local
  gather_facts: no

  vars_files:
    - external_vars.yml
    
~~~~
NOTE: this method has been deprecated and will be removed in a future version

## Environment Variables
An easy way to provide username and password is by defining the ANSIBLE_NET_USERNAME and ANSIBLE_NET_PASSWORD environment variables

~~~~
export ANSIBLE_NET_USERNAME=cisco
export ANSIBLE_NET_PASSWORD=cisco
~~~~
This method is very usefull in case the same username and password are used in all the device.

## Ansible command line
Another method is to provide username and password at run time like in the example below:

~~~~
$ ansible-playbook ios_banner.yaml -u admin -k
SSH password:
~~~~

## Ansible Vault
Vault is a feature of ansible that allows keeping sensitive data such as passwords or keys in encrypted files, rather than as plaintext in your playbooks, roles or at runtime. These vault files can then be distributed or placed in source control. Plese refer to the official Ansible Vault documentation.

# Connectivity
Let us first establish that we have basic connectivity and can login to the devices. We will use the built-in ping module to complete this test. This is not an ICMP ping, but a module to test IP conenctivity, and that the correct login details are provided.

~~~~
$ ansible leaf -m ping
cat9k1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
cat9k2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
cat9k3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
~~~~

# Documentation
Thorough documentation for all Cisco IOS XE modules can be found on the Ansible website or alternatively from the terminal, by utilizing the inbuilt documentation tool.

~~~~
$ ansible-doc ios_vrf
> IOS_VRF    (/Library/Python/2.7/site-packages/ansible/modules/network/ios/ios_vrf.py)

  This module provides declarative management of VRF definitions on
  Cisco IOS devices.  It allows playbooks to manage individual or the
  entire VRF collection.  It also supports purging VRF definitions
  from the configuration that are not explicitly defined.
<snip>
~~~~

# Example Playbook
In this initial playbook we will provision a number of vrfs across all devices. We will use the Ansible module called ios_vrf to automate this task.

~~~~
---

- name: test Ansible with IOS XE vrfs
  hosts: ios-xe
  connection: local
  gather_facts: no

  tasks:
    - name: configure a given list of vfrs and purge all the others
      ios_vrf:
        vrfs:
            - red
            - blue
            - yellow
~~~~

As with all YAML file, the playbook starts with 3 dashes.

hosts denotes the host or group of hosts that will have the tasks, executed against and tasks are the the modules/tasks that are to be run. To make sure only the given the list of vrfs are configured on the devices, a "purge" option is provided as well.

# Running a Playbook
Assuming the above playbook is called ios_vrf.yaml, this task can then be run from a terminal window. By default, Ansible will use the hosts file located in /etc/ansible/hosts however a different hosts file can be specified using the -i flag at runtime or define it in the ansible.cfg file. In the below example we will use the local hosts file defined using the latter option.

~~~~
$ ansible-playbook ios_vrf.yaml -u admin -k
SSH password:

PLAY [test Ansible with IOS XE vrfs] *******************************************

TASK [configure a given list of vfrs and purge all the others] *****************
changed: [cat9k1]
changed: [cat9k2]
changed: [cat9k3]

PLAY RECAP *********************************************************************
cat9k1             : ok=1    changed=1    unreachable=0    failed=0
cat9k2             : ok=1    changed=1    unreachable=0    failed=0
cat9k3             : ok=1    changed=1    unreachable=0    failed=0
~~~~

# Debugging
In case of errors, the playbook cab be run with the verbose option like in the example below.

~~~~
$ ansible-playbook ios_vrf.yaml -vvvv -u admin -k
Using /Users/fabrimac/MyAnsible/cisco-ansible/IOS-XE/ansible.cfg as config file
SSH password:
Loading callback plugin default of type stdout, v2.0 from /Library/Python/2.7/site-packages/ansible/plugins/callback/__init__.pyc

PLAYBOOK: ios_vrf.yaml *********************************************************
1 plays in ios_vrf.yaml

PLAY [test Ansible with IOS XE vrfs] *******************************************
META: ran handlers

TASK [configure a given list of vfrs and purge all the others] *****************
task path: /Users/fabrimac/MyAnsible/cisco-ansible/IOS-XE/ios_vrf.yaml:32
The higher the number of v the more verbose output you get.
~~~~

Ansible also has built-in support for logging. The log file can be defined either in the ansible.cfg file or by defining the ANSIBLE_LOG_PATH environment variable

~~~~
$ cat ansible.cfg
[defaults] 
log_path=/var/log/ansible.log

$ export ANSIBLE_LOG_PATH=/var/log/ansible.log
~~~~

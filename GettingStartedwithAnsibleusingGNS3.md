# EXOS: Getting Started with Ansible using GNS3

## Sections:
* [Pre-requisites](#pre-requisites)
* [Setting up GNS3 with EXOS-VM](#setting-up-gns3-with-exos-vm)
* [Setting up GNS3 with Network Automation Docker Container](#setting-up-gns3-with-network-automation-docker-container)
* [Setting up a Test Topology](#setting-up-a-test-topology)
	- [EXOS Device Setup](#exos-device-setup)
	- [Network Automation Container Setup](#network-automation-container-setup)
* [System Setup](#system-setup)
	- [Installing Ansible](#installing-ansible)
	- [Setting up the Virtual Environment](#setting-up-the-virtual-environment)
	- [Configuring the Local hosts file](#configuring-the-local-hosts-file)
	- [Creating an Ansible configuration file](#creating-an-ansible-configuration-file)
	- [Creating and configuring the Ansible inventory file](#creating-and-configuring-the-ansible-inventory-file)
	- [Creating and configuring the Variable file](#creating-and-configuring-the-variable-file)
* [Testing the Setup](#testing-the-setup)


## Pre-requisites

- GNS3 installed on any supported OS
- GNS3 VM installed on any supported Hypervisor
- Virtual EXOS Image

## Setting up GNS3 with EXOS-VM

1. Download the EXOS appliance template from [GNS3 Marketplace]
2. Download the required qcow2 file from our [GitHub Website]
3. Import the downloaded template in GNS3 and load the qcow2 file downloaded in step 2. Visit this [link] for details
4. Drag the installed appliance to the workspace and start it up
5. Run some show commands to make sure it is working. For example ```show switch``` and ```show version```
>NOTE: Default credentials to login to the device is "admin"/"". Password is blank

## Setting up GNS3 with Network Automation Docker Container

1. Download the Network Automation template from [GNS3 Marketplace]
2. Import the downloaded template in GNS3 and perform the basic installation
3. Once done, the "Network Automation" Container can be seen under the "End Devices" list
4. Drag the docker container to the workspace and GNS3 will automatically pull the required files from its repository over the Internet

> By default, the Network Automation docker container includes Python 2.7, NAPALM, pyntc, Netmiko and Ansible. So you do not need to install these separately.

## Setting up a Test Topology

Prepare the topology as show in the below image in GNS3 Workspace:
> NOTE: **Do not** Power-on the topology at this point

![Topology](http://i68.tinypic.com/5v56ch.png "Exos_Topology")

### EXOS Device Setup

Power-up any **one** of the EXOSVM devices from the above topology and perform the below configuration:

1. Configure user-defined IP address for Default VLAN interface
```sh
EXOS-VM # unconfigure vlan Default ipaddress
EXOS-VM # configure vlan Default ipaddress 192.168.122.100/24
```

2. Create an admin user and enable SSH

```sh
EXOS-VM # create account admin xtrm_user xtrm_pass
EXOS-VM # enable SSH2
```

### Network Automation Container Setup

1. Before Powering-on the container, right click on it and select "Edit config"
2. Enable DHCP by removing the hash '#' before the DHCP config items
3. Below is the sample config file after removing the hash:

```sh
#
# This is a sample network config uncomment lines to configure the network
#


# Static config for eth0
#auto eth0
#iface eth0 inet static
#	address 192.168.0.2
#	netmask 255.255.255.0
#	gateway 192.168.0.1
#	up echo nameserver 192.168.0.1 > /etc/resolv.conf

# DHCP config for eth0
auto eth0
iface eth0 inet dhcp
```
4. Run command "ifconfig" to check the current IP address
5. Power-on the container and launch its console
6. Once powered-on, the container should automatically get an IP address from the NAT cloud via DHCP with default subnet of 192.168.122.0/24
>NOTE: This is the reason why we have assigned an IP address from 192.168.122.0/24 subnet to the EXOSVM.
6. Ping the EXOSVM and make sure that it is reachable from the Automation container
7. Run command ```ansible --version``` to check the currently installed version of Ansible

## System Setup

Ideally, there is no need to install Ansible on Network Automation Docker Container as it already comes pre-installed with it. If you are using your own Linux system, you will need to go through the section below to install Ansible:

### Installing Ansible

1. Run the below commands to install the required files/packages:

```sh
root@NetworkAutomation-1:~# apt-get update
root@NetworkAutomation-1:~# apt-get install git
root@NetworkAutomation-1:~# git clone https://github.com/ansible/ansible.git --recursive
```
2. Run ```ls``` to make sure that ansible folder is successfully cloned. Sample command and its output is below:

```sh
root@NetworkAutomation-1:~# ls
ansible
```

3. One can periodically pull the latest developments from Ansible's Github Repository by running the below commands:

```sh
root@NetworkAutomation-1:~# cd ansible
root@NetworkAutomation-1:~# git pull
```

### Setting up the Virtual Environment

The main purpose of Python virtual environments is to create an isolated environment for Python projects. This means that each project can have its own dependencies, regardless of what dependencies every other project has. It is not really necessary to create a virtual environment but it is recommended to do so. Visit this [page] for details.

1. Run below commands to setup the virtual environment:

```sh
root@NetworkAutomation-1:~# apt-get install virtualenv
root@NetworkAutomation-1:~# cd ansible
root@NetworkAutomation-1:~/ansible# virtualenv venv
```
2. Run ```ls``` while being under 'ansible' folder and make sure that a 'venv' folder is successfully created. Sample command and its output is below:

```sh
root@NetworkAutomation-1:~/ansible# ls
CODING_GUIDELINES.md  bin                       hacking           shippable.yml
COPYING               changelogs                lib               test
MANIFEST.in           contrib                   licenses          tox.ini
MODULE_GUIDELINES.md  docs                      packaging         venv
Makefile              docsite_requirements.txt  requirements.txt
README.rst            examples                  setup.py
```

3. Activate the 'venv' virtual environment

```sh
root@NetworkAutomation-1:~# source ./ansible/venv/bin/activate
root@NetworkAutomation-1:~# source ./ansible/hacking/env-setup
```
>NOTE: Commands in Step 3 needs to be executed at each login.

4. Once virtual environment is activated, the prompt will look something like below:

```sh
(venv) root@NetworkAutomation-1:~#
```

5. Ansible also uses some other Python modules that need to be installed. Run the below command to install the same

```sh
(venv) root@NetworkAutomation-1:~/ansible# pip install -r ./requirements.txt
```
> NOTE: Make sure to execute the above command while being in 'ansible' directory or else the command will not work.

### Configuring the Local hosts file

Navigate to /etc/hosts and add an entry for EXOS-VM switch. Here we are using 'S1' as its resolved name/FQDN. Sample below:

```sh
(venv) root@NetworkAutomation-1:~# cat /etc/hosts
127.0.1.1       NetworkAutomation-1
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.122.100 S1
```

> If you have more than one virtual switch in your topology, then additional lines under ```192.168.122.100 S1```

> Visit this [tutorial] to understand how to edit a file using nano editor

### Creating an Ansible configuration file

Certain settings in Ansible are adjustable via a configuration file (ansible.cfg). The stock configuration should be sufficient for most users, but there may be reasons you would want to change them. Creating a new ansible.cfg file will override default settings in Ansible which is present under /etc/. There are [reasons] why one might want to do that but for the sake of this guide we don't need to go that way. Follow the below to create our own ansible.cfg file:

Create a file named as "ansible.cfg" under root directory and add below statements to it:
```sh
[defaults]
ansible_python_interpreter = ~/ansible/venv/bin/python
host_key_checking = False
inventory = ~/playbooks/hosts 
``` 
Sample output:

```sh
(venv) root@NetworkAutomation-1:~# cat ansible.cfg
[defaults]
ansible_python_interpreter =~/ansible/venv/bin/python
host_key_checking = False
inventory = ~/playbooks/hosts
```

### Creating and configuring the Ansible inventory file

Ansible works against multiple systems in your infrastructure at the same time. It does this by selecting portions of systems listed in Ansible’s inventory, which defaults to being saved in the location /etc/ansible/hosts. But in our case, we will be creating another directory and create our inventory file in there. Follow the below steps to do so:

1. Create a folder named as "playbooks" under root directory using command ```mkdir playbooks```
2. 'cd' into that folder using command ```cd playbooks```
3. Create a file named as "hosts" using command ```nano hosts```
4. Add below statements to it and save it by hitting ```ctrl+x``` and then ```y```

```sh
[all_exos]
S1
```

Sample Inventory file:

```sh
(venv) root@NetworkAutomation-1:~/playbooks# cat hosts
[all_exos]
S1
```
### Creating and configuring the Variable file

Ansible uses a combination of a "hosts" file and a "group_vars" directory to pull variables per host group and run Ansible plays/tasks against hosts. Here we are declaring all the varibles in a single file called "all_exos.yaml" under "group_vars" directory.

>NOTE: The name of the file has to be same as the group name in inventory file or else ansible will not be able to pick up the file during runtime

1. Create a folder named as "group_vars" under "playbooks" folder using command ```mkdir group_vars``` 
2. 'cd' into it using command ```cd group_vars```
3. Create a file named as "all_exos.yaml" using command ```nano all_exos.yaml```
4. Add below statements to it and save by hitting ```ctrl+x``` and then ```y```

```sh
---
ansible_network_os: exos
ansible_connection: network_cli
ansible_user: xtrm_user
ansible_ssh_pass: xtrm_pass
```

Sample File:

```sh
(venv) root@NetworkAutomation-1:~/playbooks/group_vars# cat all_exos.yaml
---
ansible_network_os: exos
ansible_connection: network_cli
ansible_user: xtrm_user
ansible_ssh_pass: xtrm_pass
```

## Testing the Setup

1. To confirm your credentials, connect to a network device(EXOS-VM) manually using command ```ssh xtrm_user@S1``` and run few show commands to verify the same

2. Once done, exit out of it and run command ```ansible S1 -m ping``` from the Network Automation container to check if S1 is reachable through ansible

Expected Result:
```sh
(venv) root@NetworkAutomation-1:~# ansible S1 -m ping
S1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
That concludes this guide. Now device S1 is ready to accept ansible [Ad-Hoc Commands](./WorkingwithAnsibleAdHocCommands.md) and Playbooks from the Network Automation container.

[GNS3 Marketplace]: <https://www.gns3.com/marketplace/appliances>
[GitHub Website]: <https://github.com/extremenetworks/Virtual_EXOS#qcow2-files-for-gns3>
[link]: <https://github.com/extremenetworks/Virtual_EXOS/blob/master/GNS3_EXOS-VM_Guide.md#step-8-import-exos-vm-as-appliance>
[page]: <https://realpython.com/python-virtual-environments-a-primer/>
[tutorial]:<https://www.tutorialspoint.com/articles/how-to-use-nano-text-editor>
[reasons]: <https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html>

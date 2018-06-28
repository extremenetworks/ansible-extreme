## Working with Ansible Ad-Hoc Commands on EXOS Devices

## Sections:

* [Introduction](#introduction)
* [Pre-requisite](#pre-requisite)
* [Basic Ansible Commands](#basic-ansible-commands)
	- [Check Ansible Version](#check-ansible-version)
	- [Check available hosts in Ansible Inventory](#check-available-hosts-in-ansible-inventory)
	- [Ansible Help](#ansible-help)
* [Basic Ansible Ad-Hoc Commands](#basic-ansible-ad-hoc-commands)
	- [Pinging Hosts](#pinging-hosts)
	- [Gathering Facts](#gathering-facts)
	- [Get Version Details](#get-version-details)
  	- [Logging in with a different user](#logging-in-with-a-different-user)
  	- [Filtering Output](#filtering-output)
  	- [Saving Output in a File](#saving-output-in-a-file)
* [Conclusion](#conclusion)

## Introduction

An ad-hoc command is something that you might type in order to do something really quick, but don’t want to save for later.

This is a good place to start to understand the basics of what Ansible can do prior to learning the playbooks language. Ad-hoc commands can also be used to do quick things that you might not necessarily want to write a full playbook for.

This guide will be focused mainly on the application of these commands on Network devices, and in our case its for EXOS devices.

## Pre-requisite:

A working setup of Ansible system with at least one EXOS device.

> Refer [this](https://github.com/extremenetworks/ansible-extreme/blob/master/GettingStartedwithAnsibleusingGNS3.md) guide to prepare a similar setup on GNS3

## Basic Ansible Commands

### Check Ansible Version

```ansible --version```

Run this command to check the version of ansible currently installed on a system. This also shows other details such as python version, ansible configuration file path etc.

Sample command and output:

```sh
(venv) root@Net:~# ansible --version
ansible 2.7.0.dev0 (devel 0102e42272) last updated 2018/06/17 04:22:52 (GMT +000)
  config file = /root/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /root/ansible/lib/ansible
  executable location = /root/ansible/bin/ansible
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
```  
### Check available hosts in Ansible Inventory

```ansible --list-hosts all```

Run this command to check all the available hosts in Ansible's Inventory. In order to check the hosts available in a particular group, replace the word ``'all'`` with the name of the group.

Sample command and output:

```sh
(venv) root@Net:~# ansible --list-hosts all
  hosts (5):
    S1
    S2
    S3
    S4
    S5
```
### Ansible Help

```ansible -h```

Run this command to see all the available options under ```ansible```.

Refer [this](https://docs.ansible.com/ansible/latest/user_guide/command_line_tools.html) document for more details on working with Command Line Tools

## Basic Ansible Ad-Hoc Commands

### Pinging Hosts

```ansible S1 -m ping```

Use this command to check if the hosts are reachable from ansible.

Sample command and output below:

```sh
(venv) root@Net:~# ansible S1 -m ping
S1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Gathering Facts

```ansible <device> -m setup```

Use this command to gather different facts of the device. Replace ```device``` with the group name or hostname of the device for which the detail needs to be gathered.

Sample command and **truncated** output below:

```sh
(venv) root@Net:~# ansible S1 -m setup
S1 | SUCCESS => {
    "ansible_facts": {
        "ansible_apparmor": {
            "status": "disabled"
        },
```
### Get Version Details

```ansible <device> -m exos_command -a "commands='show version'"```

Above command will return the output of "show version" command from the EXOS devices.

Sample command and output below:

```sh
(venv) root@Net:~# ansible S1 -m exos_command -a "commands='show version'"
S1 | SUCCESS => {
    "changed": false,
    "stdout": [
        "Switch      : PN:1N2039    SN:123456   Rev 01 BootROM: 1.2        IMG: 22.5.1.7  \nPSUCTRL-1   : PN:MEAD      SN:MD1      Rev 01 BootROM: 2.1       \nPSUCTRL-2   : PN:MEAD      SN:MD2      Rev 11 BootROM: 2.3       \nmouse-usb   : PN:MOUSE     SN:4321     Rev 11 BootROM: 4.3       \nfloppy-A    :\nPSU-1       : Internal PSU-2 PN:1N2039 SN:12345\nPSU-2       : Internal PSU-3 PN:1N2039 SN:12345\n\nImage   : ExtremeXOS version 22.5.1.7 by release-manager\n          on Tue May 22 11:01:38 EDT 2018\nBootROM : 1.2\nCertified Version : EXOS Linux  3.18.48, FIPS fips-ecp-2.0.16"
    ],
    "stdout_lines": [
        [
            "Switch      : PN:1N2039    SN:123456   Rev 01 BootROM: 1.2        IMG: 22.5.1.7  ",
            "PSUCTRL-1   : PN:MEAD      SN:MD1      Rev 01 BootROM: 2.1       ",
            "PSUCTRL-2   : PN:MEAD      SN:MD2      Rev 11 BootROM: 2.3       ",
            "mouse-usb   : PN:MOUSE     SN:4321     Rev 11 BootROM: 4.3       ",
            "floppy-A    :",
            "PSU-1       : Internal PSU-2 PN:1N2039 SN:12345",
            "PSU-2       : Internal PSU-3 PN:1N2039 SN:12345",
            "",
            "Image   : ExtremeXOS version 22.5.1.7 by release-manager",
            "          on Tue May 22 11:01:38 EDT 2018",
            "BootROM : 1.2",
            "Certified Version : EXOS Linux  3.18.48, FIPS fips-ecp-2.0.16"
        ]
    ]
}
```
> NOTE: "stdout" and "stdout_lines" are the return values of "exos_command". They both have the same result except the way the output is formatted.

### Logging in with a different User

Use below command if you wish to login to the device with a different user as opposed to what has been configured in the "group_vars" directory.

```ansible <device> -m exos_command -a "commands='show version'" -u <username> -k```

### Filtering Output

One can also make use of "grep" to filter the output. For example, use below command to just get the details of the Image:

```ansible S1 -m exos_command -a "commands='show version | grep Image'"```

Sample command and output below:

```sh
root@Net:~# ansible S1 -m exos_command -a "commands='show version | grep Image'"
S1 | SUCCESS => {
    "changed": false,
    "stdout": [
        "Image   : ExtremeXOS version 22.5.1.7 by release-manager"
    ],
    "stdout_lines": [
        [
            "Image   : ExtremeXOS version 22.5.1.7 by release-manager"
        ]
    ]
}
```
NOTE: For all the above examples in this section, ```show version``` can be replaced by any other valid ```show``` command. For example, use below command to check the details of Default VLAN.

```ansible S1 -m exos_command -a "commands='show ipconfig vlan Default'"```

### Saving Output in a File

```root@Net:~# ansible S1 -m exos_command -a "commands='show version'" > sh_ver.txt```

Above command will save the contents of ```show version``` command in a file named as "sh_ver.txt" instead of displaying it on the console. Run command ```ls``` to check if the file has been created or not. Once confirmed, open it using ```cat sh_ver.txt``` or ```more sh_ver.txt``` and verify the contents. Below is the sample command and output:

```sh
(venv) root@Net:~# ansible S1 -m exos_command -a "commands='show version'" > sh_ver.txt
(venv) root@Net:~# ls
ansible  ansible.cfg  playbooks  sh_ver.txt
```
## Conclusion

At the time of writing this guide, there is no support for running configuration commands on EXOS devices. As we expect to release an ```exos_config``` module soon, this guide will be updated accordingly with some sample configuration-related Ad-Hoc Commands.

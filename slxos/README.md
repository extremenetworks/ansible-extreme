# Ansible for SLX Devices

This document explains how to use Ansible with Extreme SLX devices.

These modules are currently in development, and have not yet been merged with upstream Ansible. As such there are additional steps required to set up a development environment tracking the latest modules. 

These instructions will show you how to set up a dev environment in your home directory, and provides example playbooks for using each of the available modules.

These instructions will be updated when the modules are merged upstream.

## Sections:

* [System Setup](#system-setup)
* [Multi-Node Demo](#multi-node-demo)
* [Command](#command-module---slxos_command) - `slxos_command`
* [Config](#config-module---slxos_config) - `slxos_config`
* [Facts](#facts-module---slxos_facts) - `slxos_facts`
* [VLAN](#vlan-module---slxos_vlan) - `slxos_vlan`
* [Interface](#interface-module---slxos_interface) - `slxos_interface`
* [LinkAgg](#link-aggregation-module---slxos_linkagg) - `slxos_linkagg`
* [LLDP](#lldp-module---slxos_lldp) - `slxos_lldp`
* [L2 Interface](#l2-interface-module---slxos_l2_interface) - `slxos_l2_interface`
* [L3 Interface](#l3-interface-module---slxos_l3_interface) - `slxos_l3_interface`

All playbooks shown in this README are also available in the [playbooks](./playbooks) directory.

## System Setup

Install Ansible in your home directory, using our latest development branch:

> NB in future you will be able to install Ansible using [normal Ansible installation procedures](https://docs.ansible.com/ansible/intro_installation.html). Our modules have not yet been merged upstream.

### First time setup:

#### Ansible Install

```shell
git clone -b slxos_modules https://github.com/StackStorm/ansible
cd ansible
virtualenv venv
.  ./venv/bin/activate
pip install -r requirements.txt
```

#### Initial Configuration

Add entries to `/etc/hosts` file for your switches - e.g. `172.16.10.42 slx01`

Create `~/.ansible.cfg` containing this:
```ini
[defaults]
ansible_python_interpreter=~/ansible/venv/bin/python
host_key_checking = False
inventory = ~/playbooks/hosts 
```

Create these directories: `~/playbooks`, `~/playbooks/backups`, `~/playbooks/group_vars`

Create the file `~/playbooks/hosts` containing this:
```ini
[slx]
slx01 ansible_network_os=slxos ansible_connection=network_cli
```

If you have more than one switch (e.g. `slx02`, `slx03`, add additional lines as required)

Create the file `~/playbooks/group_vars/slx.yaml` containing this:
```yaml
---
ansible_user: admin
ansible_ssh_pass: password
```

Replace `password` with the password you use to authenticate to your SLX devices. Yes, you should really be using `ansible-vault` for this file. If you know what that is, use it. If you don't, don't worry about it right now.

### At Each Login:

The above commands only need to be run the first time you do setup. Next time you login, you just need to run these commands:

```shell
cd ansible
. ./venv/bin/activate
. ./hacking/env-setup
```

Periodically you should pull down the latest developments. Run these commands:
```shell
cd ~/ansible
git pull
```

## Multi-Node Demo

Check the [playbooks/demo](./playbooks/demo) directory to see a playbook that configures various settings across multiple devices.

It configures interfaces, IP address, NTP settings, hostnames, and BGP, using `hosts_vars` and `group_vars`.

The rest of this README outlines individual modules.

## Command Module - slxos_command

This module is used to run commands on SLX devices - e.g `show` commands. It is not for use with configuration commands. The syntax is very similar to [ios_command](http://docs.ansible.com/ansible/latest/ios_command_module.html).

This can be used to do things like run `show chassis`, and capture the output. This output can be used in further tasks, displayed to screen, or written to file. It can also look for specific lines in the output, and fail if those lines are not present. This is useful if you want to do things like check NTP synchronisation state.

All commands below are run from the `~/playbooks` directory.

1. Run `show version` and publish output:
Create [slx_command_ver.yaml](./playbooks/slx_command_ver.yaml), containing this:
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - name: Grab version
      slxos_command:
        commands: "show version"
      register: show_ver

    - name: var_result
      debug:
        var: show_ver.stdout_lines[0]
```
Run it with `ansible-playbook slx_command_ver.yaml`. Here's some example output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_command_ver.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [Grab version] ******************************************************************************************************************
ok: [slx01]

TASK [var_result] ********************************************************************************************************************
ok: [slx01] => {
    "failed": false,
    "show_ver.stdout_lines[0]": [
        "SLX-OS Operating System Software",
        "SLX-OS Operating System Version: 17s.1.02",
        "Copyright (c) 1995-2018 Brocade Communications Systems, Inc.",
        "Firmware name:      17s.1.02",
        "Build Time:         00:06:59 Sep 28, 2017",
        "Install Time:       10:58:51 Sep 30, 2017",
        "Kernel:             2.6.34.6",
        "Host Version:       Ubuntu 14.04 LTS",
        "Host Kernel:        Linux 3.14.17",
        "",
        "Control Processor:   QEMU Virtual CPU version 2.0.0",
        "",
        "System Uptime:   143days 19hrs 29mins 7secs ",
        "",
        "Slot    Name    Primary/Secondary Versions                         Status",
        "---------------------------------------------------------------------------",
        "SW/0    SLX-OS  17s.1.02                                           ACTIVE*",
        "                17s.1.02"
    ]
}

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=3    changed=0    unreachable=0    failed=0
```

2. Run a command and check the output

In this example, we're checking for specific NTP servers in the output. On my device there is only one NTP server configured and in use. This playbook should pass the first task, and fail the second one.

Here's what's configured on my device:
```shell
SLX01# show run | inc ntp
ntp server 172.16.10.2 use-vrf mgmt-vrf
SLX01#
```
Create [slx_command_ntp.yaml](./playbooks/slx_command_ntp.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Check ntp status
      slxos_command:
        commands: "show ntp status"
        wait_for: result[0] contains 172.16.10.2
        retries: 1

    - name: Check for non-existent NTP server
      slxos_command:
        commands: "show ntp status"
        wait_for: result[0] contains 172.16.10.3
        retries: 1
```

Example output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_command_ntp.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [Check ntp status] **************************************************************************************************************
ok: [slx01]

TASK [Check for non-existent NTP server] *********************************************************************************************
fatal: [slx01]: FAILED! => {"changed": false, "failed": true, "failed_conditions": ["result[0] contains 172.16.10.3"], "msg": "One or more conditional statements have not been satisfied"}
	to retry, use: --limit @/home/lhill/playbooks/slx_command_ntp.retry

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=1

(venv) lhill@bwc:~/playbooks$
```
Note that this is the expected behavior - that second task **should** fail.

## Config Module - slxos_config

This module is used to manage configuration sections on SLXes. It is not for use with `show` commands. The syntax is very similar to [ios_config](http://docs.ansible.com/ansible/latest/ios_config_module.html). It is used to ensure config lines are present. These can be at the global level, or under specific parents (e.g. under an `interface Ethernet` stanza or an access-list). You can also run commands before or after changing lines. This is useful when updating ACLs.

This module will report if any changes were made. This is very handy for auditing devices - you can keep re-running the same playbook, and identify which devices had unauthorized changes made. It can also be run in `check` mode, where it reports differences, but does not make changes. 

You can choose if you want to save the configuration after making changes. There are four options for `save_when`:

* `never` - this is the default. Don't save changes.
* `always` - always save the config, even if nothing changed
* `changed` - save the config only if this playbook run changed it. Note the difference between this behavior and `modified` below.
* `modified` - save the config if the running and startup configs are different. This is handy if someone else has been making changes without saving them.

1. Set a global configuration value

This example sets an NTP server if not already set, and saves the configuration if we change it.

Create [slx_config_set_ntp.yaml](./slx_config_set_ntp.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: NTP server 172.16.10.3
      slxos_config:
        lines: "ntp server 172.16.10.3 use-vrf mgmt-vrf"
        save_when: changed
```

This was our config before running the playbook:
```
SLX01# show run | inc ntp
ntp server 172.16.10.2 use-vrf mgmt-vrf
SLX01#
```
Example output (note the added `-vv` here to give us additional information about what's happening):
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook -vv slx_config_set_ntp.yaml
ansible-playbook 2.6.0 (slxos_modules 2a7acba239) last updated 2018/02/21 02:06:16 (GMT +200)
  config file = /home/lhill/.ansible.cfg
  configured module search path = [u'/home/lhill/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/lhill/ansible/lib/ansible
  executable location = /home/lhill/ansible/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /home/lhill/.ansible.cfg as config file

PLAYBOOK: slx_config_set_ntp.yaml ****************************************************************************************************
1 plays in slx_config_set_ntp.yaml

PLAY [slx01] *************************************************************************************************************************
META: ran handlers

TASK [NTP server 172.16.10.3] ********************************************************************************************************
task path: /home/lhill/playbooks/slx_config_set_ntp.yaml:8
changed: [slx01] => {"changed": true, "commands": ["ntp server 172.16.10.3 use-vrf mgmt-vrf"], "failed": false, "updates": ["ntp server 172.16.10.3 use-vrf mgmt-vrf"]}
META: ran handlers
META: ran handlers

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```
Now look at the startup and running configs on the device:
```
SLX01# show run | inc ntp
ntp server 172.16.10.2 use-vrf mgmt-vrf
ntp server 172.16.10.3 use-vrf mgmt-vrf
SLX01# show startup-config | inc ntp
ntp server 172.16.10.2 use-vrf mgmt-vrf
ntp server 172.16.10.3 use-vrf mgmt-vrf
SLX01#
```
We can see our command has been added **and saved**.

Now run the playbook again:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook -v slx_config_set_ntp.yaml
Using /home/lhill/.ansible.cfg as config file

PLAY [slx01] *************************************************************************************************************************

TASK [NTP server 172.16.10.3] ********************************************************************************************************
ok: [slx01] => {"changed": false, "failed": false}

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```
Note the output: `changed=0`. It sees that the line is already there, and does not need to be changed.

2. Removing a configuration line

The `slxos_config` module is not declarative. By default it will not remove additional lines. There are ways around this with more complex playbooks. That's an exercise for the reader. But you can use `no` statements. Be aware that this will **always** report changes, because it sees that command as 'not present' so runs it every time.

Create [slx_config_remove_ntp.yaml](./playbooks/slx_config_remove_ntp.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: NTP server 172.16.10.3
      slxos_config:
        lines: "no ntp server 172.16.10.3 use-vrf mgmt-vrf"
```

Output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook -v slx_config_remove_ntp.yaml
Using /home/lhill/.ansible.cfg as config file

PLAY [slx01] *************************************************************************************************************************

TASK [NTP server 172.16.10.3] ********************************************************************************************************
changed: [slx01] => {"changed": true, "commands": ["no ntp server 172.16.10.3 use-vrf mgmt-vrf"], "failed": false, "updates": ["no ntp server 172.16.10.3 use-vrf mgmt-vrf"]}

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```

3. More complex configuration using `parents`

This example has multiple tasks - first it grabs the running config, and saves it to a local timestamped backup file. Then it makes a configuration change, but only on one specific interface. This uses the `parents` option. Also note the use of `gather_facts: yes` to get the current date & time, which is then used in the variable `ansible_facts.date_time.iso8601`. Here we're going to enable PTP on a specific interface:

[slx_config_parents.yaml](./playbooks/slx_config_parents.yaml):
```yaml
---
- hosts: slx01
  gather_facts: yes
  
  tasks:
    - name: Get running config
      slxos_command:
        commands: show run
      register: show_run

    - name: Save config to file
      copy:
        content: "{{ show_run.stdout[0] }}"
        dest: "backups/{{inventory_hostname}}-{{ansible_facts.date_time.iso8601}}.txt"

    - name: PTP on Eth 0/24
      slxos_config:
        lines: protocol ptp
        parents: interface Ethernet 0/24
        save_when: modified
```
Example output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_config_parents.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************
ok: [slx01]

TASK [Get running config] ************************************************************************************************************
ok: [slx01]

TASK [Save config to file] ***********************************************************************************************************
changed: [slx01]

TASK [PTP on Eth 0/24] ***************************************************************************************************************
changed: [slx01]

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=4    changed=2    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```
We can see that we have a backup file saved locally:
```shell
(venv) lhill@bwc:~/playbooks$ head backups/slx01-2018-02-21T06\:55\:58Z.txt
root enable
host-table aging-mode conversational
clock timezone Etc/GMT
hardware
 profile tcam default
 profile overlay-visibility default
 profile route-table default maximum_paths 8
 system-mode default
!
http server use-vrf default-vrf
(venv) lhill@bwc:~/playbooks$
```
And we can see that our `protocol ptp` command was added to one specific interface:
```
SLX01# show run | begin ^interface\ Ethernet\ 0/23
interface Ethernet 0/23
 shutdown
!
interface Ethernet 0/24
 protocol ptp
 !
 shutdown
!
interface Ethernet 0/25
 shutdown
!
```

## Facts Module - slxos_facts

This module retrieves information such as serial, model number, interface data, IPv4 & IPv4 addresses, and running config. It returns it in a structured format.

The `gather_subset:` option controls which elements are retrieved. Options are `all`, `hardware`, `config` and `interfaces`. By default, the config is not retrieved. You can also exclude options with: `!interfaces`.

The interface is similar to to the [ios_facts](http://docs.ansible.com/ansible/devel/modules/ios_interface_module.html) module.

1. Get default facts and print to screen

[slx_facts_default.yaml](./playbooks/slx_facts_default.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Get default set of facts
      slxos_facts:

    - name: Print info about device
      vars:
        msg: |
          {{ ansible_net_hostname }} is a {{ ansible_net_model }} running {{ ansible_net_version }}.
          Serial number: {{ ansible_net_serialnum }}. Free memory: {{ ansible_net_memfree_mb }}/{{ ansible_net_memtotal_mb }}MB
          All IPv4 addresses: {{ ansible_net_all_ipv4_addresses | to_nice_json }}
          All IPv6 addresses: {{ ansible_net_all_ipv6_addresses | to_nice_json }}
          LLDP neighbors: {{ ansible_net_neighbors | to_nice_json }}
      debug:
        msg: "{{ msg.split('\n') }}"
```

Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_facts_default.yaml

PLAY [slx01] **************************************************************************************************************************************

TASK [Get default set of facts] *******************************************************************************************************************
ok: [slx01]

TASK [Print info about device] ********************************************************************************************************************
---
ok: [slx01] => {
    "failed": false,
    "msg": [
        "SLX is a BR-SLX9140 running 17s.1.02.",
        "Serial number: EXH3349M00A. Free memory: 5748/7879MB",
        "All IPv4 addresses: [",
        "    \"100.100.100.100\"",
        "]",
        "All IPv6 addresses: [",
        "    \"2001::6\", ",
        "    \"2002::6\", ",
        "    \"2003::6\", ",
        "    \"2041::1\"",
        "]",
        "LLDP neighbors: {",
        "    \"Eth 0/41\": [",
        "        {",
        "            \"host\": \"SLX\", ",
        "            \"port\": \"Ethernet 0/41\"",
        "        }",
        "    ]",
        "}",
        ""
    ]
}

PLAY RECAP ****************************************************************************************************************************************
slx01                      : ok=2    changed=0    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

2. Gather interface data only

[slx_facts_interfaces.yaml](./playbooks/slx_facts_interfaces.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Get interface info
      slxos_facts:
        gather_subset: interfaces

    - name: Print interface info
      debug:
        msg: "{{ ansible_net_interfaces }}"
```

Example output (truncated):

```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_facts_interfaces.yaml

PLAY [slx01] **************************************************************************************************************************************

TASK [Get interface info] *************************************************************************************************************************
ok: [slx01]

TASK [Print interface info] ***********************************************************************************************************************
ok: [slx01] => {
    "failed": false,
    "msg": {
        "Ethernet 0/1": {
            "bandwidth": "Nil",
            "description": null,
            "duplex": "Full",
            "ipv4": [],
            "lineprotocol": "down",
            "macaddress": "609c.9fcd.1d05",
            "mtu": 1548,
            "operstatus": "admin down",
            "type": "Ethernet"
<snip>
        "Ethernet 0/40": {
            "bandwidth": "10000 Mbit",
            "description": null,
            "duplex": "Full",
            "ipv4": [],
            "lineprotocol": "up",
            "macaddress": "609c.9fcd.1d2c",
            "mtu": 1548,
            "operstatus": "up",
            "type": "Ethernet"
        },
        "Ethernet 0/41": {
            "bandwidth": "10000 Mbit",
            "description": null,
            "duplex": "Full",
            "ipv4": [],
            "ipv6": [
                {
                    "address": "2041::1",
                    "masklen": 64
                }
            ],
            "lineprotocol": "up",
            "macaddress": "609c.9fcd.1d2d",
            "mtu": 1548,
            "operstatus": "up",
            "type": "Ethernet"
        },
        "Ethernet 0/42": {
            "bandwidth": "Nil",
            "description": null,
            "duplex": "Full",
            "ipv4": [],
            "lineprotocol": "down",
            "macaddress": "609c.9fcd.1d2e",
            "mtu": 1548,
            "operstatus": "admin down",
            "type": "Ethernet"
<snip>
        "Loopback 1": {
            "bandwidth": "Nil",
            "description": null,
            "duplex": null,
            "ipv4": [
                {
                    "address": "100.100.100.100",
                    "subnet": "32"
                }
            ],
            "ipv6": [
                {
                    "address": "2001::6",
                    "masklen": 128
                }
            ],
            "lineprotocol": "up",
            "macaddress": null,
            "mtu": 1500,
            "operstatus": "up",
            "type": null
        },
        "Loopback 2": {
            "bandwidth": "Nil",
            "description": null,
            "duplex": null,
            "ipv4": [],
            "ipv6": [
                {
                    "address": "2002::6",
                    "masklen": 128
                }
            ],
            "lineprotocol": "down",
            "macaddress": null,
            "mtu": 1500,
            "operstatus": "admin down",
            "type": null
        },
        "Loopback 3": {
            "bandwidth": "Nil",
            "description": null,
            "duplex": null,
            "ipv4": [],
            "ipv6": [
                {
                    "address": "2003::6",
                    "masklen": 128
                }
            ],
            "lineprotocol": "down",
            "macaddress": null,
            "mtu": 1500,
            "operstatus": "admin down",
            "type": null
        },
        "Port-channel 73": {
            "bandwidth": "10000 Mbit",
            "description": null,
            "duplex": null,
            "ipv4": [],
            "lineprotocol": "up",
            "macaddress": null,
            "mtu": 9216,
            "operstatus": "up",
            "type": "AGGREGATE"
        }
    }
}

PLAY RECAP ****************************************************************************************************************************************
slx01                      : ok=2    changed=0    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

3. Get all facts

[slx_facts_all.yaml](./playbooks/slx_facts_all.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Get all facts
      slxos_facts:
        gather_subset: all
```

Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_facts_all.yaml

PLAY [slx01] **************************************************************************************************************************************

TASK [Get all facts] ******************************************************************************************************************************
ok: [slx01]

PLAY RECAP ****************************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

Note that no output is displayed by default. You have to either use verbose mode (`-vvvv`), or explicitly publish variables.

With verbose output (truncated):

```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_facts_all.yaml -vvvv
ansible-playbook 2.6.0 (slxos_modules c76b3db751) last updated 2018/02/27 15:32:41 (GMT -700)
  config file = /home/extreme/.ansible.cfg
  configured module search path = [u'/home/extreme/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/extreme/ansible/lib/ansible
  executable location = /home/extreme/ansible/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /home/extreme/.ansible.cfg as config file
setting up inventory plugins
Parsed /home/extreme/playbooks/hosts inventory source with ini plugin
Loading callback plugin default of type stdout, v2.0 from /home/extreme/ansible/lib/ansible/plugins/callback/default.pyc

PLAYBOOK: slx_facts_all.yaml **********************************************************************************************************************
1 plays in slx_facts_all.yaml

PLAY [slx01] **************************************************************************************************************************************
META: ran handlers

TASK [Get all facts] ******************************************************************************************************************************
task path: /home/extreme/playbooks/slx_facts_all.yaml:8
<slx01> attempting to start connection
<slx01> using connection plugin network_cli
<slx01> local domain socket does not exist, starting it
<slx01> control socket path is /home/extreme/.ansible/pc/3d7de011f8
<slx01> <slx01> ESTABLISH CONNECTION FOR USER: admin on PORT 22 TO slx01
<slx01> <slx01> ssh connection done, setting terminal
<slx01> <slx01> loaded terminal plugin for network_os slxos
<slx01> <slx01> loaded cliconf plugin for network_os slxos
<slx01> <slx01> firing event: on_open_shell()
<slx01> <slx01> ssh connection has completed successfully
<slx01> connection to remote device started successfully
<slx01> local domain socket listeners started successfully
<slx01>
<slx01> local domain socket path is /home/extreme/.ansible/pc/3d7de011f8
Using module file /home/extreme/ansible/lib/ansible/modules/network/slxos/slxos_facts.py
<slx01> ESTABLISH LOCAL CONNECTION FOR USER: extreme
<slx01> EXEC /bin/sh -c 'echo ~ && sleep 0'
<slx01> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668 `" && echo ansible-tmp-1520130364.92-159489930142668="` echo /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668 `" ) && sleep 0'
<slx01> PUT /home/extreme/.ansible/tmp/ansible-local-7538CAH2vF/tmpoB9zfl TO /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668/slxos_facts.py
<slx01> EXEC /bin/sh -c 'chmod u+x /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668/ /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668/slxos_facts.py && sleep 0'
<slx01> EXEC /bin/sh -c '/usr/bin/python /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668/slxos_facts.py && sleep 0'
<slx01> EXEC /bin/sh -c 'rm -f -r /home/extreme/.ansible/tmp/ansible-tmp-1520130364.92-159489930142668/ > /dev/null 2>&1 && sleep 0'
ok: [slx01] => {
    "ansible_facts": {
        "ansible_net_all_ipv4_addresses": [
            "100.100.100.100"
        ],
        "ansible_net_all_ipv6_addresses": [
            "2001::6",
            "2002::6",
            "2003::6",
            "2041::1"
        ],
        "ansible_net_config": "host-table aging-mode conversational\nclock timezone Etc/GMT\nhardware\n profile tcam default\n profile overlay-visibility default\n profile route-table default maximum_paths 8\n system-mode default\n!\nhttp server use-vrf default-vrf\nhttp server use-vrf mgmt-vrf\nnode-id 2\n!\nlogging raslog console INFO\nlogging auditlog class SECURITY\nlogging auditlog class CONFIGURATION\nlogging auditlog class FIRMWARE\nlogging syslog-facility local LOG_LOCAL7\nlogging syslog-client localip CHASSIS_IP\nswitch-attributes chassis-name SLX9140\nswitch-attributes host-name SLX\nno support autoupload enable\nsupport ffdc\nresource-<snip>",
        "ansible_net_gather_subset": [
            "hardware",
            "default",
            "interfaces",
            "config"
        ],
        "ansible_net_hostname": "SLX",
        "ansible_net_interfaces": {
            "Ethernet 0/1": {
                "bandwidth": "Nil",
                "description": null,
                "duplex": "Full",
                "ipv4": [],
                "lineprotocol": "down",
                "macaddress": "609c.9fcd.1d05",
                "mtu": 1548,
                "operstatus": "admin down",
                "type": "Ethernet"
<snip>
        "ansible_net_memfree_mb": 5747,
        "ansible_net_memtotal_mb": 7879,
        "ansible_net_model": "BR-SLX9140",
        "ansible_net_neighbors": {
            "Eth 0/41": [
                {
                    "host": "SLX",
                    "port": "Ethernet 0/41"
                }
            ]
        },
        "ansible_net_serialnum": "EXH3349M00A",
        "ansible_net_version": "17s.1.02"
    },
    "changed": false,
    "failed": false,
    "invocation": {
        "module_args": {
            "gather_subset": [
                "all"
            ]
        }
    }
}
META: ran handlers
META: ran handlers

PLAY RECAP ****************************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

## Interface module - slxos_interface

This module can be used to configure low-level interface parameters - e.g. enabled/disabled, speed, MTU, description. It is not used for configuring VLANs or IP addresses.

It is `declarative` - we can express our intent about the interface state, rx & tx rate, and desired LLDP neighbors, and it will report non-compliant systems. This can be used for checking cabling state. 

Note that we do not provide specific CLI.

This module has a similar interface to the [ios_interface](http://docs.ansible.com/ansible/devel/modules/ios_interface_module.html) module.

1. Enable interface and set description:
[slx_interface.yaml](./playbooks/slx_interface.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: configure interface
      slxos_interface:
        name: 'Ethernet 0/6'
        description: 'Configured by Ansible'
        enabled: True
```

Initial switch config:
```
SLX01# show run int eth 0/6
interface Ethernet 0/6
 shutdown
!
SLX01#
```
First run:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_interface.yaml -vv
ansible-playbook 2.6.0 (slxos_modules 2a7acba239) last updated 2018/02/21 02:06:16 (GMT +200)
  config file = /home/lhill/.ansible.cfg
  configured module search path = [u'/home/lhill/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/lhill/ansible/lib/ansible
  executable location = /home/lhill/ansible/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /home/lhill/.ansible.cfg as config file

PLAYBOOK: slx_interface.yaml *****************************************************************************************************************
1 plays in slx_interface.yaml

PLAY [slx01] *********************************************************************************************************************************
META: ran handlers

TASK [configure interface] *******************************************************************************************************************
task path: /home/lhill/playbooks/slx_interface.yaml:8
changed: [slx01] => {"changed": true, "commands": ["interface Ethernet 0/6", "description Configured by Ansible", "no shutdown"], "failed": false}
META: ran handlers
META: ran handlers

PLAY RECAP ***********************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```

New switch output:
```
SLX01# show run int eth 0/6
interface Ethernet 0/6
 description Configured by Ansible
 no shutdown
!
SLX01#
```

Run the playbook again, observe that it detects no changes required:

```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_interface.yaml

PLAY [slx01] *********************************************************************************************************************************

TASK [configure interface] *******************************************************************************************************************
ok: [slx01]

PLAY RECAP ***********************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```

Note **changed=0**.

2. Checking LLDP neighbors

[slx_interface_lldp.yaml](./playbooks/slx_interface_lldp.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Check LLDP neighbors
      slxos_interface:
        name: 'Ethernet 0/49'
        neighbors:
        - port: Ethernet 0/25
          host: DC2SPINE1
```
Switch status:
```
SLX01# show lldp nei
Local Port    Dead Interval  Remaining Life  Remote Port ID                    Remote Port Descr                 Chassis ID       Tx           Rx           System Name
Eth 0/10      120            118             ge-0/0/10                         ge-0/0/10.0                       001f.123f.c400   428537       125328       EX4200-VCF
Eth 0/11      120            96              ge-1/0/10                         ge-1/0/10.0                       001f.123f.c400   116010       125399       EX4200-VCF
Eth 0/13      120            107             cc4e.24dc.b7eb                    GigabitEthernet1/1/10             cc4e.24dc.b7e2   53021        53021        ICX7250-LAB
Eth 0/49      120            109             Ethernet 0/25                     Eth 0/25                          609c.9fcd.bf00   419489       213868       DC2SPINE1

Total no. of Records: 4
SLX01#
```
i.e. this playbook should pass.

Checking the output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_interface_lldp.yaml

PLAY [slx01] *********************************************************************************************************************************

TASK [Check LLDP neighbors] ******************************************************************************************************************
ok: [slx01]

PLAY RECAP ***********************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```

Now run a slightly modified playbook - this one will set the description, and check the LLDP neighbors. Note that this time we are looking for a non-existent neighbor, so it should fail:

[slx_interface_lldp_fail.yaml](./playbooks/slx_interface_lldp_fail.yaml)
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Set Description and check LLDP
      slxos_interface:
        name: Ethernet 0/49
        description: Ansible Foo
        neighbors:
        - port: Ethernet 0/25
          host: DC2SPINE2
```

And looking at the output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_interface_lldp_fail.yaml -vv
ansible-playbook 2.6.0 (slxos_modules 2a7acba239) last updated 2018/02/21 02:06:16 (GMT +200)
  config file = /home/lhill/.ansible.cfg
  configured module search path = [u'/home/lhill/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/lhill/ansible/lib/ansible
  executable location = /home/lhill/ansible/bin/ansible-playbook
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]
Using /home/lhill/.ansible.cfg as config file

PLAYBOOK: slx_interface_lldp_fail.yaml *******************************************************************************************************
1 plays in slx_interface_lldp_fail.yaml

PLAY [slx01] *********************************************************************************************************************************
META: ran handlers

TASK [Set Description and check LLDP] ********************************************************************************************************
task path: /home/lhill/playbooks/slx_interface_lldp_fail.yaml:8
fatal: [slx01]: FAILED! => {"changed": true, "failed": true, "failed_conditions": ["host DC2SPINE2"], "msg": "One or more conditional statements have not been satisfied"}
	to retry, use: --limit @/home/lhill/playbooks/slx_interface_lldp_fail.retry

PLAY RECAP ***********************************************************************************************************************************
slx01                      : ok=0    changed=0    unreachable=0    failed=1

(venv) lhill@bwc:~/playbooks$
```

## VLAN module - slxos_vlan

This module can be used to manage VLANs on an SLX. This is a _declarative_ module. This means that we can define our desired state, and it will make changes as required. We do **not** need to provide CLI commands. This has a similar interface to [ios_vlan](http://docs.ansible.com/ansible/devel/modules/ios_vlan_module.html).

1. Adding a VLAN, and setting interfaces to be access-mode for that VLAN:

[slx_vlan_add.yaml](./playbooks/slx_vlan_add.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - slxos_vlan:
        vlan_id: 101
        name: "my vlan"
        interfaces:
          - Ethernet 0/3
          - Ethernet 0/4
```

Initial switch configuration:
```
SLX01# show vlan brief
Total Number of VLANs configured    : 2
VLAN             Name            State                      Ports           Classification
(R)-RSPAN                                                   (u)-Untagged
                                                            (t)-Tagged
================ =============== ========================== =============== ====================
1                default         INACTIVE(no member port)

22               VLAN0022        INACTIVE(no member port)

SLX01#
```
Example output:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_vlan_add.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [slxos_vlan] ********************************************************************************************************************
changed: [slx01]

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```

And now our switch output looks like this:
```
SLX01# show vlan brief
Total Number of VLANs configured    : 3
VLAN             Name            State                      Ports           Classification
(R)-RSPAN                                                   (u)-Untagged
                                                            (t)-Tagged
================ =============== ========================== =============== ====================
1                default         INACTIVE(no member port)

22               VLAN0022        INACTIVE(no member port)

101              my vlan         INACTIVE(member port down) Eth 0/4(u)
                                                            Eth 0/3(u)

SLX01#
```

2. Removing a VLAN

We can use the `state: absent` option to tell our module "Ensure this VLAN is **not** present, and let me know if you need to make any changes to achieve that."

[slx_vlan_remove.yaml](./playbooks/slx_vlan_remove.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - slxos_vlan:
        vlan_id: 101
        state: absent
```
Example output, running it twice:
```shell
(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_vlan_remove.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [slxos_vlan] ********************************************************************************************************************
changed: [slx01]

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$ ansible-playbook slx_vlan_remove.yaml

PLAY [slx01] *************************************************************************************************************************

TASK [slxos_vlan] ********************************************************************************************************************
ok: [slx01]

PLAY RECAP ***************************************************************************************************************************
slx01                      : ok=1    changed=0    unreachable=0    failed=0

(venv) lhill@bwc:~/playbooks$
```
Note that on second run, `changed=0`.

And now our switch output is this:
```
SLX01# show vlan brief
Total Number of VLANs configured    : 2
VLAN             Name            State                      Ports           Classification
(R)-RSPAN                                                   (u)-Untagged
                                                            (t)-Tagged
================ =============== ========================== =============== ====================
1                default         INACTIVE(member port down) Eth 0/4(u)
                                                            Eth 0/3(u)

22               VLAN0022        INACTIVE(no member port)

SLX01#
```

## Link Aggregation module - slxos_linkagg

This module manages link aggregation groups on an SLX. We do **not** need to provide CLI commands, but instead define our desired state, and this module will make the necessary changes. This has a similar interface to [ios_linkagg](http://docs.ansible.com/ansible/devel/modules/ios_linkagg_module.html).

1. Provision LAG:

[slx_linkagg_add.yaml](./playbooks/slx_linkagg_add.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - name: Create Port Channel
      slxos_linkagg:
        group: 200
        mode: active
        members:
          - Ethernet 0/6
          - Ethernet 0/7
```
Pre-execution switch state:
```
SLX01# show port-channel summary
Flags:  D - Down                P - Up in port-channel (members)
        U - Up (port-channel)   * - Primary link in port-channel
        S - Switched            I - Insight Enabled
        M - Not in use. Min-links not met
===== =============== ========== ===============
Group Port-channel    Protocol   Member ports
===== =============== ========== ===============
34    Po 34   (U)     LACP       Eth 0/15 (D)
                                 Eth 0/16 (P)
SLX01# show run int eth 0/6
interface Ethernet 0/6
 shutdown
!
SLX01# show run int eth 0/7
interface Ethernet 0/7
 shutdown
!
```
Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_linkagg_add.yaml

PLAY [slx01] *********************************************************************************************************************************************

TASK [Create Port Channel] *******************************************************************************************************************************
changed: [slx01]

PLAY RECAP ***********************************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

Post-execution switch state:
```
SLX01# show port-channel summary
Flags:  D - Down                P - Up in port-channel (members)
        U - Up (port-channel)   * - Primary link in port-channel
        S - Switched            I - Insight Enabled
        M - Not in use. Min-links not met
===== =============== ========== ===============
Group Port-channel    Protocol   Member ports
===== =============== ========== ===============
34    Po 34   (U)     LACP       Eth 0/15 (D)
                                 Eth 0/16 (P)
200   Po 200  (D)     LACP       Eth 0/6 (D)
                                 Eth 0/7 (D)
SLX01# show run int eth 0/6
interface Ethernet 0/6
 channel-group 200 mode active type standard
 lacp timeout long
 shutdown
!
SLX01# show run int eth 0/7
interface Ethernet 0/7
 channel-group 200 mode active type standard
 lacp timeout long
 shutdown
!
SLX01#
```

1. Remove LAG:

[slx_linkagg_remove.yaml](./playbooks/slx_linkagg_remove.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - name: Create Port Channel
      slxos_linkagg:
        group: 200
        state: absent
```
Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_linkagg_remove.yaml -v
Using /home/extreme/.ansible.cfg as config file

PLAY [slx01] *********************************************************************************************************************************************

TASK [Remove Port-Channel 200] ***************************************************************************************************************************
changed: [slx01] => {"changed": true, "commands": ["no interface port-channel 200"]}

PLAY RECAP ***********************************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

Post-execution switch state:
```
SLX01# show run int eth 0/6
interface Ethernet 0/6
 shutdown
!
SLX01# show run int eth 0/7
interface Ethernet 0/7
 shutdown
!
SLX01# show port-channel sum
Flags:  D - Down                P - Up in port-channel (members)
        U - Up (port-channel)   * - Primary link in port-channel
        S - Switched            I - Insight Enabled
        M - Not in use. Min-links not met
===== =============== ========== ===============
Group Port-channel    Protocol   Member ports
===== =============== ========== ===============
34    Po 34   (U)     LACP       Eth 0/15 (D)
                                 Eth 0/16 (P)
SLX01#
```

## LLDP module - slxos_lldp

This module can be used to manage the status of LLDP on an SLX. This is a _declarative_ module. This means that we can define our desired state, and it will make changes as required. We do **not** need to provide CLI commands. This has a similar interface to [ios_lldp](http://docs.ansible.com/ansible/devel/modules/ios_lldp_module.html).

1. Disable LLDP:

[slx_lldp_absent.yaml](./playbooks/slx_lldp_absent.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - name: LLDP
      slxos_lldp:
        state: absent
```
Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_lldp_absent.yaml -v
Using /home/extreme/.ansible.cfg as config file

PLAY [slx01] *********************************************************************************************************************************************

TASK [LLDP] **********************************************************************************************************************************************
changed: [slx01] => {"changed": true, "commands": ["protocol lldp", "disable"]}

PLAY RECAP ***********************************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

2. Enable LLDP:

[slx_lldp_present.yaml](./playbooks/slx_lldp_present.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no

  tasks:
    - name: LLDP
      slxos_lldp:
        state: absent
```
Example output:
```shell
(venv) extreme@rancid:~/playbooks$ ansible-playbook slx_lldp_present.yaml -v
Using /home/extreme/.ansible.cfg as config file

PLAY [slx01] *********************************************************************************************************************************************

TASK [LLDP] **********************************************************************************************************************************************
changed: [slx01] => {"changed": true, "commands": ["protocol lldp", "no disable"]}

PLAY RECAP ***********************************************************************************************************************************************
slx01                      : ok=1    changed=1    unreachable=0    failed=0

(venv) extreme@rancid:~/playbooks$
```

## L2 Interface module - slxos_l2_interface

This module can be used to configure L2-related interface parameters - e.g. trunk or access mode, the VLAN(s) permitted, native VLAN, etc.

We do not need to provide the specific CLI. 

This module has a similar interface to the [ios_l2_interface](http://docs.ansible.com/ansible/devel/modules/ios_l2_interface_module.html) module.

1. Configure trunk and access interfaces
[slx_l2_interface.yaml](./playbooks/slx_l2_interface.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: configure access port
      slxos_l2_interface:
        name: 'Ethernet 0/6'
        mode: access
        access_vlan: 5

    - name: configure trunk port
      slxos_l2_interface:
        name: 'Ethernet 0/7'
        mode: trunk
        native_vlan: 10
        trunk_vlans: 5-10
```

The interfaces must be in 'switchport' mode first, and the VLANs must exist. The tasks will fail with an error if this is not done.

## L3 Interface module - slxos_l3_interface

This module can be used to configure L3-related interface parameters - e.g. IPv4 and IPv4 addresses.

We do not need to provide the specific CLI. This can be used to configure IP addresses on physical, VE and Port-channel interfaces.

This module has a similar interface to the [ios_l3_interface](http://docs.ansible.com/ansible/devel/modules/ios_l3_interface_module.html) module.

1. Configure IPv4 and IPv6 addresses:
[slx_l3_interface.yaml](./playbooks/slx_l3_interface.yaml):
```yaml
---
- hosts: slx01
  gather_facts: no
  
  tasks:
    - name: Configure IP Address
      slxos_l3_interface:
        name: 'Ethernet 0/8'
        ipv4: 192.0.2.8/24
        ipv6: 2001:db8::8/32
```

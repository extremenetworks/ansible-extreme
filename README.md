# Ansible + Extreme Networks

Examples and docs showing how to use Ansible with Extreme Networks switches and routers.

## Intro

[Ansible](https://www.ansible.com) is an Open Source configuration management system. It can be used for automating infrastructure and applications. It can also be used for automating network devices. This requires modules for your device. 

[MLXe](https://www.extremenetworks.com/product/mlx-series-router/) modules were added to Ansible 2.5. [SLX series](https://www.extremenetworks.com/product/slx-9850-router/) modules were added in Ansible 2.6. The first module for EXOS, `exos_command`, shipped in 2.6. Further modules were added in 2.7. Modules for [VDX](https://www.extremenetworks.com/product/vdx-6740/) and [VSP](https://www.extremenetworks.com/product/vsp-7400/) devices were added in 2.7. `httpapi` support for EXOS switches is in 2.8.

Further modules are being developed as Open Source modules, and you can follow along with the development.

As modules get developed for more Extreme platforms, we'll update this doc, and add examples to this repo.

## SLX

[SLX modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#slxos) shipped in Ansible 2.6. Currently available modules are:

* [slxos_command](http://docs.ansible.com/ansible/latest/modules/slxos_command_module.html) - Run arbitrary commands
* [slxos_config](http://docs.ansible.com/ansible/latest/modules/slxos_config_module.html) - Manage configuration sections
* [slxos_facts](http://docs.ansible.com/ansible/latest/modules/slxos_facts_module.html) - Gather facts
* [slxos_vlan](http://docs.ansible.com/ansible/latest/modules/slxos_vland_module.html) – Manage VLANs
* [slxos_interface](http://docs.ansible.com/ansible/latest/modules/slxos_interface_module.html) – Manage Interfaces
* [slxos_linkagg](http://docs.ansible.com/ansible/latest/modules/slxos_linkagg_module.html) – Manage link aggregation groups
* [slxos_lldp](http://docs.ansible.com/ansible/latest/modules/slxos_lldp_module.html) – Manage LLDP configuration
* [slxos_l2_interface](http://docs.ansible.com/ansible/latest/modules/slxos_l2_interface_module.html) – Manage L2 Interfaces
* [slxos_l3_interface](http://docs.ansible.com/ansible/latest/modules/slxos_l3_interface_module.html) – Manage L3 Interfaces

Check the [README](./slxos/README.md) in the [slxos](./slxos) folder for more details about what the modules can do, how to install them, and example playbooks.

## Ironware (MLXe)

[PaulQuack](https://github.com/PaulQuack) contributed [Ironware modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#ironware) to Ansible. These are included in Ansible 2.5.

Available modules:

* [ironware_command](http://docs.ansible.com/ansible/latest/modules/ironware_command_module.html) - Run arbitrary commands
* [ironware_config](http://docs.ansible.com/ansible/latest/modules/ironware_config_module.html) - Manage configuration sections
* [ironware_facts](http://docs.ansible.com/ansible/latest/modules/ironware_facts_module.html) - Collect facts

Click the links above to see docs on how to use those modules.

## EXOS

[Rafael Vencioneck](https://github.com/rdvencioneck) started EXOS modules. This was carried on by others, and [EXOS modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#exos) are now shipping.

Available modules:

* [exos_command](https://docs.ansible.com/ansible/devel/modules/exos_command_module.html) - Run arbitrary commands
* [exos_config](https://docs.ansible.com/ansible/devel/modules/exos_config_module.html) - Manage configuration sections
* [exos_facts](https://docs.ansible.com/ansible/devel/modules/exos_facts_module.html) - Collect facts

See [this](./GettingStartedwithAnsibleusingGNS3.md) guide to simulate Ansible on EXOS devices using GNS3

Click the links above to see docs on how to use those modules.

## NOS (VDX)

[NOS modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#nos) were included in Ansible 2.7.

Available modules:

* [nos_command](https://docs.ansible.com/ansible/devel/modules/nos_command_module.html) - Run arbitrary commands
* [nos_config](https://docs.ansible.com/ansible/devel/modules/nos_config_module.html) - Manage configuration sections
* [nos_facts](https://docs.ansible.com/ansible/devel/modules/nos_facts_module.html) - Collect facts

Click the links above to see docs on how to use those modules.


## VOSS

[VOSS modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#voss) were included in Ansible 2.7.

Available modules:

* [voss_command](https://docs.ansible.com/ansible/devel/modules/voss_command_module.html) - Run arbitrary commands
* [voss_config](https://docs.ansible.com/ansible/devel/modules/voss_config_module.html) - Manage configuration sections
* [voss_facts](https://docs.ansible.com/ansible/devel/modules/voss_facts_module.html) - Collect facts

Click the links above to see docs on how to use those modules.

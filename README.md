# Ansible + Extreme Networks

Examples and docs showing how to use Ansible with Extreme Networks switches and routers.

## Intro

[Ansible](https://www.ansible.com) is an Open Source configuration management system. It can be used for automating infrastructure and applications. It can also be used for automating network devices. This requires modules for your device. 

[MLXe](https://www.extremenetworks.com/product/mlx-series-router/) modules were added to Ansible 2.5. [SLX series](https://www.extremenetworks.com/product/slx-9850-router/) modules were added in Ansible 2.6. The first module for EXOS, `exos_command`, shipped in 2.6.

Further modules are being developed as Open Source modules, and you can follow along with the development.

As modules get developed for more Extreme platforms, we'll update this doc, and add examples to this repo.

## SLX

SLX modules shipped in Ansible 2.6. Currently available modules are:

* slxos\_command - Run arbitrary commands
* slxos\_config - Manage configuration sections
* slxos\_facts - Gather facts
* slxos\_vlan – Manage VLANs
* slxos\_interface – Manage Interfaces
* slxos\_linkagg – Manage link aggregation groups
* slxos\_lldp – Manage LLDP configuration
* slxos\_l2\_interface – Manage L2 Interfaces
* slxos\_l3\_interface – Manage L3 Interfaces

Check the [README](./slxos/README.md) in the [slxos](./slxos) folder for more details about what the modules can do, how to install them, and example playbooks.

## Ironware (MLXe)

[PaulQuack](https://github.com/PaulQuack) contributed Ironware modules to Ansible late last year. These are in included in Ansible 2.5. This is now GA.

Available modules:

* [ironware_command](http://docs.ansible.com/ansible/devel/modules/ironware_command_module.html) - Run arbitrary commands
* [ironware_config](http://docs.ansible.com/ansible/devel/modules/ironware_config_module.html) - Manage configuration sections
* [ironware_facts](http://docs.ansible.com/ansible/devel/modules/ironware_facts_module.html) - Collect facts

Click the links above to see docs on how to use those modules.

## EXOS

[Rafael Vencioneck](https://github.com/rdvencioneck) has begun work on EXOS modules. The first of these has shipped with Ansible 2.6.

Available modules:

* [exos_command](https://docs.ansible.com/ansible/devel/modules/exos_command_module.html) - Run arbitrary commands

See [this](./GettingStartedwithAnsibleusingGNS3.md) guide to simulate Ansible on EXOS devices using GNS3

Click the links above to see docs on how to use those modules.

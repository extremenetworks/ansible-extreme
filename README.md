# Ansible + Extreme Networks

Examples and docs showing how to use Ansible with Extreme Networks switches and routers.

## Intro

[Ansible](https://www.ansible.com) is an Open Source configuration management system. It can be used for automating infrastructure and applications. It can also be used for automating network devices. This requires modules for your device. 

[MLXe](https://www.extremenetworks.com/product/mlx-series-router/) modules are shipping in Ansible 2.5 (GA March 2018), and modules are now in active development for the [SLX series](https://www.extremenetworks.com/product/slx-9850-router/). These are Open Source, and you can follow along with the development, and try out modules as they are written. 

See below for more details.

As modules get developed for more Extreme platforms, we'll update this doca, and add examples to this repo.

## SLX

SLX modules are in active development. Currently available modules are:

* slxos\_command - Run arbitrary commands
* slxos\_config - Manage configuration sections
* slxos\_facts - Gather facts
* slxos\_vlan – Manage VLANs
* slxos\_interface – Manage Interfaces
* slxos\_linkagg – Manage link aggregation groups
* slxos\_lldp – Manage LLDP configuration

These modules are in active development, and have not yet been merged with Ansible upstream. Check out the [README](./slxos/README.md) in the [slxos](./slxos) folder for more details about what the modules can do, how to install them, and example playbooks.

## Ironware (MLXe)

[PaulQuack](https://github.com/PaulQuack) contributed Ironware modules to Ansible late last year. These are in included in the Ansible 2.5 branch. This is in RC2 now, and will almost certainly go GA in March 2018.

Available modules:

* [ironware_command](http://docs.ansible.com/ansible/devel/modules/ironware_command_module.html) - Run arbitrary commands
* [ironware_config](http://docs.ansible.com/ansible/devel/modules/ironware_config_module.html) - Manage configuration sections
* [ironware_facts](http://docs.ansible.com/ansible/devel/modules/ironware_facts_module.html) - Collect facts

Click the links above to see docs on how to use those modules.

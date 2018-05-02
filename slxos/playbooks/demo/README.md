## Demo Playbooks

This is a more complex set of plays, designed to configure multiple elements on multiple devices.

For a simple leaf-spine network of two leafs and one spine, it will do the following:

* Set device hostnames
* Set NTP and timezone
* Configure LLDP to advertise the management IP
* Enable the inter-switch links, and assign descriptions and IP Addresses
* Configure a Port-Channel on each Leaf switch
* Create VLANs and assign ports
* Configure BGP peering between devices.

This is all configured using group\_vars and host\_vars.

To run this playbook, configure your hosts, group\_vars and host\_vars as required, then run
`ansible-playbook site.yml`

There is also a set of "Check" plays that will:
* Check that NTP is synchronized
* Check that LLDP shows devices cabled as expected
* Grab the `show ip bgp summary` output from each device, and publish it.

To run this, use `ansible-playbook check.yml`. Note that it uses values from host\_vars to specify which LLDP neighbors to look for.

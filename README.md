
# Network as Code Demo with Nexus Dashboard

This demo is based on the [example repository](https://github.com/netascode/ansible-dc-vxlan-example) running the [NetAsCode DC VXLAN Ansible Collection](https://galaxy.ansible.com/ui/repo/published/cisco/nac_dc_vxlan/). 

It builds a 3-tier spine and leaf topology based on Nexus 9300v that can run either on CML or Containerlab. 

## Running the Playbook

Run the playbook using the following command:

```bash
ansible-playbook -i inventory.yaml vxlan.yaml
```

The data in `host_vars/nac-fabric1` will be processed by the main `vxlan.yaml` playbook and do the following:

* Create a fabric called `nac-fabric1` using the data from `fabric.nac.yaml` and `underlay.nac.yaml` files.
* Add 1 Superspine, 2 Spines, and 4 Leaf switches using the data defined in the `topology_switches.nac.yaml` file.
* Add 3 VRFs and 3 Networks using the data defined in `vrfs.nac.yaml` and `networks.nac.yaml` files.

### Tips

The following tags can be used to selectively execute stages within the `cisco.nac_dc_vxlan.dtc.create` and `cisco.nac_dc_vxlan.dtc.remove` roles:

`cr` stands for `create_role` and `rr` stands for `remove_role`

* cr_manage_fabric
* cr_manage_switches
* cr_manage_vpc_peers
* cr_manage_interfaces
* cr_manage_vrfs_networks
* cr_manage_policy
* cr_manage_links
* cr_manage_edge_connections
* rr_manage_vpc_peers
* rr_manage_interfaces
* rr_manage_networks
* rr_manage_vrfs
* rr_manage_switches
* rr_manage_links
* rr_manage_edge_connections
* rr_manage_policy

```bash
# Example
ansible-playbook -i inventory.yaml vxlan.yaml --tags cr_manage_vrfs_networks
```
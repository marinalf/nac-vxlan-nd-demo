
# Network as Code Demo with Nexus Dashboard

This demo is based on the [example repository](https://github.com/netascode/ansible-dc-vxlan-example) running the Network as Code VXLAN EVPN collection. It builds a 3-tier spine and leaf topology based on Nexus 9300v that can run either on CML or Containerlab. 

## Running the Playbook

Run the playbook using the following command:

```bash
ansible-playbook -i inventory.yaml vxlan.yaml
```

The data in `host_vars/nac-fabric1` will be processed by the main `vxlan.yaml` playbook and do the following:

* Create a fabric called `nac-fabric1` using the data from `fabric.nac.yaml` and `underlay.nac.yaml` files.
* Add 1 Superspine, 2 Spines and 4 Leaf devices using the data defined in the `topology_switches.nac.yaml` file.
* Add 3 VRFs and 3 Networks using the data defined in `vrfs.nac.yaml` and `networks.nac.yaml` files.

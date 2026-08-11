# Network as Code with Nexus Dashboard

Builds a NX-OS EVPN Fabric (`vxlan-fabric`: 2 spines, 4 leaves, eBGP_VXLAN, Same-Tier-AS) using the [`cisco.nac_dc_vxlan`](https://galaxy.ansible.com/ui/repo/published/cisco/nac_dc_vxlan/) collection, based on the [NetAsCode Example Repository](https://github.com/netascode/ansible-dc-vxlan-example).

The base fabric (`host_vars/vxlan-fabric/*.nac.yaml`) preconfigures interfaces for testing endpoints for leaf 1 and 2 on `Ethernet1/7`. `host_vars/vxlan-fabric/policy-cilium-evpn/` is a separate, self-contained example layering on `Ethernet1/6` and the BGP freeform needed to peer the fabric with a kind cluster running Cilium CNI (EVPN/Private Networks).

## CML Topology (Optional)

[`cml-topology/`](cml-topology/) builds the underlying CML lab itself (switches, kind-host, endpoints, links) from code via the MCP server from [`cml-mcp`](https://github.com/xorrkaz/cml-mcp). Optional, and a separate/complementary piece to this NaC config, only needed if you also want to automate the CML layer this fabric config runs against.

## Prerequisites

```bash
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yaml
cp secrets.sh.example secrets.sh   # fill in real ND/switch credentials
source secrets.sh
```

**Gather each switch's serial number first.** Update `topology_switches.nac.yaml` and `fix_proxy_arp.yml` with the current values before running the playbook.

## Running the Playbook

**Option 1 - full deploy, no tags:**

```bash
ansible-playbook -i inventory.yaml vxlan.yaml
```

Builds and deploys everything in **one pass**: fabric, switches, VRFs, networks, interfaces, and the Cilium-facing policy config (`policy-cilium-evpn/`). 

**IMPORTANT:** for the freeform policy, adjust the loopback address to match the allocated IPs during fabric build (if needed) and re-deploy. 

```bash
ansible-playbook -i inventory.yaml vxlan.yaml --tags cr_manage_policy,role_deploy
```

**Option 2 - selective/staged deploy via tags:**

```bash
ansible-playbook -i inventory.yaml vxlan.yaml --tags cr_manage_fabric,cr_manage_switches
```

Builds only the specific resources named by the tags. Any `cr_manage_*` tag only creates/stages the resource on Nexus Dashboard, it does not push config to the switches. A separate `role_deploy` pass is required to deploy:

```bash
ansible-playbook -i inventory.yaml vxlan.yaml --tags role_deploy
```

Run `role_deploy` after every stage that changes something, or add it together in the same run. 

### Recommended staged order

```
cr_manage_fabric
cr_manage_switches
role_deploy          # day 0 - underlay
cr_manage_vrfs
cr_manage_networks
cr_manage_interfaces
role_deploy           # day 1 - overlay
cr_manage_policy
role_deploy           # day 2 - Cilium-facing BGP freeform
```

In this example (not using vPC peering), the interfaces config must come after VRFs/networks. The `eth1/7` access ports reference VLANs that only exist once the network objects are created.

**IMPORTANT:** Before running `cr_manage_policy`, double-check the loopback addresses on `policy-cilium-evpn/policy.nac.yaml`, match with swiches allocation, and update accordingly.

### Tags

`cr` stands for `create_role`, `rr` for `remove_role`:

* cr_manage_fabric
* cr_manage_switches
* cr_manage_vpc_peers
* cr_manage_interfaces
* cr_manage_vrfs
* cr_manage_networks
* cr_manage_policy
* cr_manage_links
* cr_manage_edge_connections
* rr_manage_interfaces
* rr_manage_networks
* rr_manage_vrfs
* rr_manage_switches
* rr_manage_vpc_peers
* rr_manage_links
* rr_manage_edge_connections
* rr_manage_policy

There is no `rr_manage_fabric` tag, fabric deletion is not exposed by this collection. Fabric deletion is done in Nexus Dashboard's UI.

### Proxy ARP fix

`ip proxy-arp` on Vlan100/Vlan200 is required for Cilium's BGP-unnumbered peer to see ARP replies for the host subnet.  Run this once, after the main playbook:

```bash
ansible-playbook -i inventory.yaml fix_proxy_arp.yml
```

### Reference

- [NetAsCode DC VXLAN Example](https://github.com/netascode/ansible-dc-vxlan-example): the example repository this was structured from
- [NetAsCode Docs](https://netascode.cisco.com/docs/data_models/vxlan/overview/): official data model reference for every field used in this folder
- [NetAsCode Ansible Collection](https://galaxy.ansible.com/ui/repo/published/cisco/nac_dc_vxlan/): the Ansible collection this is built on
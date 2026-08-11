# CML Topology via MCP

Builds the CML lab itself (nodes + links, mgmt IP, and day-0 bootstrap) using the [`cml-mcp`](https://github.com/xorrkaz/cml-mcp) server, so the underlying VMs/switches this whole lab runs on are reproducible from code.

## What's in the topology

- `Spine 1`, `Spine 2`, `Leaf 1`–`Leaf 4`: Nexus 9000v switches, day-0 config is mgmt IP + gateway + credentials only
- `Kind-Node`: the Ubuntu VM that runs the `kind` cluster
- `red-endpoint`, `blue-endpoint`: single-homed test hosts on Leaf 1/Leaf 2
- `Management Switch`: L2 switch aggregating every node's mgmt interface
- `Nexus Dashboard 4.2 Mgmt`: external connector bridging the mgmt network out to Nexus Dashboard

Day-0 config for the `Kind-Node`, `red-endpoint`, and `blue-endpoint` are under `configs/`.

## Setup

```bash
brew install uv   # provides uvx, used to run the cml-mcp server
```

Register the MCP server in the repo-root `.mcp.json`:

```json
{
  "mcpServers": {
    "cml": {
      "command": "uvx",
      "args": ["cml-mcp"]
    }
  }
}
```

Credentials go in a `.env` file at the repo root in this example:

```bash
CML_URL=https://<cml-ip> 
CML_USERNAME=<username>
CML_PASSWORD=<password>
CML_VERIFY_SSL=false
```

Then start the server: open `.mcp.json`, find the `cml` entry, and click **Start**. It should log `Discovered X tools` once connected.

## Building / Removing the topology

Once the server is connected, these are sample prompts to build and tear down this lab:

- **Build the lab:** "Create a CML lab from `cml-topology/topology.yaml`, but don't start it yet."
- **Start the lab's nodes:** "Start the lab titled 'NX-OS Fabric with Cilium built via MCP'."
- **Stop a lab's nodes without deleting it:** "Stop the lab titled '<lab title>'."
- **Delete a lab entirely:** "Delete the CML lab titled '<lab title>'."
- **Check what is running:** "List all CML labs and their state" / "List the nodes in lab `<id>` and tell me which are powered on."

The underlying tool calls (`create_full_lab_topology`, `start_cml_lab`, `stop_cml_lab`, `delete_cml_lab`, `get_cml_labs`, `get_nodes_for_cml_lab`) come from the `cml-mcp` server itself, these prompts are just the natural-language requests that route to them.

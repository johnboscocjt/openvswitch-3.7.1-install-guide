# Open vSwitch 3.7.1 - Usage Guide

Here's a practical guide for using Open vSwitch after installation.

---

## Basic Open vSwitch Commands Guide

### 1. Viewing Switch Status

```bash
# Show all bridges and ports
sudo ovs-vsctl show

# List all bridges
sudo ovs-vsctl list-br

# Show detailed bridge information
sudo ovs-ofctl show br0
```

### 2. Bridge Management

```bash
# Create a bridge
sudo ovs-vsctl add-br br0

# Delete a bridge
sudo ovs-vsctl del-br br0

# Rename a bridge
sudo ovs-vsctl br-rename br0 br1
```

### 3. Port Management

```bash
# Add a physical interface to bridge
sudo ovs-vsctl add-port br0 eth0

# Add a virtual interface (internal port)
sudo ovs-vsctl add-port br0 vport0 -- set Interface vport0 type=internal

# Remove a port
sudo ovs-vsctl del-port br0 eth0

# List all ports on a bridge
sudo ovs-vsctl list-ports br0
```

### 4. VLAN Configuration

```bash
# Create VLAN 100 on bridge br0
sudo ovs-vsctl add-port br0 vlan100 tag=100 -- set Interface vlan100 type=internal

# Add a trunk port (allows multiple VLANs)
sudo ovs-vsctl add-port br0 eth0 trunk=100,200,300

# Add an access port (single VLAN)
sudo ovs-vsctl add-port br0 eth1 tag=100
```

### 5. Bonding / Link Aggregation

```bash
# Create bond with two interfaces
sudo ovs-vsctl add-bond br0 bond0 eth0 eth1

# Set bond mode (balance-slb, active-backup, balance-tcp)
sudo ovs-vsctl set Port bond0 bond_mode=balance-slb

# Check bond status
sudo ovs-appctl bond/show
```

### 6. OpenFlow Rules

```bash
# View current flows
sudo ovs-ofctl dump-flows br0

# Add a flow (forward all traffic)
sudo ovs-ofctl add-flow br0 "actions=normal"

# Add flow to drop specific traffic
sudo ovs-ofctl add-flow br0 "dl_src=00:11:22:33:44:55, actions=drop"

# Add flow to forward to specific port
sudo ovs-ofctl add-flow br0 "in_port=1, actions=output=2"

# Add flow with VLAN tag
sudo ovs-ofctl add-flow br0 "dl_vlan=100, actions=output=3"

# Delete all flows
sudo ovs-ofctl del-flows br0
```

### 7. QoS / Traffic Shaping

```bash
# Create QoS policy
sudo ovs-vsctl set Port eth0 qos=@newqos -- \
  --id=@newqos create QoS type=linux-htb queues=0=@q0 -- \
  --id=@q0 create Queue other-config:max-rate=10000000

# Apply rate limit (10 Mbps)
sudo ovs-vsctl set interface eth0 ingress_policing_rate=10000
sudo ovs-vsctl set interface eth0 ingress_policing_burst=1000
```

### 8. Mirroring / SPAN

```bash
# Mirror all traffic from eth0 to eth2
sudo ovs-vsctl -- set Bridge br0 mirrors=@m \
  -- --id=@eth0 get Port eth0 \
  -- --id=@eth2 get Port eth2 \
  -- --id=@m create Mirror name=mymirror select-dst-port=@eth0 select-src-port=@eth0 output-port=@eth2

# Remove mirror
sudo ovs-vsctl clear Bridge br0 mirrors
```

### 9. Controller Configuration (SDN)

```bash
# Set OpenFlow controller (local or remote)
sudo ovs-vsctl set-controller br0 tcp:192.168.1.100:6633

# Set controller with fail-open mode
sudo ovs-vsctl set-fail-mode br0 standalone

# Set secure mode (no fallback)
sudo ovs-vsctl set-fail-mode br0 secure

# View current controller
sudo ovs-vsctl get-controller br0

# Remove controller
sudo ovs-vsctl del-controller br0
```

### 10. Troubleshooting Commands

```bash
# Check OVS version
ovs-vswitchd --version

# View OVS logs
sudo tail -f /usr/local/var/log/openvswitch/ovs-vswitchd.log

# Check database contents
sudo ovsdb-client dump

# Show interface statistics
sudo ovs-vsctl get Interface eth0 statistics

# Check bridge connectivity
sudo ovs-appctl fdb/show br0

# Debug OVS internal state
sudo ovs-appctl vlog/list

# Increase log verbosity
sudo ovs-appctl vlog/set dbg
```

### 11. Network Namespace Integration (Containers/VMs)

```bash
# Create network namespace
sudo ip netns add ns1

# Create veth pair
sudo ip link add veth0 type veth peer name veth1

# Move one end to namespace
sudo ip link set veth1 netns ns1

# Add veth0 to OVS bridge
sudo ovs-vsctl add-port br0 veth0

# Configure IP in namespace
sudo ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth1
sudo ip netns exec ns1 ip link set veth1 up
sudo ip netns exec ns1 ip link set lo up

# Test connectivity
sudo ip netns exec ns1 ping 10.0.0.2
```

### 12. Performance Monitoring

```bash
# Show interface counters
sudo ovs-vsctl list interface

# Real-time port statistics
watch -n 2 'sudo ovs-ofctl dump-ports br0'

# Show datapath statistics
sudo ovs-dpctl show

# Monitor flow statistics
sudo ovs-ofctl dump-flows br0 | grep -v "n_packets=0"
```

## Example: Simple Virtual Network Setup

```bash
# Create bridge
sudo ovs-vsctl add-br br-int

# Add physical uplink
sudo ovs-vsctl add-port br-int eth0

# Create two internal ports for VMs/containers
sudo ovs-vsctl add-port br-int vm1 -- set Interface vm1 type=internal
sudo ovs-vsctl add-port br-int vm2 -- set Interface vm2 type=internal

# Assign IPs to internal ports
sudo ip addr add 192.168.100.1/24 dev vm1
sudo ip addr add 192.168.100.2/24 dev vm2
sudo ip link set vm1 up
sudo ip link set vm2 up

# Add simple flow rule
sudo ovs-ofctl add-flow br-int "actions=normal"

# Verify
sudo ovs-vsctl show
ping -c 3 192.168.100.2
```

## Persistent Configuration (Save/Restore)

```bash
# Save current configuration
sudo ovs-vsctl get Open_vSwitch . db_version > ovs-backup.txt
sudo ovs-vsctl list bridge > bridges-backup.txt

# Restore (save as script)
sudo ovs-vsctl add-br br0
# ... add ports and configuration
```

## Quick Reference Card

| Task | Command |
|------|---------|
| Create bridge | `ovs-vsctl add-br NAME` |
| Delete bridge | `ovs-vsctl del-br NAME` |
| Add port | `ovs-vsctl add-port BRIDGE PORT` |
| Remove port | `ovs-vsctl del-port BRIDGE PORT` |
| Show config | `ovs-vsctl show` |
| View flows | `ovs-ofctl dump-flows BRIDGE` |
| Add flow | `ovs-ofctl add-flow BRIDGE "rules"` |
| Set controller | `ovs-vsctl set-controller BRIDGE tcp:IP:PORT` |

---

This guide covers most common use cases. Save it as `USAGE.md` in your GitHub repository alongside the installation guide.

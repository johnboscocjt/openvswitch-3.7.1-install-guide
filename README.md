# GitHub Repository: Open vSwitch 3.7.1 Installation Guide

Here's a clean, working guide for your GitHub repository. I've removed all the errors and failed attempts, keeping only what actually worked.

---

## Repository Name: `openvswitch-3.7.1-install-guide`

### README.md

```markdown
# Open vSwitch 3.7.1 Installation Guide

Complete step-by-step guide to install Open vSwitch 3.7.1 from source on Linux.

## System Requirements
- Linux distribution (Ubuntu/Debian/RHEL/Rocky/AlmaLinux)
- 4+ CPU cores recommended
- Root/sudo access

## Installation Steps

### 1. Install Build Dependencies

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install -y build-essential autoconf automake libtool libssl-dev python3 python3-dev
```

**RHEL/Rocky/AlmaLinux:**
```bash
sudo dnf install -y gcc make python3-devel openssl-devel autoconf automake libtool
```

### 2. Download and Extract
```bash
wget https://www.openvswitch.org/releases/openvswitch-3.7.1.tar.gz
tar -xzvf openvswitch-3.7.1.tar.gz
cd openvswitch-3.7.1
```

### 3. Configure and Build
```bash
./boot.sh
./configure
make
```

### 4. Install
```bash
sudo make install
```

### 5. Initialize Database
```bash
sudo mkdir -p /usr/local/etc/openvswitch
sudo mkdir -p /usr/local/var/run/openvswitch
sudo ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema
```

### 6. Start Open vSwitch Services
```bash
sudo ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock --pidfile --detach
sudo ovs-vsctl --no-wait init
sudo ovs-vswitchd --pidfile --detach
```

### 7. Verify Installation
```bash
sudo ovs-vsctl show
```

**Expected Output:**
```
7a227aac-136f-4f78-8151-1e8612ddee4d
```

### 8. Create Test Bridge
```bash
sudo ovs-vsctl add-br br0
sudo ovs-vsctl show
```

**Expected Output:**
```
7a227aac-136f-4f78-8151-1e8612ddee4d
    Bridge br0
        Port br0
            Interface br0
                type: internal
```

### 9. Configure systemd (Automatic Startup)

Create `/etc/systemd/system/ovsdb-server.service`:
```ini
[Unit]
Description=Open vSwitch Database Server
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/var/run/openvswitch/ovsdb-server.pid
ExecStart=/usr/local/sbin/ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock --pidfile --detach
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Create `/etc/systemd/system/ovs-vswitchd.service`:
```ini
[Unit]
Description=Open vSwitch Daemon
After=ovsdb-server.service
Requires=ovsdb-server.service

[Service]
Type=forking
PIDFile=/usr/local/var/run/openvswitch/ovs-vswitchd.pid
ExecStartPre=/usr/local/bin/ovs-vsctl --no-wait init
ExecStart=/usr/local/sbin/ovs-vswitchd --pidfile --detach
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable services:
```bash
sudo systemctl daemon-reload
sudo systemctl enable ovsdb-server ovs-vswitchd
sudo systemctl start ovsdb-server ovs-vswitchd
```

### 10. Verify Services
```bash
sudo systemctl status ovsdb-server ovs-vswitchd
```

**Expected Output:**
```
● ovsdb-server.service - Open vSwitch Database Server
     Active: active (running)
● ovs-vswitchd.service - Open vSwitch Daemon
     Active: active (running)
```

## Uninstall (if needed)
```bash
sudo systemctl stop ovsdb-server ovs-vswitchd
sudo systemctl disable ovsdb-server ovs-vswitchd
sudo rm -rf /usr/local/etc/openvswitch
sudo rm -rf /usr/local/var/run/openvswitch
```

## License
MIT License
```

---

### Full Terminal Success Output (for reference in your repo)

```bash
$ sudo ovs-vsctl show
7a227aac-136f-4f78-8151-1e8612ddee4d

$ sudo ovs-vsctl add-br br0

$ sudo ovs-vsctl show
7a227aac-136f-4f78-8151-1e8612ddee4d
    Bridge br0
        Port br0
            Interface br0
                type: internal

$ sudo systemctl status ovsdb-server
● ovsdb-server.service - Open vSwitch Database Server
     Active: active (running)

$ sudo systemctl status ovs-vswitchd
● ovs-vswitchd.service - Open vSwitch Daemon
     Active: active (running)
```

---

This clean version includes only what worked successfully. Push it to GitHub under `johnboscocjt/openvswitch-3.7.1-install-guide`.

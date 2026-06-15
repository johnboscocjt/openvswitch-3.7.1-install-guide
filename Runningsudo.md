# Permission Denied Error Fix

You're getting a permission denied error because `ovs-vsctl` needs **root/sudo** privileges to access the OVS database socket.

## Quick Fix

Always use `sudo` with OVS commands:

```bash
sudo ovs-vsctl show
```

## Why This Happens

The OVS database socket at `/usr/local/var/run/openvswitch/db.sock` is owned by `root` for security. Regular users cannot access it.

## Solution Options

### Option 1: Use sudo (Recommended)
```bash
# Every OVS command needs sudo
sudo ovs-vsctl show
sudo ovs-vsctl add-br br0
sudo ovs-vsctl list-br
```

### Option 2: Add Your User to the Proper Group
```bash
# Check current ownership
ls -la /usr/local/var/run/openvswitch/db.sock

# Create openvswitch group and add your user
sudo groupadd openvswitch
sudo usermod -aG openvswitch $USER

# Change socket group ownership
sudo chgrp openvswitch /usr/local/var/run/openvswitch/db.sock
sudo chmod 660 /usr/local/var/run/openvswitch/db.sock

# Log out and back in, OR start a new shell
newgrp openvswitch
```

### Option 3: Quick Test After Fix
```bash
# This should work now
sudo ovs-vsctl show
```

## Expected Output
```
7a227aac-136f-4f78-8151-1e8612ddee4d
    Bridge br0
        Port br0
            Interface br0
                type: internal
```

## Verify Your Services Are Running
```bash
sudo systemctl status ovsdb-server ovs-vswitchd
```

**Bottom line:** Just add `sudo` before every `ovs-vsctl` command - that's the standard way to run it.

The error you are seeing means your current user does not have permission to access the Open vSwitch database socket.
## Why This Happens
The ovs-vsctl command needs to communicate with the OVS database daemon (ovsdb-server) via a local Unix domain socket (db.sock). By default, this socket is restricted to the root user or users in a specific administrative group.
## How to Fix It
1. Run with sudo (Quickest Fix)
The most direct way to run administrative Open vSwitch commands is by prefixing them with sudo:

sudo ovs-vsctl show

2. Check Service Status
If sudo still gives you an error, the Open vSwitch service might not be running at all. Check its status using your system's init system:

sudo systemctl status openvswitch-switch

(If it is stopped, start it with sudo systemctl start openvswitch-switch)
3. Check File Permissions
If you need to know exactly who owns the socket file, you can inspect it with:

ls -l /usr/local/var/run/openvswitch/db.sock



# virt-access -- Access control for libvirtd

This accomplishes the following completely independent tasks:

1. Allow users to connect to libvirtd securely through SSH
   without needing to give them unrestricted shell access.

   This is done via an SSH `authorized_keys` forced command and a
   script, `virt-access` which contains a set of whitelisted commands.

2. Grant individual users access to invidiual libvirt domains, so that
   they can use standard tools like virt-manager and virsh to perform
   tasks like starting and stopping VMs, or accessing the graphical
   and serial console, but make no changes to the VM configuration.

   This is done via PolicyKit. The rules allow users in a specific
   group (`vm-owners`) limited access to VMs based on the name of
   their libvirt domain.


## Setup

Put these scripts in `/opt/virt-access` and run:

```
git clone -b stable https://github.com/fxkr/virt-access.git /opt/virt-access

chown root:root /opt/virt-access/{virt-access,23-virt-access.rules}
chmod 755 /opt/virt-access/virt-access,
chmod 644 /opt/virt-access/23-virt-acess.rules

sed -i 's/^#\(access_drivers = \[ "polkit" \]\)$/\1/' /etc/libvirt/libvirtd.conf
groupadd vm-owners
ln -s /opt/virt-access/23-virt-access.rules /etc/polkit-1/rules.d/
```


## Creating users

```
adduser --shell=/usr/sbin/nologin --create-home $username
mkdir ~$username/.ssh
echo 'no-agent-forwarding,no-port-forwarding,no-pty,no-user-rc,no-X11-forwarding,command="/opt/virt-access/virt-access" ssh-rsa ...' >> ~$username/.ssh/authorized_keys
chown $username:$username ~$username/.ssh{,/.authorized_keys}
```


## Granting access

Add '$username-' to the start of the domain name or '-$username' to the end of it.


## Debugging

Log messages will end up in `/var/log/secure`.

Changing rules files makes PolKit reload them.
Unfortunately, this doesn't with for symlinks.
As a workaround, you can just rename (`mv`) the symlink.


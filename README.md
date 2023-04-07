# Coral-in-LXC
How to pass or share a Google Coral M.2 to an LXC container in Proxmox

## Create the LXC
First, create a new LXC. Select the advanced box and then deselect the unprivileged box. Move through the rest of setup normally. When finished, go to the server shell and do:
```
nano /etc/pve/nodes/(NODE-NAME)/lxc/(CONTAINER-ID ex. "100").conf
```

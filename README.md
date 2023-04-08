# Coral-in-LXC
How to pass or share a Google Coral M.2 to an LXC container in Proxmox

## Create the LXC
First, create a new LXC. Select the advanced box and then deselect the unprivileged box. Move through the rest of setup normally. When finished, go to the server shell and edit the config of the container:
```
nano /etc/pve/nodes/(NODE-NAME)/lxc/(CONTAINER-ID ex. "100").conf
```
Add in:
```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.cgroup2.devices.allow: c 189:* rwm
lxc.apparmor.profile: unconfined
lxc.cgroup2.devices.allow: a
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0
lxc.mount.entry: /dev/apex_0 dev/apex_0 none bind,optional,create=file 0, 0
lxc.cap.drop:
lxc.mount.auto: cgroup:rw
```
Then save and exit. 

## Add the Coral Drivers

Still in the shell, follow the instructions from the Google Coral Get Started guide (modified to line up with the experience with Proxmox): 
 
Add the Debian package repository: 
```
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

sudo apt-get update
```
Then install the PCIE driver and TPU runtime packages
```
apt-get install gasket-dkms libedgetpu1-std
```
Reboot the node, then run:
```
lspci -nn | grep 089a
```
It should return something like this:
```
3d:00.0 System peripheral [0880]: Global Unichip Corp. Coral Edge TPU [1ac1:089a]
```
Then verify that the driver loaded:
```
ls /dev/apex_0
```
That should return:
```
/dev/apex_0
```
Now it should be functioning in the container. If not, install the PVE headers reinstall the coral drivers?

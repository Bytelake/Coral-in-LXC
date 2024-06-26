# Coral-in-LXC
How to pass or share a Google Coral M.2 to an LXC container in Proxmox

## Create the LXC
First, create a new LXC. Select the advanced box and then deselect the unprivileged box. Move through the rest of setup normally. When finished, go to the server shell and edit the config of the container:
```
nano /etc/pve/nodes/(NODE-NAME)/lxc/(CONTAINER-ID ex. "100").conf
```
Add in (This passes through your Intel iGPU as well):
```
features: nesting=1
lxc.cgroup2.devices.allow: c 226:0 rwm #igpu
lxc.cgroup2.devices.allow: c 226:128 rwm #igpu
lxc.cgroup2.devices.allow: c 29:0 rwm #coral
lxc.cgroup2.devices.allow: c 189:* rwm #coral
lxc.apparmor.profile: unconfined #unbreaks docker for reasons unknown
lxc.cgroup2.devices.allow: a
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0 #igpu
lxc.mount.entry: /dev/apex_0 dev/apex_0 none bind,optional,create=file 0, 0 #coral
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

apt-get update
```
Then install pve-headers and the PCIE driver and TPU runtime packages
```
apt install pve-headers-$(uname -r)
 
apt-get install gasket-dkms libedgetpu1-std
```
**Reboot the node** (The entire system), then run:
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
That's it! It's now ready for whatever you want to use it for, though I assume probably Frigate.

- [PNetLab](#pnetlab)
  - [setup](#setup)
  - [ishare2](#ishare2)
  - [images](#images)
  - [convert .ova to .qcow2](#convert-ova-to-qcow2)
  - [kengs](#kengs)
    - [rescue root password if forgot](#rescue-root-password-if-forgot)
    - [install pnetlab on bare mental, failed](#install-pnetlab-on-bare-mental-failed)
    - [~~store router/switch images on host~~](#store-routerswitch-images-on-host)
      - [fix permission issue when using `9p`](#fix-permission-issue-when-using-9p)
- [eve-ng](#eve-ng)
  - [install](#install)
  - [share host folder with vm](#share-host-folder-with-vm)

# PNetLab 

* forked from eve-ng
* exam lab setup shared by others
  * [PNetLab store](https://user.pnetlab.com/store/labs/view)

## setup

* run [PNetLab](https://pnetlab.com/pages/download) as virtual machine
* login to vm
  * root/pnet
  * ssh root@192.168.122.68
* login to pnet via https
  * admin/pnet
* log file
  * `/opt/unetlab/data/Logs/unl_wrapper.txt`

## ishare2

* [ishare2](https://github.com/ishare2-org/ishare2-cli) to download images

```
# list local labs, downloaded from pnetlab
ishare2 labs
# download needed images
ishare2 labs 1
```

## images

* https://labhub.eu.org
  * https://labhub.eu.org/UNETLAB%20II/addons/qemu/
  * https://drive.labhub.eu.org/

## convert .ova to .qcow2

* unrar .ova with `tar xvf`
* there will be a .vmdk file, convert it to .qcow2
* `qemu-img convert -f vmdk -O qcow2 image-disk1.vmdk image.qcow2`

## kengs

### rescue root password if forgot

* use `virsh-rescue`

```
# Install virt-rescue (Part of libguestfs)
sudo apt update
sudo apt install libguestfs-tools

sudo virt-rescue -a <path-to-qcow2-file>

# if disk is standard parition
mount /dev/sda1 /mnt

# if disk use lvm (most ubuntu installation does so)
vgscan
vgchange -ay
lvdisplay
# mount /dev/<volume-group-name>/<logical-volume-name> /mnt
mkdir /mnt
mount /dev/ubuntu-vg/ubuntu-lv /mnt

# Change the Root Password
chroot /mnt
passwd
exit
```

### install pnetlab on bare mental, failed

* https://www.pnetlab.com/pages/documentation?slug=install-bare-metal
* add pnetlab apt repo
`deb [trusted=yes] http://repo.pnetlab.com ./`
```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
apt-get update
apt-get purge netplan.io
apt-get install pnetlab -y
```

* 2024-11-19 16:21:19: failed on ubuntu 20.04 (dell2)

```
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 pnetlab : PreDepends: pnetlab-guacamole but it is not going to be installed
           Depends: php7.2-zip but it is not installable
           Depends: pnetlab-guacamole but it is not going to be installed
           Depends: ntp but it is not going to be installed
           Depends: pnetlab-qemu but it is not going to be installed
           Depends: pnetlab-docker but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

* remove `netplan.io`
```
The following packages will be REMOVED:
  cloud-init* netplan.io* network-manager* network-manager-config-connectivity-ubuntu* network-manager-gnome* network-manager-openvpn*
  network-manager-openvpn-gnome* network-manager-pptp* network-manager-pptp-gnome* ubuntu-minimal* ubuntu-server-minimal*
```

### ~~store router/switch images on host~~

* for kernel < 5.4, use `9p`
```
mount -t 9p -o trans=virtio,version=9p2000.L /share_tag /opt/unetlab/addons/qemu
```

#### fix permission issue when using `9p`

* https://www.linux-kvm.org/page/9p_virtio

![](img/2024-10-03-11-04-43.png)

> by the way, "Squash" references a 9p security model. It means "none" and is the most permissive setting for this context

LZ: doesn't work, web cannot access the mounted dir, should try samba/nfs

---

# eve-ng

* pnetlab is more suitable for exam
* an ubuntu system with web gui

## install

* follow the [cookbook](https://www.eve-ng.net/index.php/documentation/community-cookbook/)
* it is said best run eve-ng on bare-mental
* but it seems eve-ng vm with nested router vm works just fine, and much easier

## share host folder with vm

* put images on host disk
* share it with vm
* for kernel >=5.4, use `virtiofs`
```
mount -t virtiofs /share_tag /opt/unetlab/addons/qemu
```

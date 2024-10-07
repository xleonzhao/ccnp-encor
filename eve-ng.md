- [basics](#basics)
- [install](#install)
- [where to download images](#where-to-download-images)
  - [share host folder with vm](#share-host-folder-with-vm)
- [PNetLab](#pnetlab)
  - [log file](#log-file)
  - [~~store router/switch images on host~~](#store-routerswitch-images-on-host)
    - [fix permission issue when using `9p`](#fix-permission-issue-when-using-9p)

# basics

* an ubuntu system with web gui

# install

* follow the [cookbook](https://www.eve-ng.net/index.php/documentation/community-cookbook/)
* it is said best run eve-ng on bare-mental
* but it seems eve-ng vm with nested router vm works just fine, and much easier

# where to download images

* https://labhub.eu.org/UNETLAB%20II/addons/qemu/

## share host folder with vm

* put images on host disk
* share it with vm
* for kernel >=5.4, use `virtiofs`
```
mount -t virtiofs /share_tag /opt/unetlab/addons/qemu
```

# PNetLab

* lab setup by others for exams
* forked from eve-ng
* [PNetLab](https://user.pnetlab.com/store/labs/view) store to find interesting lab
* [ishare2](https://github.com/ishare2-org/ishare2-cli) to download images

```
# list local labs, downloaded from pnetlab
ishare2 labs
# download needed images
ishare2 labs 1
```

## log file

* `/opt/unetlab/data/Logs/unl_wrapper.txt`

## ~~store router/switch images on host~~

* for kernel < 5.4, use `9p`
```
mount -t 9p -o trans=virtio,version=9p2000.L /share_tag /opt/unetlab/addons/qemu
```

### fix permission issue when using `9p`

* https://www.linux-kvm.org/page/9p_virtio

![](img/2024-10-03-11-04-43.png)

> by the way, "Squash" references a 9p security model. It means "none" and is the most permissive setting for this context

LZ: doesn't work, web cannot access the mounted dir, should try samba/nfs


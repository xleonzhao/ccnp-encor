# setup

* run [PNetLab](https://pnetlab.com/pages/download) as virtual machine
* login to vm and install [ishare2](https://github.com/ishare2-org/ishare2-cli)

## convert .ova to .qcow2

* unrar .ova with `tar xvf`
* there will be a .vmdk file, convert it to .qcow2
* `qemu-img convert -f vmdk -O qcow2 image-disk1.vmdk image.qcow2`

# vlan

* https://user.pnetlab.com/store/labs/detail?id=16405723981793

![](img/2024-10-09-15-21-04.png)

* the location of downloaded lab: `/opt/unetlab/labs/Your\ labs\ from\ PNETLab\ Store`
* download images

```
ishare2 labs          # Will show all labs available
ishare2 labs <number> # Will download images for the lab with the specified number
```

* the location of downloaded images: `/opt/unetlab/addons/`

```
root@pnetlab:/opt/unetlab/addons# ls -l
total 12
drwxr-xr-x 2 root     root     4096 Apr  4  2020 dynamips
drwxr-xr-x 4 www-data www-data 4096 Jan 13  2021 iol
drwxr-xr-x 4 root     root     4096 Oct  3 17:33 qemu
```

## .bin files

* for simulated cisco switches
* `i86bi-Linux-L2-Adventerprisek9-ms.SSA.high_iron_20190423.bin`
* symolic link it to `/opt/unetlab/tmp/<lab #>/<instance #>/

```
root@pnetlab:/opt/unetlab/tmp/1/10# ls -l
total 16
lrwxrwxrwx 1 root unl   88 Oct  3 18:45 i86bi_Linux-L2-Adventerprisek9-ms.SSA.high_iron_20190423.bin -> /opt/unetlab/addons/iol/bin/i86bi_Linux-L2-Adventerprisek9-ms.SSA.high_iron_20190423.bin
lrwxrwxrwx 1 root unl   80 Oct  3 18:47 i86bi_Linux-L3-AdvEnterpriseK9-M2_157_3_May_2018.bin -> /opt/unetlab/addons/iol/bin/i86bi_Linux-L3-AdvEnterpriseK9-M2_157_3_May_2018.bin
lrwxrwxrwx 1 root unl   33 Oct  2 19:03 iourc -> /opt/unetlab/addons/iol/bin/iourc
lrwxrwxrwx 1 root unl   40 Oct  2 19:03 keepalive.pl -> /opt/unetlab/addons/iol/bin/keepalive.pl
-rwxrwxrwx 1 root unl 1190 Oct  2 19:03 startup-config
-rwxrwxrwx 1 root unl  225 Oct  3 17:49 wrapper.txt
```

# Config l2 switch

## config trunk

```
enable
config t
switchport trunk encapulation dot1q
switchport mode trunk
switchport nonegotiate
```

## config access

```
enable
config t
interface e0/2
switchport mode access
switchport access vlan 200
```

* if vlan 200 not configured prior, it will be automatically created

## config vlan

```
enable
config t
vlan 200
name test
exit
exit
show vlan id 200


```

* all trunk port will join this vlan automatically

```
SW1#show vlan id 200

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
200  test                             active    Et0/0, Et0/1, Et0/2
```

## config STP

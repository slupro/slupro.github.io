---
layout: post
title:  "Kickstart Red Hat Enterprise Linux 8 by PXE server"
subtitle:   "使用 PXE server 自动部署 RHEL8"
date:   2021-11-20 8:00:00 +10:00
author: "Steven Lu @slupro"
categories: [学习笔记]  # up to 2.
tags: [Linux, kickstart]  # TAG names should always be lowercase, 0 to infinity.
---

## Kickstart RHEL 8

### Automated installation steps

1. Create a Kickstart file.
2. Prepare the Linux installation ISO or installation files.
3. Setup PXE server.
4. Setup Web server to provide Kickstart file and installation files. It could be replaced by NFS or FTP.
5. Start the client machine to start the installation. (PXE needs to be supported and enabled in BIOS)

### Create a Kickstart file

* Kickstart file genertor: https://access.redhat.com/labsinfo/kickstartconfig.
* (Recommended by RedHat) Perform a manual installation, then get the Kickstart file at /root/anaconda-ks.cfg.
* Use the GUI application system-config-kickstart.
* Change the Kickstart file as you need.

> In Kickstart file, we can config everything that we can config in manual installation.
> Kickstart file needs to specify the installation resource via HTTP, FTP or NFS.
> We can set post-installation scripts in Kickstart file.

### Prepare PXE server

PXE server provides boot loader for client machine.

#### DHCP server

* ```next-server``` to specify the IP of tftp server which holds the initial boot file.
* ```filename``` to specify the boot loader name on tftp server. (eg: /pxelinux.0)

#### tftp server

* Put the boot image and related files on ftp server: pxelinux.0, ldlinux.c32, vmlinuz, initrd.img. These files could be found in RHEL ISO image (/isolinux/).
* Prepare the boot loader config file(boot menu). 
  * It's under ```/pxelinux.cfg/``` folder. The client machine will follow this order to find config file: mac(eg, ```01-08-00-27-83-1e-2a```), IP address(hex value, eg, ```0a0a0ac0``` means 10.10.10.192), ```default```.
  * We can refer to ```/isolinux/isolinux.cfg``` which is in RHEL ISO image.
* Specify Kickstart file in boot loader config file by using ```ks=```. eg, ```append initrd=initrd.img ks=http://192.168.8.8/ks/rhel.cfg```

### Web server

During installation, the client machine needs to download installation files from http server. eg, http://192.168.8.8/rhel

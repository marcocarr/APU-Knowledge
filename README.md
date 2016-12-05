APU Knowledge
=============
Basic installation and settings guide for APU systems.

Get started with Creating a bootable USB-Device!
================================================

## USB-Device Settings&Preparation

Prepare the Stick with UNetBootIn. 
I recommend using a Long Time Support ("LTS") version of an Ubuntu server image.
After prepairing the USB-Device, you need to change the following files.

1. $mnt/isolinux/isolinux.cfg
2. $mnt/isolinux/txt.cfg
3. $mnt/syslinux.cfg

## isolinux.cfg


After the changes your document should look like this : 

```bash
# D-I config version 2.0CONSOLE 0SERIAL 0 115200 0include menu.cfgdefault vesamenu.c32prompt 0timeout 0
```
You should set the setting of the Serial Port to 115200 Baud. because the APU works with it.

## txt.cfg

After the changes your document should look like this : 

```bash
default installlabel install menu label ^Install Ubuntu Server kernel /install/vmlinuz append
file=/cdrom/preseed/ubuntu-server.seed \ vga=788 initrd=/install/initrd.gz -- \ console=ttyS0,115200n8
quiet –
```
Die Zeile append ..., die hier nur der Lesbarkeit halber umgebrochen ist, enthält die Kernel-
Kommandozeile mit der Einstellung console=ttyS0,115200n8, damit der Kernel Ausgaben an die Konsole
zur seriellen Schnittstelle umleitet.

The row with the append statement was wordwraped to be more readable. 
With this setting you change the Kernel settings, so it does show the Kernel output on the serial console.


## syslinux.cfg

After the changes your document should look like this : 

```bash

# D-I config version 2.0CONSOLE 0SERIAL 0 115200 0default menu.c32prompt 0menu title
UNetbootintimeout 100label unetbootindefaultkernel /install/netboot/ubuntu-
installer/amd64/linuxappend initrd=/install/netboot/ubuntu-installer/amd64/initrd.gz \ tasks=standard
pkgsel/language-pack-patterns= \ pkgsel/install-language-support=false vga=788 -- \
console=ttyS0,115200n8 --
```

This document was also wordwraped.
Here you just set the same setting as in the two documents before.


PuTTY-Settings for the serial console.
======================================

To read out the Output of what the console give you, you should connect with the preset settings
from the changed boot files.

## Settings

Session – Logging :
Connection Type : Serial

SSH – Serial :
Serial Line : Your COM-Port of ttyS-Port
Speed : 115200
Data bits : 8
Stop bits : 1
Flow Control : XON / XOFF

Installation process
====================

## How to correctly plug in the APU

Verkabelung am PC über ein serielles Kabel an den APU an und starte putty. Der USB-Stick kommt an
einen USB-Port und das Ethernet-Kabel an den Anschluss neben der seriellen Konsole.
 
## Correct Order:
 
1. Plug in a serial cable in the PC and the APU, plug in the ethernet cable beside the serial one.
2. Plug in the USB-Device in the upper slot to automaticly boot from it.

After having established the connection with the serial console you can plug in the APU.
If it does not boot you can change the boot-order with F10.

## Important Installation setting

You are going to be guided through the whole installation by the installation itself, here are just some 
important settings.

### Disk-Settings

You should never install the BootLoader to the Master Boot Load , if the question comes up to set up the
BootLoader always install in on the 16GB SSD Disk (/dev/sdb/).
To be able to do so you need to use the Guided Disk usage by using the Entire disk and selecting there the
SBD one.

### Packages and Settings

Excecute everything as ```bash SUDO ```.

```bash
apt-get install dnsmasq
apt-get install isc-dhcp-server
apt-get install openvpn
apt-get install ntp
apt-get install shorewall
apt-get install easy-rsa

```
### Network settings
You also need to set up some minor Network changes to correctly set-up all the packages.

```bash
#eth0 = p4p1

touch /etc/gateway/current/inferfaces.conf
echo 'source /etc/gateway/current/interfaces.conf' >> /etc/network/interfaces
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
echo 'net.ipv6.conf.all.disable_ipv6=1' >> /etc/sysctl.conf
```
You also need to check what kind of interface you are using (eth0 or p4p1).

### SSH
If you want to use "AuthorizedKeysFile" you need to execute the following statements.
```bash
# edit /etc/ssh/sshd_config 
# activate AuthorizedKeysFile     %h/.ssh/authorized_keys
# restart SSH
```

### dnsmasq
```bash
/var/run/dnsmasq/resolv.conf 
```
This containts a list of name servers who get used by dnsmasq.

If a new DNS gets created over the interface, it will be automaticly added to the list.

```bash
touch /etc/gateway/current/hosts.conf
echo 'addn-hosts=/etc/gateway/current/hosts.conf' >> /etc/dnsmasq.conf
```
### isc-dhcp-server

```bash
DHCP-Server-Lease-File
Fix -> http://wiki.freifunk-flensburg.de/wiki/Workaround:DHCP-Server-Lease-File
service isc-dhcp-server stop
chown -R dhcpd:dhcpd /var/lib/dhcp
nano /etc/init/isc-dhcp-server.conf
```

Just set the following statements as comments or just delete them.

```bash
The leases files need to be root:root even when droppig privileges
[ -e /var/lib/dhcp/dhcpd.leases ] || touch /var/lib/dhcp/dhcpd.leases
chown root:root /var/lib/dhcp /var/lib/dhcp/dhcpd.leases
if [ -e /var/lib/dhcp/dhcpd.leases~ ]; then
chown root:root /var/lib/dhcp/dhcpd.leases~
fi
echo 'capability dac_override,' >> /etc/apparmor.d/local/usr.sbin.dhcpdservice apparmor reload
service isc-dhcp-server start

Open
```

### openvpn

Use the recommended settings here. 

```bash
mkdir /etc/gateway/openvpn/easy-rsa
cp -r /usr/share/easy-rsa/* /etc/gateway/openvpn/easy-rsa
chown -R root:maintenance /etc/gateway/openvpn/easy-rsa
chmod -R 770 /etc/gateway/openvpn/easy-rsa
#cd /etc/gateway/openvpn/easy-rsa/
#./01_server.sh
#challenge Password: #Yr3nJNb[7
```
### ntp

You don't need to change anything here this package is just there to keep sure the time is set correctly.

### shorewall

You should always start shorewell on startup to keep sure all of your files are safe.

```bash
# Start Shorewall on Startup
echo -e 'description "shorewall firewall startup"\n\nstart on runlevel [2345]\nstop on runlevel
[!2345]\n\nexec /sbin/shorewall restart' > /etc/init/shorewall$
```

### easy-rsa

The easy-rsa package is just to lock the front-door, this will prevent people to find your VPN-Server.
Also it will implement a pass-key you should use.

Conclusion 
==========

You will find all of the Script-files ready to copy it into the right directory.

Everything was tested on Ubuntu 16.04.01 LTS and Windows Machines.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request




 
 
# Temporarily set SELinux to permissive mode
sudo setenforce 0

# Edit the SELinux configuration file
sudo vi /etc/selinux/config

# Change SELINUX=enforcing to SELINUX=disabled

# Reboot the system for changes to take effect
sudo reboot



[root@pxeserver-oel7 exadata]# ls -ltr
total 687076
-rwxrwxrwx 1 root root  10977840 Oct 17 17:18 vmlinux-iso-24.1.5.0.0-241016-cell
-rw-rw-rw- 1 root root 340013799 Oct 17 17:18 initrd-iso-24.1.5.0.0-241016-cell.img
-rwxrwxrwx 1 root root  10977840 Oct 17 17:18 vmlinux-iso-24.1.5.0.0-241016-compute
-rw-rw-rw- 1 root root 341497186 Oct 17 17:18 initrd-iso-24.1.5.0.0-241016-compute.img
drwxr-xr-x 2 root root         6 Oct 30 10:20 pxelinux.cfg
-rw-r--r-- 1 root root     26746 Oct 30 10:21 pxelinux.0
-rw-r--r-- 1 root root     55140 Oct 30 10:21 menu.c32


# vi /tftpboot/exadata/pxelinux.cfg/default
timeout 600
default menu.c32
prompt 0
menu title Please Select Image

label ExaDB
menu label Exadata Compute Node
kernel vmlinux-iso-24.1.3.0.0-240809.1-compute
append initrd-iso-24.1.3.0.0-240809.1-compute.img pxe stit updfrm dhcp reboot-on-success dualboot=no notests=diskgroup sk=192.168.101.1:/tftpboot/exadata preconf=192.168.101.1:/tftpboot/exadata/preconf.csv

label ExaCell
menu label Exadata Cell Node
kernel vmlinux-iso-24.1.3.0.0-240809.1-cell
append initrd-iso-24.1.3.0.0-240809.1-cell.img pxe stit updfrm dhcp reboot-on-success dualboot=no notests=diskgroup sk=192.168.101.1:/tftpboot/exadata preconf=192.168.101.1:/tftpboot/exadata/preconf.csv







# DHCP Server Configuration file
# see /usr/share/doc/dhcp-server/dhcpd.conf.example
# see dhcpd.conf(5) man page

deny unknown-clients;
not authoritative;
allow bootp;
allow booting;
option ip-forwarding false;     # No IP forwarding
option mask-supplier false;     # Don't respond to ICMP Mask req

subnet 192.168.101.0 netmask 255.255.255.0 {
    option routers 192.168.101.1;
}

# DB Node1
group {
    next-server 192.168.101.1;
    filename "exadata/pxelinux.0";
    option root-path "192.168.101.1:/tftpboot/exadata";
    
    host poxmp040131 {
        hardware ethernet a8:69:8c:3e:0a:00;
        fixed-address 192.168.101.131;
    }
}

# DB Node 2
group {
    next-server 192.168.101.1;
    filename "exadata/pxelinux.0";
    option root-path "192.168.101.1:/tftpboot/exadata";
    
    host poxmp040132 {
        hardware ethernet a8:69:8c:3e:07:b0;
        fixed-address 192.168.101.132;
    }
}

# Cell Node 1
group {
    next-server 192.168.101.1;
    filename "exadata/pxelinux.0";
    option root-path "192.168.101.1:/tftpboot/exadata";
    
    host poxmp040133 {
        hardware ethernet a8:69:8c:3e:1d:d0;
        fixed-address 192.168.101.133;
    }
}

# Cell Node 2
group {
    next-server 192.168.101.1;
    filename "exadata/pxelinux.0";
    option root-path "192.168.101.1:/tftpboot/exadata";
    
    host poxmp040134 {
        hardware ethernet a8:69:8c:3e:17:d8;
        fixed-address 192.168.101.134;
    }
}

# Cell Node 3
group {
    next-server 192.168.101.1;
    filename "exadata/pxelinux.0";
    option root-path "192.168.101.1:/tftpboot/exadata";
    
    host poxmp040135 {
        hardware ethernet a8:69:8c:3e:21:80;
        fixed-address 192.168.101.135;
    }
}




vi /tftpboot/exadata/pxelinux.cfg/default


timeout 600
default menu.c32
prompt 0
menu title Please Select Image

label ExaDB
menu label Exadata Compute Node
kernel vmlinux-iso-24.1.5.0.0-241016-compute
append initrd-iso-24.1.5.0.0-241016-compute.img pxe stit updfrm dhcp reboot-on-success dualboot=no notests=diskgroup sk=192.168.101.1:/tftpboot/exadata preconf=192.168.101.1:/tftpboot/exadata/preconf.csv

label ExaCell
menu label Exadata Cell Node
kernel vmlinux-iso-24.1.5.0.0-241016-cell
append initrd-iso-24.1.5.0.0-241016-cell.img pxe stit updfrm dhcp reboot-on-success dualboot=no notests=diskgroup sk=192.168.101.1:/tftpboot/exadata preconf=192.168.101.1:/tftpboot/exadata/preconf.csv



vim /tftpboot/exadata/boot.msg

Press "Enter" to boot using the default Boot Image (Exadata Compute Node), it will start in
60 seconds if not any keys pressed.

Type "ExaDB" For Exadata Compute Node ReImage
Type "ExaCell" For Exadata Cell Node ReImage


vim /etc/xinetd.d/tftp


# default: off
# description: The tftp server serves files using the trivial file transfer \
# protocol. The tftp protocol is often used to boot diskless \
# workstations, download configuration files to network-aware printers, \
# and to start the installation process for some operating systems.
service tftp
{
socket_type = dgram
protocol = udp
wait = yes
user = root
server = /usr/sbin/in.tftpd
server_args = -s /tftpboot
disable = no
per_source = 11
cps = 100 2
flags = IPv4
}

vim /etc/exports

/tftpboot/exadata *(ro)





# Global parameters
default-lease-time 600;
max-lease-time 7200;
authoritative;

# Subnet declaration for eth1 (192.168.101.0 network)
subnet 192.168.101.0 netmask 255.255.255.0 {
    range 192.168.101.100 192.168.101.200;    # IP range to assign
    option routers 192.168.101.1;             # Your eth1 IP as gateway
    option domain-name-servers 8.8.8.8;       # DNS servers
    option domain-name "localdomain";
    
    # If you're setting up PXE boot, add these lines:
    #filename "pxelinux.0";
    #next-server 192.168.101.1;
}


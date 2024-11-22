

#!/bin/bash

# Step 1: Install required packages
dnf install -y dhcp-server tftp-server syslinux-tftpboot httpd

# Step 2: Configure DHCP Server
cat > /etc/dhcp/dhcpd.conf << 'EOF'
allow booting;
allow bootp;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.100 192.168.100.200;
    option routers 192.168.100.1;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 8.8.8.8;
    filename "pxelinux.0";
    next-server 192.168.100.149;  # Replace with your PXE server IP
}
EOF

# Step 3: Configure TFTP Server
mkdir -p /var/lib/tftpboot/{pxelinux.cfg,images}
cp -v /tftpboot/{pxelinux.0,ldlinux.c32,libcom32.c32,libutil.c32,vesamenu.c32,menu.c32} /var/lib/tftpboot/

# Step 4: Create PXE boot menu
mkdir -p /var/lib/tftpboot/pxelinux.cfg
cat > /var/lib/tftpboot/pxelinux.cfg/default << 'EOF'
DEFAULT vesamenu.c32
TIMEOUT 600
ONTIMEOUT local
PROMPT 0
MENU TITLE PXE Boot Menu
MENU BACKGROUND splash.png

LABEL local
    MENU LABEL Boot from ^Local Drive
    LOCALBOOT 0x80

LABEL almalinux94
    MENU LABEL ^AlmaLinux 9.4 Installation
    KERNEL images/almalinux94/vmlinuz
    APPEND initrd=images/almalinux94/initrd.img inst.repo=http://192.168.100.149/almalinux94 inst.ks=http://192.168.100.149/ks/almalinux94.cfg

LABEL oraclelinux810
    MENU LABEL ^Oracle Linux 8.10 Installation
    KERNEL images/oraclelinux810/vmlinuz
    APPEND initrd=images/oraclelinux810/initrd.img inst.repo=http://192.168.100.149/oraclelinux810 inst.ks=http://192.168.100.149/ks/oraclelinux810.cfg
EOF

# Step 5: Create directories for Linux distributions
mkdir -p /var/www/html/{almalinux94,oraclelinux810}
mkdir -p /var/lib/tftpboot/images/{almalinux94,oraclelinux810}
mkdir -p /var/www/html/ks

# Step 6: Download and mount AlmaLinux 9.4 ISO
wget https://repo.almalinux.org/almalinux/9.4/isos/x86_64/AlmaLinux-9.4-x86_64-dvd.iso
mount -o loop AlmaLinux-9.4-x86_64-dvd.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/almalinux94/
cp -rf /mnt/* /var/www/html/almalinux94/
umount /mnt

# Step 7: Download and mount Oracle Linux 8.10 ISO
wget https://yum.oracle.com/ISOS/OracleLinux/OL8/u10/x86_64/OracleLinux-R8-U10-x86_64-dvd.iso
mount -o loop OracleLinux-R8-U10-x86_64-dvd.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/oraclelinux810/
cp -rf /mnt/* /var/www/html/oraclelinux810/
umount /mnt

# Step 8: Create sample Kickstart files
cat > /var/www/html/ks/almalinux94.cfg << 'EOF'
#version=RHEL9
autopart --type=lvm
clearpart --all --initlabel
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network --bootproto=dhcp
rootpw --iscrypted $6$xyz$xyz...  # Replace with your encrypted password
timezone Asia/Manila --utc
bootloader --append="crashkernel=auto" --location=mbr
reboot
url --url="http://192.168.100.149/almalinux94"
%packages
@^minimal-environment
%end
EOF

cat > /var/www/html/ks/oraclelinux810.cfg << 'EOF'
#version=RHEL8
autopart --type=lvm
clearpart --all --initlabel
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network --bootproto=dhcp
rootpw --iscrypted $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.  # Replace with your encrypted password
timezone Asia/Manila --utc
bootloader --append="crashkernel=auto" --location=mbr
reboot
url --url="http://192.168.100.149/oraclelinux810"
%packages
@^minimal-environment
%end
EOF

# Step 9: Set proper permissions
chmod 644 /var/www/html/ks/*.cfg
restorecon -RFv /var/lib/tftpboot
restorecon -RFv /var/www/html

# Step 10: Configure firewall
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --permanent --add-service=tftp
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

# Step 11: Enable and start services
systemctl enable --now dhcpd
systemctl enable --now tftp
systemctl enable --now httpd

# Step 12: Set SELinux boolean for TFTP
setsebool -P tftp_home_dir on

# Step 13: Verify services are running
systemctl status dhcpd
systemctl status tftp
systemctl status httpd

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

LABEL rocky89
    MENU LABEL ^Rocky Linux 8.9 Installation
    KERNEL images/rocky89/vmlinuz
    APPEND initrd=images/rocky89/initrd.img inst.repo=http://192.168.100.149/rocky89 inst.ks=http://192.168.100.149/ks/rocky89.cfg

LABEL oraclelinux810
    MENU LABEL Oracle ^Linux 8.10 Installation
    KERNEL images/oraclelinux810/vmlinuz
    APPEND initrd=images/oraclelinux810/initrd.img inst.repo=http://192.168.100.149/oraclelinux810 inst.ks=http://192.168.100.149/ks/oraclelinux810.cfg

LABEL oraclelinux79
    MENU LABEL Oracle Linux ^7.9 Installation
    KERNEL images/oraclelinux79/vmlinuz
    APPEND initrd=images/oraclelinux79/initrd.img inst.repo=http://192.168.100.149/oraclelinux79 inst.ks=http://192.168.100.149/ks/oraclelinux79.cfg

LABEL ubuntu2004
    MENU LABEL Ubuntu 20.04 ^LTS Installation
    KERNEL images/ubuntu2004/vmlinuz
    APPEND initrd=images/ubuntu2004/initrd.gz root=/dev/ram0 ramdisk_size=1500000 ip=dhcp url=http://192.168.100.149/ubuntu2004/preseed.cfg

LABEL ubuntu2204
    MENU LABEL Ubuntu 22.04 L^TS Installation
    KERNEL images/ubuntu2204/vmlinuz
    APPEND initrd=images/ubuntu2204/initrd.gz root=/dev/ram0 ramdisk_size=1500000 ip=dhcp url=http://192.168.100.149/ubuntu2204/preseed.cfg

LABEL ubuntu2404
    MENU LABEL Ubuntu 24.04 LT^S Installation
    KERNEL images/ubuntu2404/vmlinuz
    APPEND initrd=images/ubuntu2404/initrd.gz root=/dev/ram0 ramdisk_size=1500000 ip=dhcp url=http://192.168.100.149/ubuntu2404/preseed.cfg
EOF

# Step 5: Create directories for Linux distributions
mkdir -p /var/www/html/{almalinux94,rocky89,oraclelinux810,oraclelinux79,ubuntu2004,ubuntu2204,ubuntu2404}
mkdir -p /var/lib/tftpboot/images/{almalinux94,rocky89,oraclelinux810,oraclelinux79,ubuntu2004,ubuntu2204,ubuntu2404}
mkdir -p /var/www/html/ks

# Step 6: Download and mount AlmaLinux 9.4
wget https://repo.almalinux.org/almalinux/9.4/isos/x86_64/AlmaLinux-9.4-x86_64-dvd.iso
mount -o loop AlmaLinux-9.4-x86_64-dvd.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/almalinux94/
cp -rf /mnt/* /var/www/html/almalinux94/
umount /mnt

# Step 7: Download and mount Rocky Linux 8.9
wget https://download.rockylinux.org/pub/rocky/8/isos/x86_64/Rocky-8.9-x86_64-dvd1.iso
mount -o loop Rocky-8.9-x86_64-dvd1.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/rocky89/
cp -rf /mnt/* /var/www/html/rocky89/
umount /mnt

# Step 8: Download and mount Oracle Linux 8.10
wget https://yum.oracle.com/ISOS/OracleLinux/OL8/u10/x86_64/OracleLinux-R8-U10-x86_64-dvd.iso
mount -o loop OracleLinux-R8-U10-x86_64-dvd.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/oraclelinux810/
cp -rf /mnt/* /var/www/html/oraclelinux810/
umount /mnt

# Step 9: Download and mount Oracle Linux 7.9
wget https://yum.oracle.com/ISOS/OracleLinux/OL7/u9/x86_64/OracleLinux-R7-U9-Server-x86_64-dvd.iso
mount -o loop OracleLinux-R7-U9-Server-x86_64-dvd.iso /mnt
cp /mnt/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/images/oraclelinux79/
cp -rf /mnt/* /var/www/html/oraclelinux79/
umount /mnt

# Step 10: Download and extract Ubuntu ISOs
# Ubuntu 20.04 LTS
wget https://releases.ubuntu.com/20.04/ubuntu-20.04.6-live-server-amd64.iso
mount -o loop ubuntu-20.04.6-live-server-amd64.iso /mnt
cp /mnt/casper/{vmlinuz,initrd} /var/lib/tftpboot/images/ubuntu2004/
cp -rf /mnt/* /var/www/html/ubuntu2004/
umount /mnt

# Ubuntu 22.04 LTS
wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
mount -o loop ubuntu-22.04.3-live-server-amd64.iso /mnt
cp /mnt/casper/{vmlinuz,initrd} /var/lib/tftpboot/images/ubuntu2204/
cp -rf /mnt/* /var/www/html/ubuntu2204/
umount /mnt

# Ubuntu 24.04 LTS
wget https://releases.ubuntu.com/24.04/ubuntu-24.04-live-server-amd64.iso
mount -o loop ubuntu-24.04-live-server-amd64.iso /mnt
cp /mnt/casper/{vmlinuz,initrd} /var/lib/tftpboot/images/ubuntu2404/
cp -rf /mnt/* /var/www/html/ubuntu2404/
umount /mnt

# Step 11: Create Kickstart files for RHEL-based systems
cat > /var/www/html/ks/rocky89.cfg << 'EOF'
#version=RHEL8
ignoredisk --only-use=sda
autopart --type=lvm
clearpart --all --initlabel --drives=sda
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network --bootproto=dhcp
rootpw --iscrypted $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.  # Replace with your encrypted password
timezone Asia/Manila --utc
bootloader --append="crashkernel=auto" --location=mbr
reboot
url --url="http://192.168.100.149/rocky89"
%packages
@^minimal-environment
%end
EOF

# Create similar Kickstart files for other RHEL-based systems
cp /var/www/html/ks/rocky89.cfg /var/www/html/ks/oraclelinux79.cfg
sed -i 's/RHEL8/RHEL7/' /var/www/html/ks/oraclelinux79.cfg
sed -i 's/rocky89/oraclelinux79/' /var/www/html/ks/oraclelinux79.cfg

# Step 12: Create Preseed files for Ubuntu systems
cat > /var/www/html/ubuntu2004/preseed.cfg << 'EOF'
# Locale and keyboard
d-i debian-installer/locale string en_US.UTF-8
d-i keyboard-configuration/xkb-keymap select us

# Network
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string ubuntu
d-i netcfg/get_domain string local

# Mirror
d-i mirror/country string US
d-i mirror/http/hostname string archive.ubuntu.com
d-i mirror/http/directory string /ubuntu

# User account
d-i passwd/user-fullname string Ubuntu User
d-i passwd/username string ubuntu
d-i passwd/user-password-crypted password $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.  # Replace with your encrypted password

# Partitioning
d-i partman-auto/method string lvm
d-i partman-auto-lvm/guided_size string max
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Package selection
tasksel tasksel/first multiselect ubuntu-server
d-i pkgsel/include string openssh-server

# Boot loader
d-i grub-installer/only_debian boolean true
d-i grub-installer/bootdev string default

# Finish installation
d-i finish-install/reboot_in_progress note
EOF

# Copy preseed file for other Ubuntu versions
cp /var/www/html/ubuntu2004/preseed.cfg /var/www/html/ubuntu2204/
cp /var/www/html/ubuntu2004/preseed.cfg /var/www/html/ubuntu2404/

# Step 13: Set proper permissions
chmod 644 /var/www/html/ks/*.cfg
chmod 644 /var/www/html/ubuntu*/preseed.cfg
restorecon -RFv /var/lib/tftpboot
restorecon -RFv /var/www/html

# Step 14: Configure firewall
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --permanent --add-service=tftp
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

# Step 15: Enable and start services
systemctl enable --now dhcpd
systemctl enable --now tftp
systemctl enable --now httpd

# Step 16: Set SELinux boolean for TFTP
setsebool -P tftp_home_dir on

# Step 17: Verify services are running
systemctl status dhcpd
systemctl status tftp
systemctl status httpd
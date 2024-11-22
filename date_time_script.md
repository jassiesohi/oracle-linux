

#!/bin/bash

# Step 1: Check current timezone and time
echo "Current timezone and time settings:"
timedatectl status
echo "Current date command output:"
date

# Step 2: List available Manila timezones
echo -e "\nAvailable Manila timezones:"
timedatectl list-timezones | grep Manila

# Step 3: Set timezone to Asia/Manila
timedatectl set-timezone Asia/Manila

# Step 4: Enable and start chronyd for time synchronization
dnf install -y chrony
systemctl enable --now chronyd

# Step 5: Configure chrony to use Philippine NTP servers
cat > /etc/chrony.conf << 'EOF'
# Philippine NTP servers
server 0.ph.pool.ntp.org iburst
server 1.ph.pool.ntp.org iburst
server 2.ph.pool.ntp.org iburst
server 3.ph.pool.ntp.org iburst

# Allow NTP client access from local network (adjust as needed)
#allow 192.168.0.0/16

# Record the rate at which the system clock gains/losses time
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC)
rtcsync

# Specify directory for log files
logdir /var/log/chrony
EOF

# Step 6: Restart chronyd to apply changes
systemctl restart chronyd

# Step 7: Force immediate time synchronization
chronyc makestep

# Step 8: Set hardware clock to local time
timedatectl set-local-rtc 0

# Step 9: Update hardware clock
hwclock --systohc

# Step 10: Display final configuration
echo -e "\nFinal timezone configuration:"
timedatectl status

echo -e "\nChrony sources status:"
chronyc sources

echo -e "\nChrony tracking status:"
chronyc tracking

echo -e "\nCurrent system date and time:"
date
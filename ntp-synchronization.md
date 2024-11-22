#!/bin/bash

# Step 1: Backup original chrony configuration
cp /etc/chrony.conf /etc/chrony.conf.backup

# Step 2: Create new chrony configuration
cat > /etc/chrony.conf << 'EOF'
# Use all available NTP pools for better reliability
pool pool.ntp.org iburst
pool asia.pool.ntp.org iburst
pool 0.asia.pool.ntp.org iburst
pool 1.asia.pool.ntp.org iburst
pool 2.asia.pool.ntp.org iburst
pool 3.asia.pool.ntp.org iburst

# Philippine NTP servers
server 0.ph.pool.ntp.org iburst
server 1.ph.pool.ntp.org iburst
server 2.ph.pool.ntp.org iburst
server 3.ph.pool.ntp.org iburst

# Google's NTP servers as fallback
server time1.google.com iburst
server time2.google.com iburst
server time3.google.com iburst
server time4.google.com iburst

# Allow NTP client access from local network (adjust as needed)
#allow 192.168.0.0/16

# Record the rate at which the system clock gains/losses time
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
makestep 1 3

# Enable kernel synchronization of the real-time clock (RTC)
rtcsync

# Enable logging
logdir /var/log/chrony
log measurements statistics tracking

# Increase the minimum number of selectable sources
minsources 2

# Allow for larger time corrections
maxdistance 16.0

# Speed up initial synchronization
initstepslew 10 pool.ntp.org
EOF

# Step 3: Check if firewall is blocking NTP
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload

# Step 4: Check SELinux context for chrony
restorecon -rv /var/lib/chrony
restorecon -rv /var/log/chrony

# Step 5: Verify network connectivity to NTP servers
echo "Testing network connectivity to NTP servers..."
for server in pool.ntp.org time.google.com 0.ph.pool.ntp.org; do
    echo "Testing $server:"
    ping -c 2 $server
done

# Step 6: Stop chronyd, clear its state, and restart
systemctl stop chronyd
rm -f /var/lib/chrony/drift*
rm -f /var/lib/chrony/chronyd.state

# Step 7: Start and verify chronyd service
systemctl start chronyd
systemctl status chronyd

# Step 8: Wait for initial synchronization
echo "Waiting for initial synchronization (this may take a few minutes)..."
sleep 30

# Step 9: Force time synchronization
chronyc makestep

# Step 10: Check synchronization status
echo -e "\nChecking chrony sources:"
chronyc sources -v

echo -e "\nChecking chrony tracking:"
chronyc tracking

echo -e "\nChecking chrony sourcestats:"
chronyc sourcestats

# Step 11: View chronyd logs
echo -e "\nRecent chronyd logs:"
journalctl -u chronyd --since "5 minutes ago"

# Step 12: Display current system time
echo -e "\nCurrent system time:"
date
timedatectl status
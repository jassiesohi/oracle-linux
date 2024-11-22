

[root@pxe-alma9-node0 scripts]# ./password-encryption-method.sh
Choose password encryption method:
1. Python (recommended for RHEL systems)
2. OpenSSL
3. mkpasswd (requires whois package)
4. grub-crypt (RHEL systems only)
Enter choice (1-4): 1
Enter password when prompted:
Password:

Encrypted password: $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.

Use this line in kickstart files:
rootpw --iscrypted $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.

Use this line in preseed files:
d-i passwd/user-password-crypted password $6$TRIMAtbiJfQT1fKv$niT4JPcetaGsIcRfFoVcRV6XljOLe.tTlBenxecDLNJX5lduZsR3tYM4MIdjapFkp3W7R/wUldJfjBM/.c5XE.


# I'll show you how to generate an encrypted password for the kickstart and preseed files.



bash
#!/bin/bash

# Method 1: Using python for RHEL-based systems (AlmaLinux, Rocky, Oracle)
generate_rhel_password() {
    python3 -c 'import crypt,getpass; print(crypt.crypt(getpass.getpass(), crypt.mksalt(crypt.METHOD_SHA512)))'
}

# Method 2: Using openssl
generate_openssl_password() {
    openssl passwd -6 -stdin
}

# Method 3: Using mkpasswd (requires whois package)
generate_mkpasswd_password() {
    mkpasswd --method=SHA-512 --rounds=4096
}

# Method 4: Using grub-crypt (available on RHEL-based systems)
generate_grub_password() {
    grub2-mkpasswd-pbkdf2 | grep "grub.pbkdf2" | cut -d " " -f 7
}

echo "Choose password encryption method:"
echo "1. Python (recommended for RHEL systems)"
echo "2. OpenSSL"
echo "3. mkpasswd (requires whois package)"
echo "4. grub-crypt (RHEL systems only)"
read -p "Enter choice (1-4): " choice

case $choice in
    1)
        echo "Enter password when prompted:"
        encrypted=$(generate_rhel_password)
        ;;
    2)
        echo "Enter password:"
        encrypted=$(generate_openssl_password)
        ;;
    3)
        if ! command -v mkpasswd &> /dev/null; then
            echo "Installing whois package..."
            dnf install -y whois || apt install -y whois
        fi
        echo "Enter password:"
        encrypted=$(generate_mkpasswd_password)
        ;;
    4)
        if ! command -v grub2-mkpasswd-pbkdf2 &> /dev/null; then
            echo "Installing grub2-tools package..."
            dnf install -y grub2-tools
        fi
        echo "Enter password when prompted:"
        encrypted=$(generate_grub_password)
        ;;
    *)
        echo "Invalid choice"
        exit 1
        ;;
esac

echo -e "\nEncrypted password: $encrypted"
echo -e "\nUse this line in kickstart files:"
echo "rootpw --iscrypted $encrypted"
echo -e "\nUse this line in preseed files:"
echo "d-i passwd/user-password-crypted password $encrypted"


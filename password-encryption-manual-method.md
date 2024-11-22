
# Manual Method
# Quick guide for each method:

# 1. Python Method (Recommended for RHEL-based systems):

# One-liner version
python3 -c 'import crypt,getpass; print(crypt.crypt(getpass.getpass(), crypt.mksalt(crypt.METHOD_SHA512)))'


# 2. OpenSSL Method:
bash
# One-liner version
echo "YourPassword" | openssl passwd -6 -stdin


# 3. mkpasswd Method:
bash
# Install whois package first
dnf install -y whois   # For RHEL-based systems
apt install -y whois   # For Ubuntu

# Generate password
echo "YourPassword" | mkpasswd --method=SHA-512 --rounds=4096


# 4. grub-crypt Method:
bash
# Install grub2-tools if needed
dnf install -y grub2-tools

# Generate password
grub2-mkpasswd-pbkdf2


# Usage tips:

# 1. The generated password will look something like:
   
   $6$xyz123abc$hgfedcba...long_string_of_characters
   

# 2. Make sure to keep the entire string, including the `$6$` prefix

# 3. For each distribution type:
   - RHEL-based (AlmaLinux, Rocky, Oracle):
     
     rootpw --iscrypted $6$xyz123abc$hgfedcba...
     
   - Ubuntu:
     
     d-i passwd/user-password-crypted password $6$xyz123abc$hgfedcba...
     

# 4. For testing, you can use a simple password, but for production:
   - Use a strong password
   - Generate unique passwords for each system
   - Store the passwords securely
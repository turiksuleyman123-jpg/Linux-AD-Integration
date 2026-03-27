# Linux-AD-Integration
Integrating Linux with AD (Active Directory) on Windows Server 2022.

# Objective
Connect Linux (RHEL/CentOS/Ubuntu) to Active Directory to enable centralized user authentication (Single Sign-On).

# OS and Software
Linux: RHEL 10

AD: Windows Server 2022

Linux Packages: sssd, realmd, krb5-workstation, samba, oddjob, oddjob-mkhomedir

# Note: All operations are performed with ROOT privileges.

# 1. Install Required Packages
Install SSSD and Samba packages. SSSD handles AD authentication, while Samba-common tools facilitate resource sharing and domain interaction between Linux and Windows.

```
sudo dnf install realmd sssd oddjob oddjob-mkhomedir adcli samba-common-tools -y
```
# 2. Configure DNS
You must use the Domain Controller's DNS. In this example: 192.168.100.100.

Persistent configuration via NetworkManager:

```
nmcli con show
nmcli con mod ens160 ipv4.dns 192.168.100.100
nmcli con up ens160
```
Temporary configuration (manual edit):
```
echo "nameserver 192.168.100.100" > /etc/resolv.conf
```
3. Verify Domain Discovery
Check if the Linux machine can see the domain.

```
realm discover test.local
```
If you see domain information in the output, the network configuration is correct.

4. Set FQDN (Fully Qualified Domain Name)
The hostname must be set in the format hostname.domain.com. In this case: rhel.test.local.

```
hostnamectl set-hostname rhel.test.local
hostnamectl # Verify the change
```
Update the /etc/hosts file:


Add this line: [IP Address] [FQDN] [Short Name]
nano /etc/hosts -----> 192.168.100.x rhel.test.local rhel

```reboot```
5. Join the Domain
Run the join command and provide the Domain Administrator's password when prompted.
```
realm join test.local -U Administrator
```

6. Verification
Check Realm Status
```
realm list
```
If successful, you will see domain details (name, type, and "configured: kerberos-member").

Check User Resolution
Verify if Linux can "see" an AD user.

```
id user@test.local
```
A successful output means SSSD and Kerberos are functioning.

Check Kerberos Tickets
```
kinit user@TEST.LOCAL
klist
```
If a ticket is granted, Kerberos is working perfectly.

Test Login
Try switching to the AD user locally or via SSH:

```
su - user@test.local
```
# OR
```
ssh user@test.local@localhost
```
7. Windows Side Verification
Open Active Directory Users and Computers.

Navigate to the Computers container.

If the Linux host (e.g., RHEL) appears in the list, the integration is complete!

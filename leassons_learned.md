# Issues Encountered During the Process

Note: All operations are performed with ROOT privileges.

# 1. Missing kinit Command
If the kinit command is not found, you need to install the Kerberos workstation utilities.
```
sudo dnf install krb5-workstation -y
```
After installation, you can proceed with the authentication check.

# 2. Kerberos Token Validation Error
When attempting to run:
```
kinit user@test.local
```

You might receive the following error:
kinit: KDC reply did not match expectations while getting initial credentials

Potential Causes:

Incorrect username or password.

Misconfiguration in the Kerberos configuration file (/etc/krb5.conf). Specifically, Kerberos may not know which domain to use by default.

Solution:
Edit the default_realm field in the configuration file:
```
nano /etc/krb5.conf
```
Change the following field (ensure it is in UPPERCASE):    default_realm = TEST.LOCAL

Try again:
```
kinit user@TEST.LOCAL #Вводим credentials
klist
```

If a token is displayed, your Kerberos configuration is now correct!

# 3. SSSD Configuration
If the automatic SSSD configuration fails, you must manually edit the /etc/sssd/sssd.conf file.
```nano /etc/sssd/sssd.conf```
Adjust the configuration to match the following structure:
```
[sssd]
domains = test.local
config_file_version = 2
services = nss, pam

[domain/test.local]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = TEST.LOCAL
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u@%d
ad_domain = test.local
use_fully_qualified_names = True
ldap_id_mapping = True
access_provider = ad
```
Key fields to verify:
domains = test.local
krb5_realm = TEST.LOCAL

After editing, restart and enable the SSSD service to apply the changes:
```
sudo systemctl restart sssd
sudo systemctl enable sssd
```




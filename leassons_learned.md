# Проблемы, с которыми я столкнулся в процессе


# Я все делаю от имени ROOT

# 1. Отсутствие пакета kinit 
```
sudo dnf install krb5-workstation -y
```
Затем можно попробовать еще раз.

# 2. Ошибка при проверке Kerberos токена

Затем пробуем:
```
kinit user@test.local
```

Выводится ошибка:
kinit: KDC reply did not match expectations while getting initial credentials

В этом случае либо мы ввели неправильные имя юзера или пароль юзера, или же дело в конфигурационном файле Kerberos /etc/krb5.conf
И так мы должны отредактировать поле #default_realm, так как Kerberos не понимает какой домен используется по умолчанию.
```
nano /etc/krb5.conf
```
Меняем поле:    default_realm = TEST.LOCAL

Пробуем снова:
```
kinit user@TEST.LOCAL #Вводим credentials
klist
```

Если выводится токен, то все в порядке с Kerberos конфигурацией!

# 3. Конфигурация SSSD
Если не произошла автоматическая конфигурация SSSD, то нам нужно отредактировать конфигурационный файл /etc/sssd/sssd.conf
```nano /etc/sssd/sssd.conf```
Отредактируем так, чтобы все выглядело так:
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
Основные редактирования делаем в полях:
domains = test.local
krb5_realm = TEST.LOCAL

Затем следует только перезапустить сервис SSSD:
```
sudo systemctl restart sssd
sudo systemctl enable sssd
```




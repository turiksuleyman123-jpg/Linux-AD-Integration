# Problems i faced in process


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

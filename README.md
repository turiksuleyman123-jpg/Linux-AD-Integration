# Linux-AD-Integration
Интеграция Linux с AD (Active Directory) Windows Server 2022

#Цель
Подключить Linux (RHEL/CentOS/Ubuntu) к Active Directory для единой аутентификации пользователей.

## ОС и ПО
- Linux: RHEL 10
- AD: Windows Server 2022
- Пакеты на Linux: sssd, realmd, krb5-workstation, samba, oddjob, oddjob-mkhomedir

# Я все делаю от лица ROOT
# 1.Установка нужных пакетов
Установка пакетов SSSD, Samba: для аутентификации в AD (SSSD) и совместного использования — папок и принтеров между Linux и Windows(SAMBA).

```sudo dnf install realmd sssd oddjob oddjob-mkhomedir adcli samba-common-tools -y```
# 2.Настройка DNS

#Нужно использовать DNS домена. В моем случае 192.168.100.100

```
nmcli con show
nmcli con mod ens160 ipv4.dns 192.168.100.100
nmcli con up ens160
```

#или же временно:
```
echo "nameserver  192.168.100.100" > /etc/resolv.conf
```
# 3.Проверяем домен
```
realm discover test.local
```
#Если видим информацию о домене, то все ок!


# 4.Меняем FQDN для подключения к домену.(если заранее не поменяли)

Нужно поставить hostname в виде: rhel.example.com
А вместо example.com вставляем реальный домен нашего AD. В моем случае:  rhel.test.local

```
hostnamectl set-hostname rhel.test.local
hostnamectl #проверка
```

Нужно добавить строку в /etc/hosts
```
nano /etc/hosts -----> 192.168.100.x rhel.test.local rhel
reboot
```

# 6.Подключаемся к домену
```
realm join test.local -U Administrator
```
#Вводим пароль администратора домена и входим в домен

# 7.Проверка
```
realm list
```
#Если ты реально в домене, то увидишь инфу о домене.(имя, тип, configured:)

```
id user@test.local
```
#Если вывод есть, значит LINUX видит пользователя, работает SSSD, работает Kerberos.

```
kinit user@TEST.LOCAL
klist
```
#Если получим ticket, значит Kerberos работает

#Пробуем залогиниться
```
su - user@test.local
```
или же через SSH
```
ssh user@test.local@localhost
```
# Проверяем на стороне Windows
Открываем Active Directory Users and Computer
Заходим в Computers

Если появился наш Linux-хост(например: RHEL), то все, МЫ В ДОМЕНЕ!

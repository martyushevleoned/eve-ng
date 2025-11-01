# Стенд ГИС

Требуется построить сеть государственной информационной системы:
* VPN во внутреннюю сеть
* Captive portal для доступа к внешней сети
* IDS
* Межсетевой экран

## Разворачиваем виртуалку

Используемые образы:
* [eve-ng](https://customers.eve-ng.net/eve-ce-prod-6.2.0-4-full.iso) - среда виртуализации
* ubuntu desktop - пользовательские устройства (root/Test123)
* ubuntu server - узлы сети с системным ПО (root/Test123)
* с7200 - маршрутизатор

Закидываем образы
```shell
scp -r ./local root@192.168.0.85:/opt/uetlab/addons # образы
/opt/wrappers/unl_wrapper -a fixpermissions # фикс прав доступа 
```

## Настройка статической маршрутизации

### Firewall линукс сервер с выходом в интернет:
Функции
* Межсетевой экран
* Зеркалирование трафика на IDS сервер
* Статическая маршрутизация
```shell
nano /etc/netplan/00-installer-config.yaml

# конфиг
network:
  version: 2
  ethernets:
    ens3: # интернет
      dhcp4: true
    ens4: # IDS
      addresses: [192.168.2.1/24]
    ens5: # внутренняя сеть
      addresses: [192.168.3.1/24]
      routes:
        - to: 192.168.4.0/24 # Пользователь
          via: 192.168.3.2
        - to: 192.168.5.0/24 # VPN
          via: 192.168.3.2
        - to: 192.168.6.0/24 # Auth
          via: 192.168.3.2

sudo netplan apply # применить
ping 8.8.8.8 # проверка

# включаем ip forwarding
nano /etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
sysctl -p

# настриваем NAT и Разрешаем форвардинг
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o ens3 -j MASQUERADE
iptables -P FORWARD ACCEPT
iptables -t nat -L -n -v

# Костыль для сохранения правил при перезагрзке
apt install iptables-persistent -y
netfilter-persistent save
```

### IDS сервер
```shell
nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens3:
      addresses: [192.168.2.2/24]
      routes:
        - to: 0.0.0.0/0
          via: 192.168.2.1
netplan apply
```

### маршрутизатор в пользовательской сети
* обеспечивает связность устройств во внутренней сети
```shell
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 192.168.3.1

interface e1/0 # firewall
 ip address 192.168.3.2 255.255.255.0
 no shutdown
 exit

interface e1/1 # пользователь
 ip address 192.168.4.1 255.255.255.0
 no shutdown
 exit

interface e1/2 # vpn
 ip address 192.168.5.1 255.255.255.0
 no shutdown
 exit

interface e1/3 # auth
 ip address 192.168.6.1 255.255.255.0
 no shutdown
 exit

exit
copy running-config startup-config
```

### Пользователь
* имитирует пользователя внутренней сети
```shell
sudo nano /etc/netplan/01-network-manager.yaml

network:
  version: 2
  ethernets:
    ens3:
      addresses: [192.168.4.2/24]
      routes
        - to: 0.0.0.0/0
          via: 192.168.4.1

sudo netplan apply
```

### VPN сервер
* обеспечивает доступ к ресурсам внутренней сети из внешней сети
```shell
nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 192.168.5.2/24  
      routes
        - to: 0.0.0.0/0
          via: 192.168.5.1

netplan apply
```

### Auth сервер
* Обеспечивает доступ к внешней сети из внутренней по реквизитам
```shell
nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 192.168.6.2/24  
      routes
        - to: 0.0.0.0/0
          via: 192.168.6.1

netplan apply
```

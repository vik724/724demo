# Методичка к демоэкзамену

Данное руководство содержит пошаговые инструкции по настройке сетевого оборудования и серверов для демонстрационного экзамена.  
Все команды выполняются от **root** или с использованием `sudo`.

## Топология и таблица адресации

Требования к подсетям (Модуль 1, п.1):

| Сегмент | VLAN | Требование | Маска | Подсеть |
| --- | --- | --- | --- | --- |
| HQ — SRV | 111 | не более 32 адресов | /27 | 192.168.1.0/27 |
| HQ — CLI | 211 | не менее 16 адресов | /27 | 192.168.2.0/27 |
| HQ — MGMT | 811 | не более 8 адресов | /29 | 192.168.3.0/29 |
| BR — SRV | — | не более 16 адресов | /28 | 192.168.4.0/28 |
| ISP ↔ HQ-RTR | — | задано | /28 | 172.16.10.0/28 |
| ISP ↔ BR-RTR | — | задано | /28 | 172.16.20.0/28 |
| GRE tunnel | — | служебная | /30 | 10.10.0.0/30 |

| Устройство | Интерфейс | IP-адрес | Шлюз | Назначение |
| --- | --- | --- | --- | --- |
| ISP | ens192 | DHCP | — | INTERNET |
|  | ens224 | 172.16.10.1/28 | — | ISP↔HQ |
|  | ens256 | 172.16.20.1/28 | — | ISP↔BR |
| HQ-RTR | ens192 | 172.16.10.2/28 | 172.16.10.1 | uplink |
|  | ens224.111 | 192.168.1.1/27 | — | VLAN 111 |
|  | ens224.211 | 192.168.2.1/27 | — | VLAN 211 |
|  | ens224.811 | 192.168.3.1/29 | — | VLAN 811 |
|  | tun1 | 10.10.0.1/30 | — | GRE → BR-RTR |
| HQ-SRV | ens192 | 192.168.1.2/27 | 192.168.1.1 | VLAN 111 |
| HQ-CLI | ens192 | DHCP | 192.168.2.1 | VLAN 211 |
| BR-RTR | ens192 | 172.16.20.2/28 | 172.16.20.1 | uplink |
|  | ens224 | 192.168.4.1/28 | — | BR-NET |
|  | tun1 | 10.10.0.2/30 | — | GRE → HQ-RTR |
| BR-SRV | ens192 | 192.168.4.2/28 | 192.168.4.1 | BR-NET |

DNS-записи (Модуль 1, п.10) — прямая зона `au-team.irpo`:
`hq-rtr → 192.168.1.1`, `hq-srv → 192.168.1.2`, `hq-cli → DHCP-выданный`,
`br-rtr → 192.168.4.1`, `br-srv → 192.168.4.2`, `web → 172.16.10.1`, `docker → 172.16.20.1`.

Коллега SirKitman и его помощники oversans и valerarybalov 
сайт А: https://github.com/SirKitman/demos2006
сайт Б: https://codeberg.org/SirKitman/demos2006

---

## <p align="center"><b>Устройство ISP</p>

### Имя устройства
```bash
hostnamectl set-hostname isp.au-team.irpo; newgrp
```
![Screenshot](assets/1.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
![Screenshot](assets/2.png)

### Настройка сетевых адресов
```bash
nano /etc/network/interfaces
```
Перепишите файл к виду:
```ini
allow-hotplug ens192
iface ens192 inet dhcp

allow-hotplug ens224
iface ens224 inet static
address 172.16.10.1
netmask 255.255.255.240

allow-hotplug ens256
iface ens256 inet static
address 172.16.20.1
netmask 255.255.255.240
```
![Screenshot](assets/3.png)

Перезапустим сеть:
```bash
systemctl restart networking
```
И проверим выставление IP:
```bash
ip a
```

### Временная настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Вставляем временное содержимое:
```
nameserver 1.1.1.1
```
![Screenshot](assets/8.png)

> [!NOTE]
> Перед началом следует обновить пакеты на устройстве командой:
> ```bash
> apt-get update -y
> ```

### Переадресация (NAT)
Вставляем строку `net.ipv4.ip_forward=1`, читаем файл и применяем в системе:
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf; sysctl -p
```
![Screenshot](assets/6.png)

Установите iptables:
```bash
apt install iptables iptables-persistent -y
```
И прописываем правило:
```bash
iptables -t nat -A POSTROUTING -s 172.16.10.0/28 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.20.0/28 -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
```

![Screenshot](assets/7.png)

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```
---
## <p align="center"><b>Устройство HQ-RTR</p>

### Имя устройства
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo; newgrp
```
![Screenshot](assets/5.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
![Screenshot](assets/2.png)

### Настройка VLAN и интерфейсов
Отредактируйте `/etc/network/interfaces`:
```bash
nano /etc/network/interfaces
```
Пример содержания:
```ini
allow-hotplug ens192
iface ens192 inet static
address 172.16.10.2
netmask 255.255.255.240
gateway 172.16.10.1

allow-hotplug ens224
iface ens224 inet static
address 192.168.1.1
netmask 255.255.255.224
```
> [!NOTE]
> Писать именно до этого момента, если заполните все, конфиг будет ругаться на то что нет VLAN, и не сможет скачать VLAN :)

Перезапустим сеть:
```bash
systemctl restart networking
```

### Временная настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Временное содержимое:
```
nameserver 1.1.1.1
```
![Screenshot](assets/8.png)

> [!NOTE]
> Перед началом следует обновить устройство командой
> ```bash
> apt-get update -y
> ```

Установите VLAN:
```bash
apt install vlan -y
```
![Screenshot](assets/9.png)

Выдайте поддержку VLAN
```bash
modprobe 8021q
echo 8021q >> /etc/modules
```
![Screenshot](assets/10.png)

Отредактируйте `/etc/network/interfaces`:
```bash
nano /etc/network/interfaces
```
Продолжите заполнять содержимое:
```bash
allow-hotplug ens224:1
iface ens224:1 inet static
address 192.168.2.1
netmask 255.255.255.224

allow-hotplug ens224:2
iface ens224:2 inet static
address 192.168.3.1
netmask 255.255.255.248

allow-hotplug ens224.111
iface ens224.111 inet static
address 192.168.1.3
netmask 255.255.255.224
vlan_raw_device ens224

allow-hotplug ens224.211
iface ens224.211 inet static
address 192.168.2.3
netmask 255.255.255.224
vlan_raw_device ens224:1

allow-hotplug ens224.811
iface ens224.811 inet static
address 192.168.3.3
netmask 255.255.255.248
vlan_raw_device ens224:2

allow-hotplug tun1
iface tun1 inet tunnel
address 10.10.0.1
netmask 255.255.255.252
mode gre
local 172.16.10.2
endpoint 172.16.20.2
ttl 64
```
![Screenshot](assets/11.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Добавление пользователя, пароль `P@ssw0rd`:
```bash
adduser net_admin
```
![Screenshot](assets/23.png)

Выдайте привилегии через `visudo`:
```bash
visudo
```
Под root пользователем добавьте строки:
```
net_admin ALL=(ALL:ALL) ALL
sshuser ALL=(ALL:ALL) ALL
```
![Screenshot](assets/25.png)

### Переадресация (NAT)
Вставляем строку `net.ipv4.ip_forward=1`, читаем файл и применяем в системе:
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf; sysctl -p
```
![Screenshot](assets/6.png)

### Временная настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Временное содержимое:
```
nameserver 1.1.1.1
```
![Screenshot](assets/8.png)

Установите iptables:
```bash
apt install iptables iptables-persistent -y
```
И прописываем правило:
```bash
iptables -t nat -A POSTROUTING -s 192.168.1.0/27 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.2.0/27 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.3.0/29 -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
```
![Screenshot](assets/28.png)

### Настройка GRE туннеля
Добавьте модуль `ip_gre` в автозагрузку:
```bash
echo "ip_gre" >> /etc/modules
```
![Screenshot](assets/33.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Динамическая маршрутизация (OSPF)
Установите FRRouting:
```bash
apt install frr -y
```
![Screenshot](assets/34.png)

Включите демон OSPF:
```
nano /etc/frr/daemons
```
Где меняем ospfd на статус yes:
```bash
ospfd=yes
```
![Screenshot](assets/35.png)

Перезапустите FRR:
```bash
systemctl restart frr
```

Настройте OSPF через `vtysh`:
```bash
vtysh
```
Команды для настройки:
```bash
conf t
router ospf
router-id 10.10.0.1
passive-interface default
network 192.168.1.0/27 area 0
network 192.168.2.0/27 area 0
network 10.10.0.0/30 area 0
exit
interface tun1
no ip ospf passive
ip ospf area 0
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
do write
exit
write memory
wr
exit
```
![Screenshot](assets/36.png)

Перезапускаем службу:
```bash
systemctl restart frr
```
Перезапустите сеть:
```bash
systemctl restart networking
```
Добавляем frr в автозагрузку:
```bash
systemctl enable frr
```

>[!IMPORTANT]
>ОБЯЗАТЕЛЬНО ВВЕСТИ ЭТУ КОМАНДУ ДЛЯ ПРОВЕРКИ КОНФИГА
>`vtysh -c "show running-config"`

### DHCP-сервер на HQ-RTR
Установите DHCP-сервер:
```bash
apt install isc-dhcp-server -y
```
![Screenshot](assets/38.png)

Укажите интерфейс в конфиге для DHCP:
```bash
nano /etc/default/isc-dhcp-server
```
То что туда нужно вписать:
```ini
INTERFACESv4="ens224:1"
```
![Screenshot](assets/40.png)

Настройте пул в `/etc/dhcp/dhcpd.conf`:
```bash
nano /etc/dhcp/dhcpd.conf
```
Добавьте:
```
subnet 192.168.2.0 netmask 255.255.255.224 {
    range 192.168.2.4 192.168.2.19;
    option domain-name-servers 192.168.1.2;
    option domain-name "au-team.irpo";
    option routers 192.168.2.1;
    default-lease-time 600;
    max-lease-time 7200;
}
```
![Screenshot](assets/41.png)

Перезапустите сеть:
```bash
systemctl restart networking
```
Перезапустите службу:
```bash
systemctl restart isc-dhcp-server
```
Включаем в автозапуск:
```bash
systemctl enable isc-dhcp-server
```

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```

---

## <p align="center"><b>Устройство BR-RTR</p>

### Имя устройства
```bash
hostnamectl set-hostname br-rtr.au-team.irpo; newgrp
```
![Screenshot](assets/4.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
![Screenshot](assets/2.png)

### Настройка адресов
```bash
nano /etc/network/interfaces
```
Пример:
```ini
allow-hotplug ens192
iface ens192 inet static
address 172.16.20.2
netmask 255.255.255.240
gateway 172.16.20.1

allow-hotplug ens224
iface ens224 inet static
address 192.168.4.1
netmask 255.255.255.240

allow-hotplug tun1
iface tun1 inet tunnel
address 10.10.0.2
netmask 255.255.255.252
mode gre
local 172.16.20.2
endpoint 172.16.10.2
ttl 64
```
![Screenshot](assets/16.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Временная настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Временное содержимое:
```
nameserver 1.1.1.1
```
![Screenshot](assets/8.png)

### Добавление пользователя пароль `P@ssw0rd`:
```bash
adduser net_admin
```
![Screenshot](assets/24.png)

Выдайте привилегии через `visudo`:
```bash
visudo
```
Под root пользователем добавьте строки:
```
net_admin ALL=(ALL:ALL) ALL
sshuser ALL=(ALL:ALL) ALL
```
![Screenshot](assets/25.png)

### Переадресация (NAT)
Вставляем строку `net.ipv4.ip_forward=1`, читаем файл и применяем в системе:
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf; sysctl -p
```
![Screenshot](assets/6.png)

> [!NOTE]
> Перед началом следует обновить устройство командой
> ```bash
> apt-get update -y
> ```

Установите iptables:
```bash
apt install iptables iptables-persistent -y
```
И прописываем правило:
```bash
iptables -t nat -A POSTROUTING -s 192.168.4.0/28 -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
```
![Screenshot](assets/29.png)

Добавьте модуль `ip_gre` в автозагрузку:
```bash
echo "ip_gre" >> /etc/modules
```
![Screenshot](assets/33.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Динамическая маршрутизация (OSPF)
Установите FRRouting:
```bash
apt install frr -y
```
![Screenshot](assets/34.png)

Включите демон OSPF:
```
nano /etc/frr/daemons
```
Где меняем ospfd на статус yes:
```bash
ospfd=yes
```
![Screenshot](assets/35.png)

Перезапустите FRR:
```bash
systemctl restart frr
```

Настройте OSPF через `vtysh`:
```bash
vtysh
```
Команды для настройки:
```bash
conf t
router ospf
router-id 10.10.0.2
passive-interface default
no passive-interface tun1
network 192.168.4.0/28 area 0
network 10.10.0.0/30 area 0
exit
interface tun1
ip ospf area 0
ip ospf authentication
ip ospf authentication-key P@ssw0rd
exit
do write
exit
write memory
exit
```
![Screenshot](assets/37.png)

Перезапускаем службу:
```bash
systemctl restart frr
```
Перезапустите сеть:
```bash
systemctl restart networking
```
Добавляем frr в автозагрузку:
```bash
systemctl enable frr
```

>[!IMPORTANT]
>ОБЯЗАТЕЛЬНО ВВЕСТИ ЭТУ КОМАНДУ ДЛЯ ПРОВЕРКИ КОНФИГА
>`vtysh -c "show running-config"`

>[!NOTE]
>После этого пингануть HQ-RTR командой `ping 10.10.0.1`

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```

---

## <p align="center"><b>Устройство HQ-SRV</p>

### Имя устройства
```bash
hostnamectl set-hostname hq-srv.au-team.irpo; newgrp
```
![Screenshot](assets/12.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
![Screenshot](assets/2.png)

### Настройка адресов
```bash
nano /etc/network/interfaces
```
```ini
allow-hotplug ens192
iface ens192 inet static
address 192.168.1.2
netmask 255.255.255.224
gateway 192.168.1.1
```
![Screenshot](assets/13.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Временная настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Временное содержимое:
```
nameserver 1.1.1.1
```
![Screenshot](assets/8.png)

> [!CAUTION]
> Перед началом следует обновить устройство командой
> ```bash
>apt-get update -y
> ```

### Добавление пользователя, пароль `P@ssw0rd`:
```bash
adduser sshuser -u 2011
```
![Screenshot](assets/27.png)

Выдайте привилегии через `visudo`:
```bash
visudo
```
Добавьте строку:
```
sshuser ALL=(ALL:ALL) ALL
```
![Screenshot](assets/21.png)

### Настройка SSH
```bash
nano /etc/ssh/sshd_config
```
Внесите изменения:
```
Port 2011
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh-banner
```
![Screenshot](assets/30.png)

Создайте баннер:
```bash
nano /etc/ssh-banner
```
```
Authorized access only
```
![Screenshot](assets/32.png)

Перезапустите SSH:
```bash
systemctl restart sshd
```

### Настройка DNS-сервера (BIND9)
Установите BIND:
```bash
apt install bind9 dnsutils -y
```
![Screenshot](assets/44.png)

#### 1. Файл `/etc/bind/named.conf.options`
```bash
nano /etc/bind/named.conf.options
```
```nginx
    listen-on port 53 { any; };
    listen-on-v6 port 53 { any; };
    directory "/var/cache/bind";
    allow-query { any; };
    allow-recursion { any; };
    forwarders { 77.88.8.8; 77.88.8.7; 1.1.1.1; };
    forward only;
    dnssec-validation no;
};
```
![Screenshot](assets/56.png)

#### 2. Файл `/etc/bind/named.conf.default-zones`
Добавьте в конец:
```bash
nano /etc/bind/named.conf.default-zones
```
```nginx
zone "au-team.irpo" {
    type master;
    file "/etc/bind/au-team.irpo";
    allow-query { any; };
    allow-transfer { any; };
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/au-team.irpo_obr";
};

zone "2.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/au-team.irpo_hqobr";
};
```
![Screenshot](assets/49.png)

#### 3. Прямая зона `au-team.irpo`
Скопируйте шаблон:
```bash
cp /etc/bind/db.local /etc/bind/au-team.irpo
nano /etc/bind/au-team.irpo
```

>[!IMPORTANT]
>**У HQ-CLI может быть другой IP т.к. у него DHCP, проверить у него IP командой ip a**

Приведите к виду:
```dns
$TTL    604800
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      hq-srv.au-team.irpo.
hq-srv  IN      A       192.168.1.2
hq-rtr  IN      A       192.168.1.1
hq-cli  IN      A       192.168.2.postav_IP
br-rtr  IN      A       192.168.4.1
br-srv  IN      A       192.168.4.2
docker  IN      A       172.16.20.1
web     IN      A       172.16.10.1
```
![Screenshot](assets/50.png)

#### 4. Обратная зона для 192.168.1.0/26
```bash
cp /etc/bind/db.127 /etc/bind/au-team.irpo_obr
nano /etc/bind/au-team.irpo_obr
```
```dns
$TTL    604800
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      hq-srv.au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
2       IN      PTR     hq-srv.au-team.irpo.
```
![Screenshot](assets/51.png)

#### 5. Обратная зона для 192.168.2.0/28

>[!IMPORTANT]
>**У HQ-CLI может быть другой IP т.к. у него DHCP, проверить у него IP командой ip a**

Скопируйте предыдущую и отредактируйте:
```bash
cp /etc/bind/au-team.irpo_obr /etc/bind/au-team.irpo_hqobr
nano /etc/bind/au-team.irpo_hqobr
```
```dns
$TTL    604800
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      hq-srv.au-team.irpo.
Posledniy_oktet_IP       IN      PTR     hq-cli.au-team.irpo.
```
![Screenshot](assets/53.png)

#### 6. Проверка конфигурации
```bash
named-checkconf
named-checkzone au-team.irpo /etc/bind/au-team.irpo
named-checkzone 1.168.192.in-addr.arpa /etc/bind/au-team.irpo_obr
named-checkzone 2.168.192.in-addr.arpa /etc/bind/au-team.irpo_hqobr
```
Ожидайте вывод **OK**. Если выдает ошибки, то поменяй IP у CLI в настройках обратной зоны и прямой зоны

Перезапустите BIND:
```bash
systemctl restart bind9
```

>[!IMPORTANT]
>**На DNS не грешить, конфигурация была проверена много раз с использованием нескольких средств виртуализации, если что-либо идет не так, виноваты кривые руки**

#### 7. Проверка работы DNS (проверка на **HQ-SRV**, **HQ-RTR**, **HQ-CLI**)
Переходим в resolv.conf:
```bash
nano /etc/resolv.conf
```
Вставляем в конфиг:
```bash
search au-team.irpo
nameserver 192.168.1.2
```
Выходим и прописываем:
```bash
dig -x 192.168.1.1 @192.168.1.2
dig -x 192.168.1.2 @192.168.1.2
```
Или:
```bash
nslookup 192.168.1.2
nslookup 192.168.1.1
```
![Screenshot](assets/55.png)

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```

---

## <p align="center"><b>Устройство HQ-CLI</p>

### Имя устройства
```bash
hostnamectl set-hostname hq-cli.au-team.irpo; newgrp
```
![Screenshot](assets/17.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
![Screenshot](assets/2.png)

### Настройка адресов (DHCP)
```bash
nano /etc/network/interfaces
```
```ini
allow-hotplug ens192
iface ens192 inet dhcp
```
![Screenshot](assets/42.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

### Настройка DNS
```bash
nano /etc/resolv.conf
```
```
search au-team.irpo
nameserver 192.168.1.2
```
![Screenshot](assets/54.png)

> [!CAUTION]
> Также следует обновить пакеты командой
> ```bash
>apt-get update -y
> ```

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```

---

## <p align="center"><b>Устройство BR-SRV</p>

### Имя устройства
```bash
hostnamectl set-hostname br-srv.au-team.irpo; newgrp
```
![Screenshot](assets/4.png)

### Закомментировать загрузку
```bash
nano /etc/apt/sources.list
```
### Настройка DNS серверов 
```bash
nano /etc/resolv.conf
```
Постоянное содержимое:
```
search au-team.irpo
nameserver 192.168.1.2
```
![Screenshot](assets/54.png)

### Настройка адресов
```bash
nano /etc/network/interfaces
```
```ini
allow-hotplug ens192
iface ens192 inet static
address 192.168.4.2
netmask 255.255.255.240
gateway 192.168.4.1
```
![Screenshot](assets/15.png)

Перезапустите сеть:
```bash
systemctl restart networking
```

> [!CAUTION]
> Перед началом следует обновить устройство командой
> ```bash
>apt-get update -y
> ```

### Добавление пользователя с паролем `P@ssw0rd`
```bash
adduser sshuser -u 2011
```
![Screenshot](assets/19.png)

Выдайте привилегии через `visudo`:
```bash
visudo
```
Добавьте строку:
```
sshuser ALL=(ALL:ALL) ALL
```
![Screenshot](assets/21.png)

### Настройка SSH
```bash
nano /etc/ssh/sshd_config
```
```
Port 2011
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh-banner
```
![Screenshot](assets/30.png)

Создайте баннер:
```bash
nano /etc/ssh-banner
```
```
Authorized access only
```
![Screenshot](assets/32.png)

Перезапустите SSH:
```bash
systemctl restart sshd
```

### Настройка часового пояса
```bash
timedatectl set-timezone Asia/Tomsk
```
Проверка:
```bash
timedatectl
```

---

> [!NOTE]
> Самое главное, после всего этого, на HQ-RTR, HQ-SRV, HQ-CLI, BR-RTR, BR-SRV выставить новый DNS
```
search au-team.irpo
nameserver 192.168.1.2
```
![Screenshot](assets/54.png)

Зеркала Яндекса
```bash
deb http://mirror.yandex.ru/debian/ bookworm main
deb-src http://mirror.yandex.ru/debian/ bookworm main
deb http://mirror.yandex.ru/debian-security bookworm-security main contrib
deb-src http://mirror.yandex.ru/debian-security bookworm-security main contrib
deb http://mirror.yandex.ru/debian/ bookworm-updates main contrib
deb-src http://mirror.yandex.ru/debian/ bookworm-updates main contrib
```

>[!IMPORTANT]
>Доп команды:
>
>Разрыв связей resolv.conf, далее поможет с ISP
>```bash
>unlink /etc/resolv.conf
>```
>Можно посмотрть сохраненные правила iptables через команду:
>```bash
>iptables -L -t nat
>```
>Если вы по какой то причине перезагрузили устройство, не забывайте про команду
>```bash
>sysctl -p
>```
>чтобы применить переадресацию NAT
>
>Иногда виртуалки зависают так как вставка команды идет построчно, это нормально

>[!WARNING]
>До приступления к выполнению второго модуля убедитесь что:
>
>Все пакеты на рабочих машинах обновленны, командой `apt-get update -y`: Должно быть обновленно 0 пакетов
>
>Доступ в интернет и сетевая связанность не нарушена путем команд:
>
>С HQ-CLI `ping ya.ru` и `ping 192.168.4.2`
>
>С BR-SRV `ping ya.ru` и `ping 192.168.1.2`
>
>Также нужно проверить работу ssh на HQ-SRV и BR-SRV командами:
>
>С BR-RTR `ssh -p 2011 sshuser@192.168.4.2` и `ping 10.10.0.1`
>
>С HQ-RTR `ssh -p 2011 sshuser@192.168.1.2` и `ping 10.10.0.2`
>
>Проверка BIND9 и DHCP:
>
>На HQ-RTR `systemctl status isc-dhcp-server`
>
>На HQ-SRV `systemctl status bind9`
>
>На всех машинах `timedatectl`
---

## <p align="center"><b>МОДУЛЬ 2 — Организация сетевого администрирования</b></p>

> [!IMPORTANT]
> Чек-лист по 11 пунктам задания (Модуль 2):
> 1. Samba AD DC на BR-SRV (домен au-team.irpo), ввод HQ-CLI, 5 пользователей hquser1..5, группа hq, sudo только для cat/grep/id
> 2. RAID 1 (md1) из двух дисков 1ГБ на HQ-SRV, ext4, монтирование в `/raid`
> 3. NFS `/raid/nfs` на HQ-SRV, rw для сети HQ-CLI; автомонтирование `/mnt/nfs` на клиенте
> 4. chrony на ISP (stratum **6**); клиенты: HQ-SRV, HQ-CLI, BR-RTR, BR-SRV
> 5. Ansible на BR-SRV (inventory: HQ-SRV/HQ-CLI/HQ-RTR/BR-RTR), `/etc/ansible`, `ping → pong`
> 6. Docker stack на BR-SRV: `site` (site_latest) + `db` (mariadb_latest), БД testdb1 / test1 / P@ssw0rd, порт 8081
> 7. Apache + MariaDB + PHP на HQ-SRV, БД webdb (dump.sql), пользователь web1 / P@ssw0rd, index.php + images
> 8. Проброс портов: 8081→site (BR-RTR), 8081→apache (HQ-RTR), 2011→SSH HQ/BR-SRV
> 9. Nginx reverse-proxy на ISP: web.au-team.irpo → HQ-SRV, docker.au-team.irpo → site (BR-SRV)
> 10. Basic Auth на ISP для web.au-team.irpo: Khariton / P@ssw0rd, файл `/etc/nginx/.htpasswd`
> 11. Яндекс.Браузер на HQ-CLI (проверка обоих сайтов)

---

## <p align="center"><b>1. Samba AD DC на BR-SRV</b></p>

### Установка необходимых пакетов
```bash
apt install samba smbclient winbind krb5-user krb5-config -y
```
![Screenshot](assets/59.png)

При установке `krb5-config` будут запрошены параметры:
- **Default Kerberos realm**: `AU-TEAM.IRPO`
- **Kerberos servers**: `br-srv.au-team.irpo`
- **Administrative server**: `br-srv.au-team.irpo`

### Остановка служб и очистка конфигурации
```bash
systemctl stop samba-ad-dc smbd nmbd winbind 2>/dev/null
systemctl disable samba-ad-dc smbd nmbd winbind 2>/dev/null
mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```
![Screenshot](assets/60.png)

### Развёртывание домена
```bash
samba-tool domain provision --use-rfc2307 --interactive
```
Параметры при установке:
- **Realm**: `AU-TEAM.IRPO`
- **Domain**: `AU-TEAM`
- **Server Role**: `dc`
- **DNS backend**: `SAMBA_INTERNAL`
- **DNS forwarder IP**: `192.168.1.2`
- **Administrator password**: `P@ssw0rd`
![Screenshot](assets/61.png)

### Настройка Kerberos
```bash
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

### Запуск Samba AD DC
```bash
systemctl mask smbd nmbd winbind
systemctl unmask samba-ad-dc
systemctl enable samba-ad-dc
systemctl restart samba-ad-dc
systemctl unmask winbind
systemctl restart winbind
```
![Screenshot](assets/62.png)

### Проверка работы домена
```bash
samba-tool domain level show
smbclient -L localhost -N
```
![Screenshot](assets/63.png)

### Создание пользователей и группы
Создайте группу `hq`:
```bash
samba-tool group add hq
```

Создайте 5 пользователей:
```bash
samba-tool user create hquser1 P@ssw0rd
samba-tool user create hquser2 P@ssw0rd
samba-tool user create hquser3 P@ssw0rd
samba-tool user create hquser4 P@ssw0rd
samba-tool user create hquser5 P@ssw0rd
```

Добавьте пользователей в группу `hq`:
```bash
samba-tool group addmembers hq hquser1,hquser2,hquser3,hquser4,hquser5
```
![Screenshot](assets/64.png)

### Ограничение команд для группы hq
Создайте каталог и символические ссылки для разрешённых команд:
```bash
mkdir -p /usr/local/bin/restricted
ln -sf /usr/bin/cat /usr/local/bin/restricted/cat
ln -sf /usr/bin/grep /usr/local/bin/restricted/grep
ln -sf /usr/bin/id /usr/local/bin/restricted/id
```

Создайте скрипт `/usr/local/bin/restricted_shell.sh`:
```bash
nano /usr/local/bin/restricted_shell.sh
```
```bash
#!/bin/bash
export PATH="/usr/local/bin/restricted"
exec /bin/rbash
```
```bash
chmod +x /usr/local/bin/restricted_shell.sh
```

> [!NOTE]
> Не пользовался, но есть - после ввода пользователей в домен и входа на HQ-CLI, ограничение команд будет применяться через назначение этого скрипта как оболочки.
> На BR-SRV для каждого пользователя можно задать оболочку:
> ```bash
> samba-tool user setattr hquser1 loginShell /usr/local/bin/restricted_shell.sh
> samba-tool user setattr hquser2 loginShell /usr/local/bin/restricted_shell.sh
> samba-tool user setattr hquser3 loginShell /usr/local/bin/restricted_shell.sh
> samba-tool user setattr hquser4 loginShell /usr/local/bin/restricted_shell.sh
> samba-tool user setattr hquser5 loginShell /usr/local/bin/restricted_shell.sh
> ```
> Также скопируйте restricted_shell.sh и каталог /usr/local/bin/restricted на HQ-CLI.

## <p align="center"><b>Ввод HQ-CLI в домен</b></p>

>[!WARNING]
> НА УСТРОЙСТВЕ HQ-CLI ПРИ ВСТАВКЕ МЕНЯЮТСЯ СИМВОЛЫ, СЛЕДИТЕ ВНИМАТЕЛЬНО ПЕРЕД НАЖАТИЕМ КЛАВИШИ ВВОДА
>
> НАСТОЯТЕЛЬНО РЕКОМЕНДУЮ СДЕЛАТЬ СНИМОК СОСТОЯНИЯ УСТРОЙСТВА ПЕРЕД ПРОДОЛЖЕНИЕМ ВЫПОЛНЕНИЯ ЗАДАНИЯ

На **HQ-CLI** установите необходимые пакеты:
```bash
apt install realmd sssd sssd-tools adcli packagekit samba-common-bin -y
```
![Screenshot](assets/65.png)

Настройте DNS на HQ-CLI для использования BR-SRV:
```bash
nano /etc/resolv.conf
```
```
search au-team.irpo
nameserver 192.168.4.2
```

Введите машину в домен:
```bash
/usr/sbin/realm join AU-TEAM.IRPO -U Administrator
# Пароль: P@ssw0rd
```
![Screenshot](assets/67.png)

>[!WARNING]
>При добавлении в домен может возникнуть проблема с зависанием процесса
>Нужно ввести в новом окне терминала  /usr/sbin/realm join AU-TEAM.IRPO -U Administrator
>После ввода пароля вы увидете надпись с командой journalctl REALMD_OPERATION=r1333.5246
>Введя эту команду вы увидете процесс после слова realmd Например 5030
>Удаляете этот процесс командой kill
>И заново вводите команду для ввода в домен
>![Screenshot](assets/118.png)

Проверка:
```bash
/usr/sbin/realm list
id hquser1@au-team.irpo
```
![Screenshot](assets/68.png)

Настройте автоматическое создание домашних каталогов:
```bash
nano /etc/pam.d/common-session
```
Добавьте строку:
```
session required    pam_mkhomedir.so umask=0077 skel=/etc/skel
```
![Screenshot](assets/69.png)

Проверьте вход под доменным пользователем:
```bash
su - hquser1@au-team.irpo
```
![Screenshot](assets/70.png)

> [!NOTE]
> Если не входит под пользователем, пропустите и работайте над другими заданиями
> После ввода в домен верните DNS обратно:
> ```
> search au-team.irpo
> nameserver 192.168.1.2
> ```

---

## <p align="center"><b>2. RAID-массив на HQ-SRV</b></p>

> Согласно заданию: RAID **уровня 1** (зеркало), имя устройства **md1**, конфигурация в `/etc/mdadm.conf`.

### Установка mdadm
```bash
apt install mdadm -y
```
![Screenshot](assets/71.png)

### Просмотр доступных дисков
```bash
lsblk
```
> На виртуалке два доп. диска по 1 ГБ. Предположим, что доступны диски `/dev/sdb` и `/dev/sdc`.
![Screenshot](assets/73.png)

### Создание RAID 1 (разбиение)
```bash
mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```
Подтвердите: `y`

### Проверка состояния RAID
```bash
cat /proc/mdstat
mdadm --detail /dev/md1
```
![Screenshot](assets/74.png)

### Сохранение конфигурации (это может занять время)
```bash
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u
```

### Создание файловой системы ext4
```bash
mkfs.ext4 /dev/md1
```

### Монтирование в /raid
```bash
mkdir -p /raid
mount /dev/md1 /raid
```

Добавьте в `/etc/fstab` для автомонтирования:
```bash
echo '/dev/md1 /raid ext4 defaults 0 0' >> /etc/fstab
```

### Проверка
```bash
df -h /raid
```
![Screenshot](assets/75.png)

---

## <p align="center"><b>3. NFS (сетевая файловая система) на HQ-SRV</b></p>

### Установка NFS-сервера на HQ-SRV
```bash
apt install nfs-kernel-server -y
```
![Screenshot](assets/76.png)

### Создание каталога для NFS
```bash
mkdir -p /raid/nfs
chmod 777 /raid/nfs
```

### Настройка экспорта
```bash
nano /etc/exports
```
Добавьте:
```
/raid/nfs 192.168.2.0/27(rw,sync,no_subtree_check,no_root_squash)
```
![Screenshot](assets/77.png)

### Применение и запуск
```bash
exportfs -a
systemctl restart nfs-kernel-server
systemctl enable nfs-kernel-server
```
![Screenshot](assets/78.png)

### <p align="center"><b>Настройка NFS-клиента на HQ-CLI</b></p>
```bash
apt install nfs-common -y
```
![Screenshot](assets/79.png)

```bash
mkdir -p /mnt/nfs
mount 192.168.1.2:/raid/nfs /mnt/nfs
```

Добавьте в `/etc/fstab`:
```bash
echo '192.168.1.2:/raid/nfs /mnt/nfs nfs defaults 0 0' >> /etc/fstab
```
![Screenshot](assets/80.png)

### Проверка
На HQ-SRV создайте файл:
```bash
touch /raid/nfs/test_file
```
![Screenshot](assets/81.png)

На HQ-CLI проверьте:
```bash
ls /mnt/nfs/
```
![Screenshot](assets/82.png)

---

## <p align="center"><b>4. Настройка NTP (chrony) на ISP</b></p>

### Установка chrony на ISP
```bash
apt install chrony -y
```
![Screenshot](assets/83.png)

### Настройка ISP как NTP-сервера
```bash
nano /etc/chrony/chrony.conf
```
Замените/добавьте:
```ini
local stratum 6
allow 172.16.0.0/28
allow 172.16.10.0/28
allow 172.16.20.0/28
allow 192.168.0.0/16
```
![Screenshot](assets/84.png)

Перезапустите:
```bash
systemctl restart chrony
systemctl enable chrony
```
![Screenshot](assets/85.png)

### Настройка NTP-клиентов

На **HQ-SRV**, **HQ-CLI**, **HQ-RTR** установите chrony и настройте:
```bash
apt install chrony -y
nano /etc/chrony/chrony.conf
```

Закомментируйте строку `pool` и добавьте:
```ini
server 172.16.10.1 iburst
```
![Screenshot](assets/86.png)

На **BR-RTR** и **BR-SRV**:
```bash
apt install chrony -y
nano /etc/chrony/chrony.conf
```

Закомментируйте строку `pool` и добавьте:
```ini
server 172.16.20.1 iburst
```
![Screenshot](assets/87.png)

Перезапустите на всех клиентах:
```bash
systemctl restart chrony
systemctl enable chrony
```

### Проверка синхронизации (на всех устройствах)
```bash
chronyc sources
chronyc tracking
```
![Screenshot](assets/88.png)

---

## <p align="center"><b>5. Ansible на BR-SRV</b></p>

### Установка Ansible
```bash
apt install ansible sshpass -y
```

### Настройка инвентаря
```bash
mkdir -p /etc/ansible
nano /etc/ansible/hosts
```
![Screenshot](assets/89.png)

>**У HQ-CLI скорее всего другой IPv4**
```ini
[hq]
hq-srv ansible_host=192.168.1.2 ansible_user=sshuser ansible_password=P@ssw0rd ansible_port=2011
hq-rtr ansible_host=192.168.1.1 ansible_user=net_admin ansible_password=P@ssw0rd
hq-cli ansible_host=192.168.2.5 ansible_user=locadm ansible_password=P@ssw0rd

[br]
br-rtr ansible_host=192.168.4.1 ansible_user=net_admin ansible_password=P@ssw0rd
```
![Screenshot](assets/90.png)

### Настройка ansible.cfg
```bash
nano /etc/ansible/ansible.cfg
```
```ini
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
```
![Screenshot](assets/91.png)

### Проверка подключения (ping - pong)
```bash
ansible all -m ping
```
![Screenshot](assets/92.png)

Ожидаемый вывод: `pong` от каждого хоста, кроме isp.

>[!NOTE]
>Если с HQ-CLI не приходит ответ вида **pong**, то возможно следует установить ssh-server `apt install openssh-server`
>
> Для хостов без нестандартного порта SSH порт по умолчанию 22.

---

## <p align="center"><b>6. Docker на BR-SRV</b></p>

### Установка Docker
```bash
apt install docker.io docker-compose -y
systemctl enable docker
systemctl start docker
```
![Screenshot](assets/94.png)

### Просмотр доступных дисков
```bash
lsblk
```
![Screenshot](assets/95.png)

### Загрузка образов из Additional.iso
> Подмонтируйте Additional.iso и загрузите образы (данный образ весит ~918мб):
```bash
mkdir -p /mnt/iso
mount /dev/cdrom /mnt/iso
docker load < /mnt/iso/docker/site_latest.tar
docker load < /mnt/iso/docker/mariadb_latest.tar
```

### Проверка загруженных образов
```bash
docker images # Этой командой можно узнать ТЕГ образа для yml файла
```
![Screenshot](assets/96.png)

### Создание docker-compose.yml
```bash
mkdir -p /opt/site
nano /opt/site/docker-compose.yml
```
> По заданию: основной контейнер называется `site` (образ `site_latest`), БД — `testdb1`, пользователь `test1` с паролем `P@ssw0rd`, порт приложения **8081**.
> Расположение данных `/mnt/iso/docker/readme.txt`
```yaml
version: '3'

services:
  site:
    image: site:latest
    container_name: site
    restart: always
    ports:
      - "8081:8000"
    environment:
      DB_HOST: db
      DB_PORT: "3306"
      DB_NAME: testdb1
      DB_USER: test1
      DB_PASS: P@ssw0rd
      DB_TYPE: maria
    depends_on:
      - db

  db:
    image: mariadb:10.11
    container_name: db
    ports:
      - "3306:3306"
    environment:
      MARIADB_DATABASE: testdb1
      MARIADB_USER: test1
      MARIADB_PASSWORD: P@ssw0rd
      MARIADB_ROOT_PASSWORD: P@ssw0rd
    restart: always
```
![Screenshot](assets/97.png)

### Запуск контейнеров
```bash
cd /opt/site
docker-compose up -d
```
![Screenshot](assets/98.png)

### Проверка
```bash
docker ps
apt install curl -y
curl http://localhost:8081
```
![Screenshot](assets/99.png)

---

## <p align="center"><b>7. Apache + MariaDB + web на HQ-SRV</b></p>

### Установка Apache
```bash
apt install apache2 -y
systemctl enable apache2
systemctl start apache2
```
![Screenshot](assets/100.png)

### Установка MariaDB
```bash
apt install mariadb-server mariadb-client -y
systemctl enable mariadb
systemctl start mariadb
```
![Screenshot](assets/101.png)

### Установка PHP
```bash
apt install php libapache2-mod-php php-mysql -y
```
![Screenshot](assets/102.png)

### Просмотр доступных дисков
```bash
lsblk
```
![Screenshot](assets/103.png)

### Импорт базы данных
> Подмонтируйте Additional.iso (данный образ весит ~918мб):
```bash
mkdir -p /mnt/iso
mount /dev/cdrom /mnt/iso
```

Создайте базу данных и пользователя:
```bash
mysql -u root <<EOF
CREATE DATABASE webdb;
CREATE USER 'web1'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'web1'@'localhost';
FLUSH PRIVILEGES;
EOF
```

Импортируйте дамп:
```bash
mysql -u root webdb < /mnt/iso/web/dump.sql
```
![Screenshot](assets/104.png)

### Размещение файлов сайта
Скопируйте `index.php` и каталог `images` из Additional.iso:
```bash
cp /mnt/iso/web/index.php /var/www/html/
cp -r /mnt/iso/web/logo.png /var/www/html/
chown -R www-data:www-data /var/www/html/
```

> [!NOTE]
> Убедитесь, что `index.php` содержит правильные параметры подключения к БД. При необходимости отредактируйте:
> ```bash
> nano /var/www/html/index.php
> ```
> Проверьте параметры: хост `localhost`, БД `webdb`, пользователь `web1`, пароль `P@ssw0rd`.
![Screenshot](assets/119.png)

Удалите стандартную страницу:
```bash
rm -f /var/www/html/index.html
```

Перезапустите Apache:
```bash
systemctl restart apache2
```
![Screenshot](assets/106.png)

### Проверка
```bash
apt install curl -y
curl http://localhost
```

---

## <p align="center"><b>8. Проброс портов (iptables)</b></p>

### На BR-RTR: проброс портов 8081, 2011 на BR-SRV
```bash
iptables -t nat -A PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 192.168.4.2:8081
iptables -t nat -A PREROUTING -p tcp --dport 2011 -j DNAT --to-destination 192.168.4.2:2011
iptables -A FORWARD -p tcp -d 192.168.4.2 --dport 8081 -j ACCEPT
iptables -A FORWARD -p tcp -d 192.168.4.2 --dport 2011 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```
![Screenshot](assets/107.png)

### На HQ-RTR: проброс порта 8081, 2011 на HQ-SRV
```bash
iptables -t nat -A PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 192.168.1.2:80
iptables -t nat -A PREROUTING -p tcp --dport 2011 -j DNAT --to-destination 192.168.1.2:2011
iptables -A FORWARD -p tcp -d 192.168.1.2 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -d 192.168.1.2 --dport 2011 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```
![Screenshot](assets/108.png)

---

## <p align="center"><b>9. Nginx reverse proxy на ISP</b></p>

### Установка Nginx
```bash
apt install nginx -y
```
![Screenshot](assets/109.png)

### Настройка reverse proxy для web.au-team.irpo
```bash
nano /etc/nginx/sites-available/web.au-team.irpo
```
```nginx
server {
    listen 80;
    server_name web.au-team.irpo;

    location / {
        proxy_pass http://172.16.10.2:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
![Screenshot](assets/110.png)

### Настройка reverse proxy для docker.au-team.irpo
```bash
nano /etc/nginx/sites-available/docker.au-team.irpo
```
```nginx
server {
    listen 80;
    server_name docker.au-team.irpo;

    location / {
        proxy_pass http://172.16.20.2:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
![Screenshot](assets/111.png)

### Активация конфигураций
```bash
ln -s /etc/nginx/sites-available/web.au-team.irpo /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/docker.au-team.irpo /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t
systemctl restart nginx
systemctl enable nginx
```
![Screenshot](assets/112.png)

---

## <p align="center"><b>10. Basic Auth на ISP (web.au-team.irpo)</b></p>

### Установка утилиты htpasswd
```bash
apt install apache2-utils -y
```
![Screenshot](assets/113.png)

### Создание файла паролей
```bash
htpasswd -cb /etc/nginx/.htpasswd Khariton P@ssw0rd
```
![Screenshot](assets/114.png)

### Обновление конфигурации Nginx (перезаписание web)
```bash
rm /etc/nginx/sites-available/web.au-team.irpo
nano /etc/nginx/sites-available/web.au-team.irpo
```
```nginx
server {
    listen 80;
    server_name web.au-team.irpo;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;

    location / {
        proxy_pass http://172.16.10.2:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
![Screenshot](assets/115.png)

Перезапустите Nginx:
```bash
nginx -t
systemctl restart nginx
```

---

## <p align="center"><b>11. Проверка доступа с HQ-CLI</b></p>

### Установка 💩Yandex💩Browser💩 
Через нормальный бразуер пишем в поисковой строке
```bash
https://browser.yandex.ru
#И дважды нажимаем на кнопку скачать, после того как скачали нажать по пакету yandex.deb > ПКМ > Open With > Software Install, спустя минут 5 установит.
```

1. **http://web.au-team.irpo** — должен запросить логин/пароль (Khariton / P@ssw0rd), после чего отобразить сайт
![Screenshot](assets/116.png)

2. **http://docker.au-team.irpo** — должен отобразить контейнер `site`
![Screenshot](assets/117.png)

Также можно проверить с помощью curl:
```bash
curl http://web.au-team.irpo -u Khariton:P@ssw0rd
curl http://docker.au-team.irpo
```

> [!IMPORTANT]
> Убедитесь, что DNS на HQ-CLI настроен на HQ-SRV (192.168.1.2) и все записи корректно резолвятся:
> ```bash
> nslookup web.au-team.irpo
> nslookup docker.au-team.irpo
> ```

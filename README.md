[README.md](https://github.com/user-attachments/files/28719003/README.md)
<img width="561" height="671" alt="изображение" src="https://github.com/user-attachments/assets/c2ae75fa-bb8c-4d8b-97fc-5d024a828db3" />
<br/>

# <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 1</strong></div>

<br/>

<br/>

> [!WARNING]
> ## ПРЕДНАСТРОЙКА
> <details>
> 
> ```
> systemctl stop NetworkManager
> ```
> ```
> systemctl disable NetworkManager
> ```
> ```
> systemctl mask NetworkManager
> ```
> 
> 
></br>
>
> На всех устройствах, кроме **ISP**:
>```
>nano /etc/resolv.conf
>```
>```
>nameserver 8.8.8.8 
>nameserver 77.88.8.7
>nameserver 77.88.8.3  
>nameserver 1.1.1.1
>```
></br>
>
> На **РОУТЕРАХ** sysctl -p:
>```
> echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
>```
> 
> После, приминем настройки
>
> ```
> sysctl --system
> ```
>
> ## Если OSPF в задании 7 не заработал (не уверен в легальности, использовать на свой страх и риск)
>
>  ```
> echo net.ipv4.conf.default.rp_filter = 0 > /etc/sysctl.conf
> ```
>
> ```
> echo net.ipv4.conf.all.rp_filter = 0 > /etc/sysctl.conf
> ```
> 
></br>
>
>
> **Сурс** листы везде:
> ```
>nano /etc/apt/sources.list
>```
>```
> # deb cdrom:.......
>  ↑
>  Ставим комментарий
>```
>```
>apt update
>```
>
> </details>
</br>

<table>
  <thead>
    <tr>
      <th>Маска подсети</th>
      <th>Полная маска</th>
      <th>Сколько адресов вмещает</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/26</td>
      <td>255.255.255.192</td>
      <td align="center">62</td>
    </tr>
    <tr>
      <td>/27</td>
      <td>255.255.255.224</td>
      <td align="center">30</td>
    </tr>
    <tr>
      <td>/28</td>
      <td>255.255.255.240</td>
      <td align="center">14</td>
    </tr>
    <tr>
      <td>/29</td>
      <td>255.255.255.248</td>
      <td align="center">6</td>
    </tr>
  </tbody>
</table>

</br>

## ✔️ Задание 1 <code>[ Адрессация + Имя устройства]</code>

> [!WARNING]
> В инструкции по сетевой настройке используется редактирование: <strong>`etc/network/interfaces`</strong>

> [!NOTE]
> **Используй редактор файлов: `nano`**

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Произведите `базовую` настройку устройств

- Настройте имена устройств согласно топологии. Используйте полное доменное имя
- На всех устройствах необходимо сконфигурировать IPv4
- IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918
- Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 32 адресов
- Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не более 16 адресов
- Локальная сеть для управления(VLAN999) должна вмещать не более 8 адресов
- Локальная сеть в сторону BR-SRV должна вмещать не более 16 адресов-
 Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 2

</details>

<br/>

<details>
<summary><strong>Настройка имени</strong></summary>
  
<br/>

  - Для **Linux** используется команда: <strong>`hostnamectl set-hostname (имя устройства.au-team.irpo)`</strong>
  
  - Обновить имя можно введя команду: **`newgrp`**
### ISP
```
hostnamectl set-hostname isp.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-RTR
```
hostnamectl set-hostname hq-rtr.au-team.irpo
```
```
newgrp
```
<br/>

### BR-RTR
```
hostnamectl set-hostname br-rtr.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-SRV
```
hostnamectl set-hostname hq-srv.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-CLI
```
hostnamectl set-hostname hq-cli.au-team.irpo
```
```
newgrp
```
<br/>

### BR-SRV
```
hostnamectl set-hostname br-srv.au-team.irpo
```
```
newgrp
```
<br/>

</details>

<details>
<summary><strong>Настройка адрессации в файле <code>etc/network/interfaces</code></strong></summary>
<br/>

### Настройка адресации (кроме HQ-CLI(он позже))
##
### ISP:

```
 nano /etc/network/interfaces
```


```
auto ens224
iface ens224 inet static
address 172.16.10.1/28
auto ens256
iface ens256 inet static
address 172.16.20.1/28
```

### HQ-RTR: (в 4 и 6 задании продолжение)

```
 nano /etc/network/interfaces
```

```
auto ens192  
iface ens192 inet static  
address 172.16.10.2/28
gateway 172.16.10.1

auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.10.2
endpoint 172.16.20.2
ttl 64
  
auto ens224  
iface ens224 inet static  
address 192.168.111.1/27 
  
auto ens224:1  
iface ens224:1 inet static  
address 192.168.211.1/28

auto ens224:2  
iface ens224:2 inet static  
address 192.168.81.1/29

auto ens224.111  
iface ens224.111 inet manual   
Vlan-raw-device ens224  
  
auto ens224.211  
iface ens224.211 inet manual   
Vlan-raw-device ens224:1

auto ens224.811  
iface ens224.811 inet manual   
Vlan-raw-device ens224:2
```

### BR-RTR: (в 6 задании продолжение)

```
 nano /etc/network/interfaces
```

```
allow-hotplug ens192
iface ens192 inet static
address 172.16.20.2/28
gateway 172.16.20.1

auto ens224
iface ens224 inet static
address 192.168.0.1/28

auto gre1
iface gre1 inet tunnel
address 172.16.0.2
netmask 255.255.255.240
mode gre
local 172.16.20.2
endpoint 172.16.10.2
ttl 64
```

### BR-SRV:

```
 nano /etc/network/interfaces
```

```
allow-hotplug ens192
iface ens192 inet static
address 192.168.0.2/28
gateway 192.168.0.1
dns-nameservers 192.168.111.2 192.168.0.2
dns-search au-team.irpo
```

### HQ-SRV:

```
 nano /etc/network/interfaces
```

```
allow-hotplug ens192
iface ens192 inet static
address 192.168.111.15/27
gateway 192.168.111.1
```

</details>

<details>
<summary><strong>Таблицы сетей</strong></summary>

</br>

<table align="center">
  <tr>
    <td align="center"><strong>Сеть</strong></td>
    <td align="center"><strong>Адрес подсети</strong></td>
    <td align="center"><strong>Пул-адресов</strong></td>
  </tr>
  <tr>
    <td align="center">SRV-Net (VLAN 100)</td>
    <td align="center">192.168.111.0/27</td>
    <td align="center">192.168.111.1-62</td>
  </tr>
  <tr>
    <td align="center">CLI-Net (VLAN 200)</td>
    <td align="center">192.168.211.0/28</td>
    <td align="center">192.168.211.1-14</td>
  </tr>
  <tr>
    <td align="center">MGMT (VLAN 999)</td>
    <td align="center">192.168.81.0/29</td>
    <td align="center">192.168.81.1-6</td>
  </tr>
  <tr>
    <td align="center">BR-Net</td>
    <td align="center">192.168.0.0/27</td>
    <td align="center">192.168.0.1-30</td>
  </tr>
  <tr>
    <td align="center">ISP-HQ</td>
    <td align="center">172.16.10.0/28</td>
    <td align="center">172.16.10.1 - 14</td>
  </tr>
  <tr>
    <td align="center">ISP-BR</td>
    <td align="center">172.16.20.0/28</td>
    <td align="center">172.16.20.1 - 14</td>
  </tr>
</table>
<p align="center"><strong>Таблица подсетей</strong></p>

#

<table align="center">
  <tr>
    <td align="center"><strong>Устройство</strong></td>
    <td align="center"><strong>Интерфейс</strong></td>
    <td align="center"><strong>IPv4/IPv6</strong></td>
    <td align="center"><strong>Маска/Префикс</strong></td>
    <td align="center"><strong>Шлюз</strong></td>
    <td align="center"><strong>Сеть</strong></td>
  </tr>
  <tr>
    <td align="center" rowspan="3">ISP</td>
    <td align="center">ens192</td>
    <td align="center">(DHCP)</td>
    <td align="center">(DHCP)</td>
    <td align="center">(DHCP)</td>
    <td align="center">INTERNET</td>
  </tr>
  <tr>
    <td align="center">ens224</td>
    <td align="center">172.16.10.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">ISP-HQ-RTR</td>
  </tr>
  <tr>
    <td align="center">ens256</td>
    <td align="center">172.16.20.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">ISP-BR-RTR</td>
  </tr>
  <tr>
    <td align="center" rowspan="3">HQ-RTR</td>
    <td align="center">ens192</td>
    <td align="center">172.16.10.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.10.1</td>
    <td align="center">ISP-HQ-RTR</td>
  </tr>
  <tr>
    <td align="center">ens224.211</td>
    <td align="center">192.168.211.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">HQ-RTR-CLI</td>
  </tr>
  <tr>
    <td align="center">ens224.111</td>
    <td align="center">192.168.111.1</td>
    <td align="center">/26</td>
    <td align="center"></td>
    <td align="center">HQ-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center" rowspan="2">BR-RTR</td>
    <td align="center">ens192</td>
    <td align="center">172.16.20.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.20.1</td>
    <td align="center">ISP-BR-RTR</td>
  </tr>
  <tr>
    <td align="center">ens224</td>
    <td align="center">192.168.0.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">BR-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">HQ-SRV</td>
    <td align="center">ens192</td>
    <td align="center">192.168.111.15</td>
    <td align="center">/26</td>
    <td align="center">192.168.111.1</td>
    <td align="center">HQ-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">BR-SRV</td>
    <td align="center">ens192</td>
    <td align="center">192.168.0.2</td>
    <td align="center">/27</td>
    <td align="center">192.168.0.1</td>
    <td align="center">BR-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">HQ-CLI</td>
    <td align="center">ens192</td>
    <td align="center">192.168.211.##(DHCP)</td>
    <td align="center">/28</td>
    <td align="center">192.168.211.1</td>
    <td align="center">HQ-RTR-CLI</td>
  </tr>
</table>
<p align="center"><strong>Таблица адресации</strong></p>

</details>
</br>

## ✔️ Задание 2 <code>[NAT на ISP]</code>

<br/>

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка `ISP`

- Настройте адресацию на интерфейсах:

  - Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP **[Выполнено в задании 1]**

  - Настройте маршруты по умолчанию там, где это необходимо 

  - Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.10.0/28 **[Выполнено в задании 1]**

  - Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.20.0/28 **[Выполнено в задании 1]**

  - На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR для доступа к сети Интернет

</details>

<br/>

<details>
<summary><strong>Настройка динамической трансляции через iptables <code>NAT</code></strong></summary>

### Настройка динамической сетевой трансляции на _`ISP`_

```
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
```
```
apt-get install iptables iptables-persistent –y
```
```
iptables –t nat –A POSTROUTING –s 172.16.10.0/28 –o ens192 –j MASQUERADE  
iptables –t nat –A POSTROUTING –s 172.16.20.0/28 –o ens192 –j MASQUERADE
netfilter-persistent save
systemctl restart netfilter-persistent  
```
</br>

<details>
<summary><strong><code>Либо другая настройка</code></strong></summary>  

```  
apt install iptables  
apt install iptables iptables-persistent  
iptables –t nat –A POSTROUTING –s 172.16.10.0/28 –o ens192 –j MASQUERADE   
iptables –t nat –A POSTROUTING –s 172.16.20.0/28 –o ens192 –j MASQUERADE   
iptables-save > /etc/iptables/rules.v4  
```
</br>

###Если не работает, презагрузите машину, на которую пытаетесь раздать интернет, либо, возвращайтесь к другому варианту.

> **`ens192`** - интерфейс с которого приходит **интернет**
> 
> Для проверки можно использовать команду: **`iptables –L –t nat`** - должны высветится в Chain POSTROUTING две настроенные подсети  

> Для того, чтобы сбросить настройку *nat*, можно использовать команду **`iptables -t nat -F`**

#

</details>

</details>

<details>
<summary><strong><code>Настройка через nftables (быстрее и удобнее) </code></strong></summary> 

# На ISP

  ```
nano /etc/nftables.conf
   ```
   ```
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname "ens192" masquerade
    }
}
 ```

# На BR-RTR

 ```
nano /etc/nftables.conf
   ```

```

table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.0.2:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.0.2:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}


table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}
```

# На HQ-RTR

 ```
nano /etc/nftables.conf
   ```

```

table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.111.15:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.111.15:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}

table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}
```

Если не работает:

```
systemctl restart nftables
```

```
systemctl restart networking
```

```
sysctl -p
```

Потом на соседнем устройстве

```
systemctl restart networking
```



</details>

<br/>

## ✔️ Задание 3 `[Создание учетных записей]` 

> [!WARNING]
> Обратите внимание, что группа <code><strong>WHEEL</strong></code> не используется!
<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Создание локальных учетных записей

- Создайте пользователя sshuser на серверах HQ-SRV и BR-SRV

  - Пароль пользователя sshuser с паролем P@ssw0rd

  - Идентификатор пользователя 2011

  - Пользователь sshuser должен иметь возможность запускать sudo без ввода пароля.

- Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BR-RTR

  - Пароль пользователя net_admin с паролем P@ssw0rd

  - При настройке на OC на базе Linux пользователь net_admin должен обладать максимальными привилегиями

  - При настройке ОС на базе Linux, запускать sudo без дополнительной аутентификации
 
</details>

<br/>

<details>
<summary><strong>Как создать <code>sshuser</code></strong></summary>
<br/>

- ## Создание учёток _`sshuser`_ производится на _`HQ-SRV`_ и _`BR-SRV`_

**1.** Создаём sshuser следующими командами:
```
useradd sshuser -u 2011
```
```
passwd sshuser
```
```
P@ssw0rd
```
<br/>

**2.** После чего даем пользователю <strong><code>root</strong></code> права:
```yml
usermod -aG sudo sshuser
```

<br/>

**3.** Добавляем следующую строку в файле `/sudoers`:
```
nano /etc/sudoers
```

```yml
sshuser ALL=(ALL) NOPASSWD:ALL
```
> Позволяет запускать **`sudo`** без аутентификации

<br/>

**4.** Создаем и задаем необходимые права на домашнюю папку
```
mkdir /home/sshuser
```
```
chown sshuser:sshuser /home/sshuser
```
```
chmod 700 /home/sshuser
```
<br/>
</details>

<details>
<summary><strong>Как создать <code>net_admin</code></strong></summary>
<br/>

- ## Пользователь _`net_admin`_ на _`HQ-RTR`_ и _`BR-RTR`_

**1.** Создаём **`net_admin`**, следующими командами, но уже без `-u 1010` и с новым паролем:
```
useradd net_admin
```
```
passwd net_admin
```
```
P@ssw0rd
```
<br/>

**2.** После чего даем пользователю <strong><code>root</strong></code> права:
```yml
usermod -aG sudo net_admin
```
<br/>

**3.** Добавляем следующую строку в `/sudoers`:
```
nano /etc/sudoers
```
```yml
net_admin ALL=(ALL) NOPASSWD:ALL
```
> Позволяет запускать **`sudo`** без аутентификации
<br/>

**4.** Создаем и задаем необходимые права на домашнюю папку
```
mkdir /home/net_admin
```
```
chown net_admin:net_admin /home/net_admin
```
```
chmod 700 /home/net_admin
```
<br/>

</details>

<br/>

## ✔️ Задание 4 `[VLAN]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройте на интерфейсе `HQ-RTR` в сторону офиса `HQ` виртуальный коммутатор

- Сервер HQ-SRV должен находиться в ID VLAN 111
- Клиент HQ-CLI в ID VLAN 211
- Создайте подсеть управления с ID VLAN 811
- Основные сведения о настройке коммутатора и выбора реализации разделения на VLAN занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка VLAN на <code>HQ-RTR</code></strong></summary>

## Настройка VLAN на **`HQ-RTR`**

- Для начала установи пакет _**`vlan`**_:

```
apt install vlan
```

- После чего добавляем модуль _**`modprobe 8021q`**_ командой:
```
echo 8021q >> /etc/modules
```
- Далее переходим к конфигурации файла _**`/etc/network/interfaces`**_ и приводим её к виду:
```
nano /etc/network/interfaces
```

```
# The primary network interface
auto ens192  
iface ens192 inet static  
address 172.16.10.2/28
gateway 172.16.10.1

auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.10.2
endpoint 172.16.20.2
ttl 64
  
auto ens224  
iface ens224 inet static  
address 192.168.111.1/27 
  
auto ens224:1  
iface ens224:1 inet static  
address 192.168.211.1/28

auto ens224:2  
iface ens224:2 inet static  
address 192.168.81.1/29

auto ens224.111  
iface ens224.111 inet manual   
Vlan-raw-device ens224  
  
auto ens224.211  
iface ens224.211 inet manual   
Vlan-raw-device ens224:1

auto ens224.811  
iface ens224.811 inet manual   
Vlan-raw-device ens224:2
```

</details>

<br/>

## ✔️ Задание 5 `[SSH]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка безопасного удаленного доступа на серверах `HQ-SRV` и `BR-SRV`

- Для подключения используйте порт 2011
- Разрешите подключения только пользователю sshuser
- Ограничьте количество попыток входа до двух
- Настройте баннер «Authorized access only»

</details>
<br/>

<details>
<summary><strong>Настройка <code>SSH</code></strong></summary>
<br/>

### SSH настраиваем на `HQ-SRV` и `BR-SRV`

**1.** Для настройки **SSH** необходимо его установить коммандой:
```
apt-get install openssh-server -y
```

</br>

**2.** После чего необходимо добавить строчки в файл **`sshd_config`**:
```
nano /etc/ssh/sshd_config
```

```
Port 2011
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/ssh/banner
AllowUsers  sshuser
         
```
          ^ - это TAB



<br/>

**3.** После чего требуется создать файл **`/etc/ssh/banner`** и привести его в следующую форму:

```
nano /etc/ssh/banner
```


```
----------------------
Authorized access only
----------------------
```
</br>

**4.** Далее необходимо перезапустить **`SSH`** коммандой:
  
```
systemctl restart sshd
```


</details>

<br/>

## ✔️ Задание 6 `[GRE-TUNNEL]`

> [!WARNING]
> **Не используйте** устройства с именем **`tun0`, `gre0` и `sit0`**, т.к они являются зарезервированными в iproute2 («base devices») и имеют особое поведение.

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Между офисами `HQ` и `BR` необходимо сконфигурировать _`IP-туннель`_

- Сведения о туннеле занесите в отчёт  
- На выбор технологии GRE или IP in IP
</details>
<br/>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code>на <code>HQ-RTR</code></strong></summary>
<br/>

Для настройки _**`GRE`**_ туннеля на **`HQ-RTR`** переходим в файл конфигурации

```
nano /etc/network/interfaces
```

И убеждаемся в наличии этих строк:
```
auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.10.2
endpoint 172.16.20.2
ttl 64
```

Для работы туннеля необходимо добавить строчку в файл `/etc/modules`
```
echo gre_ip >> /etc/modules
```
```
systemctl restart networking
```
</details>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code> на <code>BR-RTR</code></strong></summary>
<br/>
Для настройки *GRE* туннеля на *BR-RTR* переходим в файл конфигурации
  
```
nano /etc/network/interfaces
```

И убеждаемся в наличии этих строк:
```
auto gre1
iface gre1 inet tunnel
address 172.16.0.2
netmask 255.255.255.240
mode gre
local 172.16.20.2
endpoint 172.16.10.2
ttl 64
```

Для работы туннеля необходимо добавить строчку в файл `/etc/modules`
```
echo gre_ip >> /etc/modules
```
```
systemctl restart networking
```
</details>
<br/>


## ✔️ Задание 7 `[OSPF]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте `link state` протокол на ваше усмотрение

- Разрешите выбранный протокол только на интерфейсах в ip туннеле  
- Маршрутизаторы должны делиться маршрутами только друг с другом  
- Обеспечьте защиту выбранного протокола посредством парольной защиты  
- Сведения о настройке и защите протокола занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка <code>OSPF</code></strong></summary>
<br/>


## Настройка `OSPF` производится на `HQ-RTR` и `BR-RTR`:

### HQ-RTR

**1.** Устанавливаем пакет `FRR`

```
sudo apt install -y frr
```

**2.** В конфигурационном файле `/etc/frr/daemons` необходимо активировать выбранный протокол `OSPF` для дальнейшей реализации его настройки:

```
nano /etc/frr/daemons
```

!!! Ищем следующую строку и меняем с (no) на (yes)
ospfd = yes



**3.** Далее перезаргужаем и добавляем в автозагрузку службу **`FRR`**

```
systemctl restart frr
```

```
systemctl enable --now frr
```

**4.** Переходим в интерфейс управления симуляцией **`FRR`** командой:
```
vtysh
```

**5.** Пишем команды для настройки **маршрутизации:**
 
```
conf t
router ospf
  passive-interface default
  router-id 1.1.1.1
  network 172.16.0.0/28 area 0
  network 192.168.111.0/27 area 1
  network 192.168.211.0/28 area 2
  area 0 authentication
exit

int gre1
  no ip ospf network broadcast
  no ip ospf passive
  ip ospf authentication
  ip ospf authentication-key password
exit
exit
write
```
<br>

### BR-RTR

**1-4.** Пункты такие же как и в HQ-RTR

**5.** Пишем команды для настройки **маршрутизации:**

**Меняется:** 

- `id-router: 2.2.2.2`

- `network 192.168.0.0/27 area 3`

- `network 172.16.0.0/28 area 0`

```
conf t
router ospf
  passive-interface default
  router-id 2.2.2.2
  network 192.168.0.0/27 area 3
  network 172.16.0.0/28 area 0
  area 0 authentication
exit

int gre1
  no ip ospf network broadcast
  no ip ospf passive
  ip ospf authentication
  ip ospf authentication-key password
exit
exit
write
```

### ПРОВЕРКА

Пингуем: **`BR-SRV - > HQ-SRV`** и **`BR-SRV - > HQ-CLI`**

Проверка в **FRR**:

```
vtysh
  show ip ospf neighbor
  show ip route ospf
```

</details>

<br/>

<details>
<summary><strong> Если вы неправильно настроили OSPF или возникли ошибки</strong></summary>

```
vtysh
```

Полный сброс настроек

```
conf t
no router ospf

int gre1
  no ip ospf network broadcast
  no ip ospf passive
  no ip ospf authentication
  no ip ospf authentication-key
exit
exit
write
```

  
</details>

<br/>
Настройка маршрутов, если не работает <code><strong>OSPF</strong></code>:

<br/>
<details>
<summary><strong>Настройка <code>маршрутов</code></strong></summary>
<br/>

После создания влана настраиваем маршрут для подсетей (чтобы они видели друг друга)  
На роутере *HQ-RTR* настройка выглядит так:
```
nano /etc/systemd/system/iproute.service
```

<br/>

После чего добавляем текст:
```
[Unit]
Description=iproute
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ip route add 192.168.0.0/27 via 172.16.0.2 dev gre1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

<br/>

На роутере *BR-RTR* создаем тот же файли настраивеим:
```
nano /etc/systemd/system/iproute.service
```
Для HQ-RTR:
```
[Unit]
Description=iproute
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ip route add 192.168.111.0/27 via 172.16.0.1 dev gre1
ExecStart=/sbin/ip route add 192.168.211.0/28 via 172.16.0.1 dev gre1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

<br/>

После на обоих устройствах прописываем:
```
systemctl daemon-reload
systemctl enable iproute.service
systemctl start iproute.service
```

</details>

<br/>

## ✔️ Задание 8 `[NAT на HQ-rtr и BR-rtr]`

> [!NOTE]
> Пул адрессов можно посмотреть в таблице <strong>[Задания 1](https://github.com/Flicks1383/Demo2025_debian/tree/main/Module1#задание-1)</strong> или же отталкиваться от вашей адрессации.

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка динамической трансляции адресов

- Настройте динамическую трансляцию адресов для обоих офисов.  
- Все устройства в офисах должны иметь доступ к сети Интернет

</details>
<br/>

<details>
<summary><strong>Настройка NAT на <code>BR</code> и <code>HQ</code></strong></summary>
<br/>

## > Настройка динамической трансляции адресов <

> ### Настройка на `ISP` выполнена в [Задании 2](https://github.com/Flicks1383/Demo09.02.06_2025/tree/main/module1#задание-2)

### Настройка динамической сетевой трансляции на `HQ-RTR`
```
apt-get install iptables iptables-persistent –y
```
```
iptables –t nat –A POSTROUTING –s 192.168.111.0/27 –o ens192 –j MASQUERADE
```
```
iptables –t nat –A POSTROUTING –s 192.168.211.0/28 –o ens192 –j MASQUERADE
```
```
netfilter-persistent save
```
```
systemctl restart netfilter-persistent  
```
### Настройка динамической сетевой трансляции на `BR-RTR`

```
apt-get install iptables iptables-persistent –y
```
```
iptables –t nat –A POSTROUTING –s 192.168.0.0/27 –o ens192 –j MASQUERADE
```
```
netfilter-persistent save
```
```
systemctl restart netfilter-persistent  
```
> Для того, чтобы **сбросить** настройку *nat*, можно использовать команду **`iptables -t nat -F`**

</br>

</details>

<details>
<summary><strong>Альтарнативная настройка сетевой трансляции через <code> nftables </code> (быстрее и удобнее) </strong></summary>

### ВНИМАНИЕ! Если у вас не вышло настроить <code> NAT </code> через <code> nftables </code>, возвращайтесь к первому варианту!

# На BR-RTR 

```
table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.0.2:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.0.2:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}


table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}
```

# На HQ-RTR

```
 table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.111.15:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.111.15:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}

table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}
```

</details>

</br>

## ✔️ Задание 9 `[DHCP]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка протокола динамической конфигурации хостов

- Настройте нужную подсеть  
- Для офиса HQ в качестве сервера DHCP выступает маршрутизатор HQ-RTR.  
- Клиентом является машина HQ-CLI.  
- Исключите из выдачи адрес маршрутизатора  
- Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR.  
- Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV.  
- DNS-суффикс для офисов HQ – au-team.irpo  
- Сведения о настройке протокола занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка DHCP на <code>HQ-RTR</code></strong></summary>
<br/>

## > Настройка _`DHCP`_ на _`HQ-RTR`_ для `CLI` <

<br/>

**1.** Устанавливаем сам **DHCP-сервер**:  
```
apt install isc-dhcp-server -y
```
<br/>

**2.** После чего переходим в конфигурацию файла `dhcpd.conf` и добавляем следующие строчки:
```
nano /etc/dhcp/dhcpd.conf
```

```
subnet 192.168.211.0 netmask 255.255.255.240 {
  range 192.168.211.2 192.168.211.14;
  option domain-name-servers 192.168.111.15;
  option domain-name "au-team.irpo";
  option routers 192.168.211.1;
  default-lease-time 600;
  max-lease-time 7200;
}
```
<br/>

**3.** После чего переходим в конфигурацию файла `isc-dhcp-server` и меняем ее добалвяя данный текст:

```
nano /etc/default/isc-dhcp-server
```

Порт смотрящий в сторону CLI 

```
INTERFACESv4="ens224:1" 
```

<br/>

**4.** Включаем сервиc **`DHCP`** и добавляем в автозагрузку на **`HQ-RTR`**:

```
systemctl start isc-dhcp-server
```

```
systemctl enable isc-dhcp-server
```

**5.** Далее на клиенсткой машине необходимо в настройках сет.адаптера выбрать режим **DHCP** и проверить работоспособность
<br/>

</details>

</br>

## ✔️ Задание 10 `[DNS]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка DNS для офисов HQ и BR  
- Основной DNS-сервер реализован на HQ-SRV.  
- Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с таблицей 3  
- В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер  

</details>
<br/>

<table align="center">
  <tr>
    <td align="center">Устройство</td>
    <td align="center">Запись</td>
    <td align="center">Тип</td>
  </tr>
  <tr>
    <td align="center">HQ-RTR</td>
    <td align="center">hq-rtr.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">BR-RTR</td>
    <td align="center">br-rtr.au-team.irpo</td>
    <td align="center">A</td>
  </tr>
  <tr>
    <td align="center">HQ-SRV</td>
    <td align="center">hq-srv.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">HQ-CLI</td>
    <td align="center">hq-cli.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">BR-SRV</td>
    <td align="center">br-srv.au-team.irpo</td>
    <td align="center">A</td>
  </tr>
  <tr>
    <td align="center">HQ-RTR</td>
    <td align="center">moodle.au-team.irpo</td>
    <td align="center">CNAME</td>
  </tr>
  <tr>
    <td align="center">BR-RTR</td>
    <td align="center">wiki.au-team.irpo</td>
    <td align="center">CNAME</td>
  </tr>
</table>

<p align="center"><strong>Таблица 2</strong></p>

<br/>

<details>
<summary><strong>Настройка при помощи <code>bind9</code></strong>[Дольше]</summary>

<details>
<summary><strong>Если возникли ошибки при установке</strong>[Дольше]</summary>

Полное очищение от bind9:

```
apt-get purge bind9 bind9-utils bind9-dnsutils dnsmasq -y
```

```
apt-get autoremove --purge -y
```

```
rm -rf /etc/bind
```

Обновление имеющейся информации о пакетах на устройстве:

```
apt update
```
  
</details>

<br/>

## > Настройка BIND9 на `HQ-SRV`<

### HQ-SRV

<br/>

**1.** Для работы с **DNS** требуется установить **`bind`** и доп. пакет командой:

```
apt-get install bind9 bind9-utils -y
```
<br/>

**2.** Далее необходимо сконфигурировать файл **`named.conf.options`** таким образом:

```
nano /etc/bind/named.conf.options
```

```
options {
  directory "/var/cache/bind";
  listen-on { 127.0.0.1; 192.168.111.0/27; 192.168.211.0/28; 192.168.0.0/28; 172.16.0.0/28; };
  forwarders { 8.8.8.8;  8.8.4.4; };
  recursion yes;
  allow-query { 127.0.0.1; 192.168.111.0/27; 192.168.211.0/28; 192.168.0.0/28; 172.16.0.0/28; };
  allow-query-cache { 127.0.0.1; 192.168.111.0/27; 192.168.211.0/28; 192.168.0.0/28; 172.16.0.0/28; };
  allow-recursion { 127.0.0.1; 192.168.111.0/27; 192.168.211.0/28; 192.168.0.0/28; 172.16.0.0/28; };
  dnssec-validation auto;
};
```
<br/>

**3.** Конфигурация ключей **`rndc`**:
```
rndc-confgen > /etc/rndc.key 
```
</br>

**❗ Для проверки, можно использовать комманду:** 

```
named-checkconf
```
</br>

**4.** Далее необходимо запустить **утилиту** коммандой:

```
systemctl enable --now named
```
</br>

**5.** Далее требуется изменить конфигурацию файла на **`HQ-SRV`** **`resolv.conf`**:

```
nano /etc/resolv.conf
```

```
nameserver 192.168.111.15
search au-team.irpo
```
</br>

**6.** После чего требуется прописать в **`/etc/bind/named.conf.local`**:
```
nano /etc/bind/named.conf.local
```


```
zone "au-team.irpo" {
  type master;
  file "/etc/bind/au-team.irpo";
};

zone "111.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/111.168.192.in-addr.arpa";
};

zone "211.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/211.168.192.in-addr.arpa";
};

zone "0.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/0.168.192.in-addr.arpa";
};
```
</br>

**7.** Далее следующими командами **создаём копию** файла и присваиваем права:

```
cp /etc/bind/db.local /etc/bind/au-team.irpo
```

</br>

# Если файл не найден, то ничего страшного, просто создайте его сами через команду <code>touch</code>!!

**8.** После чего приводим **файл `au-team.irpo`** к следующему виду:
```
nano /etc/bind/au-team.irpo
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2026102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
@       IN      NS      hq-srv.au-team.irpo.
@       IN      A       192.168.111.15
hq-rtr  IN      A       192.168.111.1
br-rtr  IN      A       192.168.0.1
hq-srv  IN      A       192.168.111.15
hq-cli  IN      A       192.168.211.2
br-srv  IN      A       192.168.0.2
moodle  IN      CNAME   hq-rtr
wiki    IN      CNAME   br-rtr
_ldap._tcp.au-team.irpo. IN SRV 0 5 389 br-srv.au-team.irpo.
_kerberos._tcp.au-team.irpo.        IN      SRV     0 100 88  br-srv.au-team.irpo.
_kdc._tcp.au-team.irpo.             IN      SRV     0 100 88  br-srv.au-team.irpo.
_kpasswd._tcp.au-team.irpo.         IN      SRV     0 100 464 br-srv.au-team.irpo.
```
</br>

</br>

**9.** После чего **создаем файлы** командами:
```
cp /etc/bind/db.127 /etc/bind/111.168.192.in-addr.arpa
cp /etc/bind/db.127 /etc/bind/211.168.192.in-addr.arpa
cp /etc/bind/db.127 /etc/bind/0.168.192.in-addr.arpa
```
</br>

**10.** После изменений файл **`111.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/111.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2026102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      hq-srv.au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
15      IN      PTR     hq-srv.au-team.irpo.
```

</br>

**11.** После изменений файл **`211.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/211.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2026102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      hq-srv.au-team.irpo.
2       IN      PTR     hq-cli.au-team.irpo.
```

</br>

**12.** После изменений файл **`0.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/0.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2026102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      hq-srv.au-team.irpo.
1       IN      PTR     br-rtr.au-team.irpo.
2       IN      PTR     br-srv.au-team.irpo.
```

### ❗ ❗ ❗ Все пробелы выше ^ ставятся TAB'ом

</br>

**13.** После чего можно проверить **ошибки** командой:
```
named-checkconf -z
```
</br>

Вывод должен быть такой:




**14.** А также перезапускаем **`bind`** командой:

```
systemctl restart named bind9
```
</br>

**15.** На всех устройствах локальной сети необходимо указать в конфигурационном файле `resolv.conf`:
```
nano /etc/resolv.conf
```

```
nameserver 192.168.111.15
search au-team.irpo
```
</br>

**16.** Проверить работоспособность можно **командой**:
```
nslookup **IP-адрес/DNS-имя**
```
</br>

</details>

<br/>

<details>
<summary><strong>Настройка при помощи <code>dnsmasq</code></strong>[Быстрее]</summary>
<br/>

1. Установка пакетов (без iptables)

```
apt update && apt install nftables dnsmasq -y
```

2. Настройка nftables (разрешение UDP и TCP для DNS)

```
nft add rule inet filter input udp dport 53 accept
nft add rule inet filter input tcp dport 53 accept
nft list ruleset > /etc/nftables.conf
systemctl enable nftables
systemctl restart nftables
```

3. Конфигурация /etc/dnsmasq.conf

```
cat << 'EOF' > /etc/dnsmasq.conf
no-resolv
interface=ens192
listen-address=192.168.111.15,127.0.0.1
read-ethers

server=8.8.8.8
server=8.8.4.4

address=/hq-rtr.au-team.irpo/192.168.111.1
address=/hq-srv.au-team.irpo/192.168.111.15
address=/br-rtr.au-team.irpo/192.168.0.1
address=/br-srv.au-team.irpo/192.168.0.2
address=/hq-cli.au-team.irpo/192.168.211.3

srv-host=_ldap._tcp.au-team.irpo,br-srv.au-team.irpo,389
srv-host=_kerberos._tcp.au-team.irpo,br-srv.au-team.irpo,88
srv-host=_kdc._tcp.au-team.irpo,br-srv.au-team.irpo,88
srv-host=_kpasswd._tcp.au-team.irpo,br-srv.au-team.irpo,464

cname=moodle.au-team.irpo,hq-rtr.au-team.irpo
cname=wiki.au-team.irpo,br-rtr.au-team.irpo
EOF
```

4. Конфигурация локального сопоставления имен /etc/hosts

```
cat << 'EOF' > /etc/hosts
127.0.0.1  localhost
127.0.1.1  server.localdomain  server
192.168.111.1  hq-rtr.au-team.irpo  hq-rtr
192.168.111.15  hq-srv.au-team.irpo  hq-srv
192.168.0.1  br-rtr.au-team.irpo  br-rtr
192.168.0.2  br-srv.au-team.irpo  br-srv
192.168.211.3  hq-cli.au-team.irpo  hq-cli
EOF
```

5. Отключение конфликтующего системного DNS-резолвера (если активен)

```
systemctl disable --now systemd-resolved 2>/dev/null || true
```

6. Перезапуск и добавление dnsmasq в автозагрузку

```
systemctl enable dnsmasq
systemctl restart dnsmasq
```

7. Настройка клиентов и самого сервера (/etc/resolv.conf)

```
rm -f /etc/resolv.conf
cat << 'EOF' > /etc/resolv.conf
search au-team.irpo
nameserver 192.168.111.15
EOF
```
Проверка:

```
nslookup ip 
```
или

```
nslookup DNS name
```

Результат:
<img width="580" height="316" alt="image" src="https://github.com/user-attachments/assets/30b30163-e0c4-4b1c-8c2c-74e336cbd6cc" />


</details>
  
<br/>

## ✔️ Задание 11 `[ВРЕМЯ/ДАТА]`

### Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена

<br/>

<details>
<summary><strong>Настройка <code>часового</code> пояса</strong></summary>
<br/>

## > Настройте часовой пояс на всех устройствах <
- На Linux настраивается часовой пояс командой:
```
timedatectl set-timezone Asia/Tomsk
```  

- Если на `CLI` возникли проблемы, делаем через настройки в графическом интерфейсе и настраиваем там временную зону

</details>

# <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 2</strong></div>

<br/>


# ВНИМАНИЕ❗❗❗

# Устройство HQ-CLI вставляет копированный текст некоректно! По этому, перепроверяйте вводимые данные! От этого часто могут возникать ошибки!!!


ДОРАБОТАТЬ
### Задание 1

<details>
<summary><strong> ## Описание задания: </strong></summary>

Настройте контроллер домена Samba DC на сервере BR-SRV: 

•	Имя домена au-team.irpo 

•	Введите в созданный домен машину HQ-CLI 

•	Создайте 5 пользователей для офиса HQ: имена пользователей формата hquser№ (например hquser1, hquser2 и т.д.) 

•	Создайте группу hq, введите в группу созданных пользователей 

•	Убедитесь, 	что 	пользователи 	группы 	hq 	имеют 	право аутентифицироваться на HQ-CLI  

• Пользователи группы hq должны иметь возможность повышать привилегии для выполнения ограниченного набора команд: cat, grep, id. Запускать другие команды с повышенными привилегиями пользователи группы права не имеют. 

</details>

<details>
<summary><strong>Создание контрлера домена </strong></summary>

<br/>

Прописать в файле /etc/hosts ip, доменное имя (полностью) и имя сервера. Как это должно выглядеть:

```
nano /etc/hosts
```

```
192.168.0.2 br-srv.au-team.irpo br-srv
```

Удаляем старую папку /etc/resolv.conf и создаём её заново:

```
unlink /etc/resolv.conf
```

```
nano /etc/resolv.conf
```
После, прописываем следующие занчениея:

```
nameserver 8.8.8.8

nameserver 192.168.111.15

nameserver 1.1.1.1

search br-srv.au-team.irpo

search au-team.irpo
```

</br>

```
apt install samba samba-dsdb-modules samba-vfs-modules acl attr winbind libpam-winbind libnss-winbind krb5-config krb5-user libpam-krb5 smbclient dnsutils net-tools -y
```

Указываем следующие значения, в следующих трёх появившихся полях, по порядку:

```
AU-TEAM.IRPO
```

```
br-srv.au-team.irpo
```
```
br-srv.au-team.irpo
```

После установки пакетов нам нужно остановить и запретить для автозапуска следующие службы:

```
systemctl disable --now smbd nmbd winbind
```

Удаление/перенос класического конфигурационного фала smb.conf, иначе так же может возникнуть ошибка:

```
mv /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

Все эти функции на себя возьмет служба samba-ad-dc. Если мы не выключим перечисленные сервисы, будет конфликт и ошибка в работе. По идее, они будут заблокированы автоматически (masked0, но лишняя осторожность не повредит.

```
samba-tool domain provision 
```


👉 создаёт контроллер домена

<br/>

<details>
<summary><strong>Если не создался домен, прописываем следующее </strong></summary>

</br>

```
systemctl stop samba-ad-dc smbd nmbd winbind 2>/dev/null
```
```
apt purge samba samba-common-bin samba-ad-dc winbind -y
```
```
apt autoremove -y
```
```
rm -rf /etc/samba
```
```
rm -rf /var/lib/samba
```
```
rm -rf /var/cache/samba
```
Проверяем имя машины на соответствие нужному доменному имени

Проверяем nano /etc/hosts на наличи ip машины своему домену, если нет, прописываем слудующее:

(BR-SRV-IP либо 192.168.xxx.xxx)  br-srv.au-team.irpo br-srv

Проверяем nano /etc/resolv.conf на наличие локального ip, если нет, прописываем:

```
nameserver 192.168.0.2
```
Сохроням и возвращаемся к пункту об установке. 

</details>

</br>

Убираем оригинальный конфигурационный файл krb5.conf и заменяем его тем, что сформировала утилита samba-tool:

```
mv /etc/krb5.conf /etc/krb5.conf.backup
```

```
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

Открываем конфигурационный файл resolv.conf:

```
nano /etc/resolv.conf
```

Удаляем старую запись DNS сервера и заменяем на запись локального сервера:

```
nameserver 192.168.0.2
```
Перезапускаем сервис и включаем его для автозапуска:

```
systemctl restart samba
```

```
systemctl enable --now samba-ad-dc
```

Проверьте статус, если всё работет продолжайте дальше. (Позже посмотрю в техе на ошибки и если найду таковые постараюсь исправить и внести изменения).


<details>
<summary><strong>Проверка работы домена и его обнаружение (не обязательно если уверены в работе)</strong></summary>

Убедимся, что сервер возвращает правильный IP при запросе адреса по домену:

```
host -t A au-team.irpo
```

Должен вернуть - au-team.irpo has address 192.168.0.2 
Пользователи и группа

Проверяем dns запись для имени контроллера домена:

```
host -t A au-team.irpo
```
Должен вернуть - au-team.irpo has address 192.168.0.2

Также DNS должен правильно вернуть SRV запись для kerberos:

```
host -t SRV _kerberos._udp.au-team.irpo
```
Должен вернуть - _ldap._tcp.au-team.irpo has SRV record 0 100 389 br-rtr.au-team.irpo

Нужно проверить появление тикета в системе

```
klist
```
Должен вывести: 

<br/>

Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@AU-TEAM.IRPO

Valid starting     Expires            Service principal
12/17/25 16:14:12  12/18/25 02:14:12  krbtgt/AU-TEAM.IRPO@AU-TEAM.IRPO
        renew until 12/18/25 16:14:05

</br>

Так же попытаться аунтифицироваться через администратора

```
kinit administrator
```

Вывод - Warning: Your password will expire in 41 days on Wed Jan 28 15:08:31 2026

</details>

</details>

<details>
<summary><strong>Создание полльзователей</strong></summary>

Создание пользователей от 1 до 5:

```
for i in {1..5}; do samba-tool user create hquser$i P@ssw0rd; done
```

Создание группы hq

```
samba-tool group add hq
```

Добавление пользователей в группу hq

```
for i in {1..5}; do samba-tool group addmembers hq hquser$i; done
```

Sudo ограничения, прописывать на HQ-CLI:

<details>
<summary><strong>visudo</strong></summary>



```
apt install sudo -y
```

```
visudo
```

Добавить:

```
%hq ALL=(ALL) NOPASSWD: /usr/bin/cat, /usr/bin/grep, /usr/bin/id
```

</details>

<details>
<summary><strong>через файл политик <code>/etc/sudoers.d/hq_policy </code>/</strong></summary>

Устанавливаем sudo

```
apt install sudo -y
```

Переходим в конфиги:

```
touch /etc/sudoers.d/hq_policy
```

Переходим в файл:

```
nano /etc/sudoers.d/hq_policy
```

Прописываем правило:

```
%hq ALL=(ALL) NOPASSWD: /usr/bin/cat, /usr/bin/grep, /usr/bin/id
```

</details>

</details>

<details>
<summary><strong>Настройка HQ-CLI на домен</strong></summary>

Первым делом, необходимо ввести HQ-CLI в домен, это делается достаточно просто. Заходим в /etc/resolv.conf и прописываем следующие занчения:

```
nano /etc/resolv.conf
```

```
nameserver 192.168.0.2

search au-team.irpo
```

После устанавливаем  <code>realm </code>, он понадобится нам в большинстве случаев.

```
apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit -y
```

Проверяем нахождение домена на HQ-CLI

```
realm discover au-team.irpo
```

Добавляем устройство в домен:

```
realm join au-team.irpo -U Administrator
```


Добавляем дополнительные права пользоватлею HQ-CLI для возможности пользоваться командами <code>realm</code> без пользования <code>root</code> прав. Для этого переходим в каталог ```nano /home/locadm/.bashrc``` и прописываем следующие значения:

```
nano /home/locadm/.bashrc
```


```
export PATH=$PATH:/usr/sbin
```

Применяем изминения:

```
source ~/.bashrc
```

После этого, нужно настройить права доступа группы hq. Для этого, разрешаем группе hq вход:

```
realm permit -g hq
```

После, прописываем права доступа на hq-cli в каталоге ```nano /etc/sssd/sssd.conf``` :

```
nano /etc/sssd/sssd.conf
```

```
[sssd]
services = nss, pam
domains = au-team.irpo
config_file_version = 2

[domain/au-team.irpo]
default_shell = /bin/bash
override_shell = /bin/bash
krb5_store_password_if_offline = true
cache_credentials = false
krb5_realm = AU-TEAM.IRPO
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u@%d
ad_domain = au-team.irpo
use_fully_qualified_names = true
ldap_id_mapping = false
access_provider = simple
simple_allow_groups = hq

```

Задаём права "только для чтения" на этот файл:

```
chmod 600 /etc/sssd/sssd.conf
```

Так же, настраиваем вход по поролю для всех, даже если заходят из-под <code> root </code>. Для этого переходим в конфиги <code> PAM </code>

```
nano /etc/pam.d/su
```

После чего, коментим следующую строку:

<img width="354" height="30" alt="image" src="https://github.com/user-attachments/assets/e51f67a5-dbd8-44d4-a8ba-ac05aff0dc97" />

Рестартим:

```
systemctl restart sssd
```

Включаем вход через GUI (PAM):

```
pam-auth-update
```

Перезапускаем вход:

```
systemctl restart sssd
```

```
systemctl restart display-manager
```

После, прописывем:

```
hquser1@au-team.irpo
```

Вводим пароль в появившемся окне:

```
P@ssw0rd
```

<details>
<summary><strong>Если возникли ошибки </strong></summary>

## Если не получается зайти на пользователя из начального экрана, необходимо проверить, что это за ошибка. Для этого заходим в уже имеющегося, прописывам: 

```
su - hquser1@au-team.irpo
```
<details>
<summary><strong>Если вывод - <code> System eror </code></strong></summary>


```
systemctl stop sssd
```

```
rm -f /etc/krb5.keytab
```

```
rm -rf /var/lib/sss/db/*
```

```
realm leave au-team.irpo --remove
```

И переподключаемся заново:

```
realm join au-team.irpo -U Administrator
```

```
realm permit -g hq
```

```
systemctl restart sssd
```

После, проверяем через: 
```
su - hquser1@au-team.irpo 
```

</details>


<details>
<summary><strong>Если <code> Permission denied </code> </strong></summary>

Скорее всего, пользователь не входит в группу. Для проверки, введите команду:

```
id hquser1@au-team.irpo
```

Результат должен быть следующим:

<img width="801" height="46" alt="изображение" src="https://github.com/user-attachments/assets/433a3f68-acd8-4aec-9112-151247359f63" />

Если в показателях <code> gid </code> и <code> groups </code> вы не видете группу hq, то делаем следующее.

Заходим на сервер и прописываем следующую команду, выдавая индивидуальный gid группе:

```
samba-tool group addunixattrs hq 10065
```

После, выдём каждому пользователям собственный uid и gid:

```
 samba-tool user addunixattrs hquser1 10060 --gid-number=10065 

 samba-tool user addunixattrs hquser2 10061 --gid-number=10065

 samba-tool user addunixattrs hquser3 10062 --gid-number=10065 

 samba-tool user addunixattrs hquser4 10063 --gid-number=10065 

samba-tool user addunixattrs hquser5 10064 --gid-number=10065 
```

Проверяем группу:

```
samba-tool group show hq
```

Ищем выделленый строки и проверяем наличие пользователей

Зелённый - пользователи в группе

Красный - персональный gid группы


<img width="658" height="456" alt="image" src="https://github.com/user-attachments/assets/d47fefd7-2105-4f4a-a8fe-f7a9944bc1b9" />


Таким же образом проверяем пользователя проверяеем пользователя:

```
samba-tool user show hquser1
```

Ищем выделенные строки:

<img width="776" height="662" alt="image" src="https://github.com/user-attachments/assets/d7b44e8f-da11-4e68-828c-8c64d60170f6" />

 

 После, очищаем кэш у <code> sssd </code> на HQ-CLI

```
sss_cache -E
```

Или точечным методом:

  
 ```
systemctl stop sssd
```

```
rm -rf /var/lib/sss/db/*

```

```
rm -rf /var/lib/sss/mc/*
```

```
systemctl start sssd
```

Если ошибка по прежнему сохраняется, то возможно были допущены ошибки в заполнении конфига. На клиенте, при копировании текста, символы могут вводиться не правильно, так что следует следить за правильностью заполнения. ДЛя этого поможет данная команда:

```
sssd -i -d 6
```
Она указывает ошибки в работе SSSD. Ищите слова <code> Error </code> и ему подобные.

Так же, возможно sssd не может проситать файл из-за проблем с правами доступа. Для этого прописываем:

```
chown -R root:root /etc/sssd
```
```
chmod 700 /etc/sssd
```
```
chmod 600 /etc/sssd/sssd.conf
```

Так же, sssd может не знать что ему нужно брать. Для этого проверяем файл <code> /etc/nsswitch.conf </code>.

```
cat /etc/nsswitch.conf
```

Должен быть слудующий результат:

<img width="696" height="217" alt="image" src="https://github.com/user-attachments/assets/695b726e-e452-48f3-82c4-f0a1fba4b3d9" />
  
</details>

<details>
<summary><strong>Если <code> ошибка отсутствия realm, но при попытке установки сообщается что служба есть </code> </strong></summary>

Обновляем имеющиеся пакеты и пути к ним

```
apt update
```

Принудительно переустанавливаем и обновляем службу:

```
apt install --reinstall realmd adcli sssd sssd-tools samba-common-bin
```
  
</details>


<details>
<summary><strong>Если <code> No such file or directory </code> на <code> au-team.irpo </code> </strong></summary>

```
chown root:root /etc/sssd/sssd.conf
```

```
chmod 600 /etc/sssd/sssd.conf
```

```
systemctl restart sssd
```
  
</details>

<details>
<summary><strong>Если <code> Already running another action </code></strong></summary>

```
systemctl restart realmd
```

```
pkill -9 adcli
```

```
pkill -9 packagekitd
```

```
rm -f /var/lib/sss/db/*
```

```
realm join au-team.irpo -U Administrator --verbose
```
  
</details>

## Должно появиться следующее сообщение:


<img width="454" height="43" alt="изображение" src="https://github.com/user-attachments/assets/e3b40380-1796-435e-a009-d34fa7a4c049" />

После прописываем <code> bash </code> для входа в обычный терминал:

```
bash
```

</details>

</details>

Ошибка - нет возможности активировать sudo с пользователя hq. Исправть

## Задание 2 RAID (HQ-SRV)

<details>
<summary><strong> Описание задания </strong></summary>


• Сконфигурируйте файловое хранилище на сервере HQ-SRV: 

•	При помощи двух подключенных к серверу дополнительных дисков размером 1 Гб сконфигурируйте дисковый массив уровня 1 

•	Имя устройства – md1, при необходимости конфигурация массива размещается в файле /etc/mdadm.conf 

•	Создайте раздел, отформатируйте раздел, в качестве файловой системы используйте ext4 

•	Обеспечьте автоматическое монтирование в папку /raid 


  
</details>


<details>
<summary><strong> ## Настройка RAID 1 </strong></summary>


<br/>

Проверяем наличие дисков на машине:

```
lsblk
```

Устанавливаем необходимую утелиту:

```
apt install mdadm -y
```

Компелируем диски в рейд:

```
mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

Создаём файл для коректного сюора рейда и заночим изминения:

```
mkdir -p /etc/mdadm
```

```
mdadm --detail --scan --verbose >> /etc/mdadm.conf
```

Разграничеваем пространстов диска:

```
mkfs.ext4 /dev/md1
```

Создаём точку монтирования:

```
mkdir /raid
```

Узнаём UUID масива, для внесения изминений в /etc/fstab

```
blkid /dev/md1
```

Прверяем, создался ли масив:

<img width="680" height="54" alt="изображение" src="https://github.com/user-attachments/assets/60644be5-5e7d-4a0e-84f0-701f1fedd127" />



После, переходим в конфигурационный файл mdadm.conf и смотрим UUID масива (он будет отличаться от предыдущего)

```
nano /etc/mdadm.conf
```

Создайте скриншот UUID и сохраните на устройство, что бы была возможность вернуться к нему позже

Теперь записываем его файл /etc/fstab:

```
nano /etc/fstab
```

Вы должны привести его к такому виду:

<img width="573" height="50" alt="изображение" src="https://github.com/user-attachments/assets/b69f8e4e-7b3d-445b-af86-d96e7ee9d3c4" />

После внесения изминений, нужно перезагрузить демона:

```
systemctl daemon-reload
```

Монтируем раздел:

```
mount -a
```

В завершение, обновите <code>initramfs </code> для обновления информации о запуске масива.

```
update-initramfs -u
```

</details>

<details>
<summary><strong> ## Если в работе возникли ошибки (полное удаление RAID) </strong></summary>

Удаляем точку монтирования:

```
umount /raid
```

Останавливаем RAID:

```
mdadm --stop /dev/md1
```

Очищаем суперблоки дисков:

```
mdadm --zero-superblock /dev/sdb
```

```
mdadm --zero-superblock /dev/sdc
```

Проверяем:

```
lsblk
```

</details>

<br/>

## ЗАДНИЕ 3 NFS

<details>
<summary><strong> Описание задания </strong></summary>

Настройте сервер сетевой файловой системы (nfs) на HQ-SRV: 

•	В качестве папки общего доступа выберите /raid/nfs, доступ для чтения и записи исключительно для сети в сторону HQ-CLI 

•	На HQ-CLI настройте автомонтирование в папку /mnt/nfs 

•	Основные параметры сервера отметьте в отчёте 

  
</details>


<br/>

<details>
<summary><strong> Настройка на HQ-SRV </strong></summary>

Первым делом устанавливаем необходимую утилиту:

```
apt install nfs-kernel-server -y
```

После создаём необходимый каталог:

```
mkdir -p /raid/nfs
```
Выыдаём права на его чтение:

```
chmod 777 /raid/nfs
```

Переходим в конфиги экспорта:

```
nano /etc/exports
```

И прописываем в нём следующее значение (ip адресс устройства должен быть точным):

```
/raid/nfs  192.168.211.0/28(rw,sync,no_subtree_check)
```

Или

```
echo "/raid/nfs 192.168.211.2/28(rw,sync,no_subtree_check)" >> /etc/exports
```

## Если дальше по заданию не заработает, укажите точный IP

Применям изминения:

```
exportfs -a
```

Включаем сервис:

```
systemctl enable --now nfs-server
```

Перезапускаем:

```
systemctl restart nfs-kernel-server
```
</details>

<details>
<summary><strong> Настройка на HQ-CLI </strong></summary>

Устанавливаем утилиту:

```
apt install autofs -y
```

Проверяем соедиение:

```
showmount -e 192.168.111.15
```

Должени быть следующий результат:

<img width="406" height="73" alt="изображение" src="https://github.com/user-attachments/assets/0fde90d6-245e-4afd-825e-45421a62880f" />

Далее, создаём папку для монтирования:

```
mkdir -p /mnt/nfs
```

Переходим к автозапуску при загрузке системы:

```
nano /etc/fstab
```

Добавляем следующую строку в файл:

```
192.168.111.15:/raid/nfs  /mnt/nfs  nfs  defaults,_netdev  0  0
```

Или

```
echo "192.168.111.15:/raid/nfs  /mnt/nfs  nfs  defaults,_netdev  0  0" >> /etc/fstab
```

Перезагружаем демона

```
systemctl daemon-reload
```


Производим монтирование:

```
mount -a
```

Проверяем результат, должен быть вывод как на рисунке:

```
df -h | grep nfs
```

<img width="531" height="66" alt="изображение" src="https://github.com/user-attachments/assets/f95864c9-e5cb-4b4d-93b3-2f82a4336ad0" />

</details>

<details>
<summary><strong> Проверка работоспособности </strong></summary>

Помимо основных проверок, чт обыли сделаны выше:

```
showmount -e 192.168.111.15
```

```
df -h | grep nfs
```

Создайте тестовый файл на клиенте в директорию, которую вы смонтированили:

```
touch /mnt/nfs/testfile
```

Полсе чего проверьте его наличие на сервере:

```
ls -l /raid/nfs/testfile
```

Результат должен быть следующим:


<img width="521" height="75" alt="изображение" src="https://github.com/user-attachments/assets/1bc0540c-64a5-4dea-84d6-b67ff2a7de59" />


</details>


<br/>

### Задание 4 - NTP (ISP)

<details>
<summary><strong> Описание задания </strong></summary>

Настройте службу сетевого времени на базе сервиса chrony на маршрутизаторе ISP: 

•	Вышестоящий сервер ntp на маршрутизаторе ISP - на выбор участника 

•	Стратум сервера - 5 

•	В качестве клиентов ntp настройте: HQ-SRV, HQ-CLI, BR-RTR, BRSRV. 

</details>



<details>
<summary><strong> Настройка на ISP </strong></summary>

<br/>

Устанавливаем необходимую утилиту:

```
apt install chrony -y
```

Переходим в конфиг:

```
nano /etc/chrony/chrony.conf
```
Добавить:

```
server 1.ru.pool.ntp.org iburst
allow 0.0.0.0/0
local stratum 5
```

На месте сервера указываем свой внешний ntp

Перезапускаем утилиту:

```
systemctl restart chrony
```

Проверяем что ISP синхронизировался с внешним мирои:

```
chronyc sources -v
```

</details>

<details>
<summary><strong> Настройка на остальных машинах (HQ-SRV, HQ-CLI, BR-RTR, BR-SRV) </strong></summary>

Так же производим установку утилиты:

```
apt install chrony -y
```

Редактируем конфиг (настраиваем ip в зависимости от подсети):

```
nano /etc/chrony/chrony.conf
```

```
server 172.16.10.1 iburst
```

```
server 172.16.20.1 iburst
```

Приминяем настройки:

```
systemctl restart chrony
```

</details>

<details>
<summary><strong> Проверка работы </strong></summary>

Прописываем следующие команды:

```
chronyc sources
```

```
chronyc tracking
```

Результат должен быть следующий:

<img width="705" height="384" alt="изображение" src="https://github.com/user-attachments/assets/dabaa33d-0a04-465d-a5a1-25e5ec8e3a44" />


  
</details>

<br/>

## Задание 5 . ANSIBLE (BR-SRV)

<details>
<summary><strong> Описание задания </strong></summary>

Сконфигурируйте ansible на сервере BR-SRV: 

•	Сформируйте файл инвентаря, в инвентарь должны входить HQ-SRV, HQ-CLI, HQ-RTR и BR-RTR 

•	Рабочий каталог ansible должен располагаться в /etc/ansible 

• Все указанные машины должны без предупреждений и ошибок отвечать pong на команду ping в ansible посланную с BR-SRV. 


</details>

<br/>

<details>
<summary><strong> Преднастройка на машинах (HQ-SRV, HQ-CLI, BR-RTR, HQ-RTR) </strong></summary>

Первым делом устанавливаем SSH:

```
apt install ssh -y
```

Полсе переходим в конфигруации и прописываем два правила, а так же раскоментируем порт 22 на роутервх:

```
nano /etc/ssh/sshd_config
```

```
Port 8081

Port 2011

PermitRootLogin yes
```

# Как должн выглядеть сервер:

<img width="560" height="146" alt="изображение" src="https://github.com/user-attachments/assets/53e0cce3-5677-43bd-a0a8-7e4ea1b7eb6a" />

 
# Как должен выглядеть роутер и HQ-CLI:

<img width="639" height="287" alt="изображение" src="https://github.com/user-attachments/assets/252596c2-8ce4-4f67-a34c-fd828f955a97" />


Перезапускаем SSH:

```
systemctl restart ssh
```

## НА BR-SRV

Переходим а файл <code> /etc/hosts </code>:

```
nano /etc/hosts
```

Прописываем адресацию, для того, что бы сервер мог обнаружить устройства.

```
192.168.111.15 HQ-SRV

192.168.211.0 HQ-CLI ( if no jobs -> 192.168.211.ip_hq-cli)

172.16.10.2 HQ-RTR

172.16.20.2 BR-RTR
```

Посел так же устанавливаем SSH на BR-SRV:

```
apt install ssh -y
```

Генерируем ключ (нажимаем везщде Enter):

```
ssh-keygen -t rsa
```

Далее, производим копирование ключа на устройства, везде нужно будет ввести пароль P@ssw0rd:

```
ssh-copy-id -p 2011 sshuser@192.168.111.15
```

```
ssh-copy-id -p 2011 root@192.168.211.2
```

```
ssh-copy-id  root@172.16.10.2
```

```
ssh-copy-id  root@172.16.20.2
```

Проверьте подключенеиме.

```
ssh -p 2011 sshuser@192.168.111.15
```

```
ssh -p 2011 root@192.168.211.3
```

```
ssh root@172.16.10.2
```

```
ssh  root@172.16.20.2
```

</details>


<details>
<summary><strong> Настройка инвентаря Ansible на BR-SRV </strong></summary>

Устанавливаем Ansible:

```
apt install ansible -y
```

Создаём необходимую директорию:

```
mkdir -p /etc/ansible
```

Переходим в настройку хостов:

```
nano /etc/ansible/hosts
```

Прописываме настроку хостов:

```
HQ-SRV ansible_host=192.168.111.15 ansible_port=2011  ansible_port=22
HQ-CLI ansible_host=192.168.211.2  ansible_port=2011  ansible_port=22
HQ-RTR ansible_host=172.16.10.2    ansible_port=2011    ansible_port=22
BR-RTR ansible_host=172.16.20.2    ansible_port=2011    ansible_port=22
```

Отключаем проверку подлиности, рнонходим в конфигурации Ansible:

```
nano /etc/ansible/ansible.cfg
```

Вставляем данную конфигурацию

```
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
remote_user = "{{ 'sshuser' if inventory_hostname == 'HQ-SRV' else 'root' }}"
ansible_port = "{{ 2011 if inventory_hostname == 'HQ-SRV' else 22 }}"
```

<details>
<summary><strong> Старый вариант настройки (на всякий случай)</strong></summary>

# Данный вариант использует только root подключение! По этому, придётся с HQ-SRV удалить подключение по sshuser, не знаю на сколько это правильно будет сделать, но так это задание заработает

```
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
remote_user = root
```

</details>

Запускаем проверку:

```
ansible all -m ping
```

Результат должен быть следующим:

<img width="1294" height="463" alt="изображение" src="https://github.com/user-attachments/assets/4d45f50d-a034-4614-bbe7-219c719836dc" />


</details>

<br/>

## Задание 6. DOCKER (BR-SRV)

<details>
<summary><strong> Описание задания </strong></summary>

Разверните веб приложение в docker на сервере BR-SRV: 

•	Средствами docker должен создаваться стек контейнеров с веб приложением и базой данных 

•	Используйте образы site_latestи mariadb_latest располагающиеся в директории docker в образе Additional.iso 

•	Основной контейнер testapp должен называться testpapp 

•	Контейнер с базой данных должен называться db 

•	Импортируйте образы в docker, укажите в yaml файле параметры подключения к СУБД, имя БД - testdb, пользователь testс паролем P@ssw0rd, порт приложения 8080, при необходимости другие параметры 

•	Приложение должно быть доступно для внешних подключений через порт 8080 

  
</details>

<br/>

<details>
<summary><strong> Настройка DOCKER на BR-SRV </strong></summary>

Установка

```
apt install docker.io docker-compose -y
```

Создание директории монтирования

```
mkdir -p /mnt/iso
```

Монтирование

```
mount /dev/sr0 /mnt/iso
```

Перехол в точку монтирования

```
cd /mnt/iso/docker
```

Загрузка необходимых образов

```
docker load -i /mnt/iso/docker/site_latest.tar
```

```
docker load -i /mnt/iso/docker/mariadb_latest.tar
```

Проверка наличия образов

```
docker images
```

Создание рабочей лиректории и файла конфигурации

```
mkdir -p /opt/testapp && cd /opt/testapp
```

Перехол в файл конфигурации

```
nano docker-compose.yml
```

Конфиг

```
version: '3.8'

services:
  db:
    image: mariadb:10.11       
    container_name: db
    restart: always
    environment:
      MYSQL_DATABASE: testdb
      MYSQL_USER: test
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_ROOT_PASSWORD: root_password
    networks:
      - backend

  testapp:
    image: site:latest          
    container_name: testapp
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      DB_TYPE: maria 
      DB_HOST: db
      DB_NAME: testdb
      DB_USER: test
      DB_PASS: P@ssw0rd
    networks:
      - backend

networks:
  backend:
    driver: bridge
```

Запуск стека

```
docker-compose up -d
```
Запуск контейнера, так как он запускается раньше базы данных от чего могут возникать ошибки

```
docker-compose start testapp
```
Проверяем

```
docker-compose ps
```

Результат:

<img width="789" height="116" alt="image" src="https://github.com/user-attachments/assets/5d2ab1a5-ee51-40c9-bb10-be3e85f53b4a" />


</details>

<br/>

## Задание 7. WEB (HQ-SRV)

<details>
<summary><strong> Описание задания </strong></summary>

Разверните веб приложение на сервере HQ-SRV: 

•	Используйте веб-сервер apache 

•	В качестве системы управления базами данных используйте mariadb 

•	Файлы веб приложения и дамп базы данных находятся в директории web образа Additional.iso 

•	Выполните импорт схемы и данных из файла dump.sql в базу данных webdb 

•	Создайте пользователя webс паролем P@ssw0rd и предоставьте ему права доступа к этой базе данных 

•	Файлы index.php и директорию images скопируйте в каталог веб сервера apache 

•	В файле index.php укажите правильные учётные данные для подключения к БД 

•	Запустите веб сервер и убедитесь в работоспособности приложения 
 

  
</details>


<br/>

<details>
<summary><strong> Настройка веб-приложения на HQ-SRV </strong></summary>

Установка необходимых компонентов:

```
apt install apache2 mariadb-server php php-mysql -y
```

Проверка

```
systemctl status apache2
```

```
systemctl status mariadb
```

Вхолд в MariaDB

```
mariadb
```

```
CREATE DATABASE webdb;
CREATE USER 'web'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD('P@ssw0rd');
CREATE USER 'web'@'%' IDENTIFIED VIA mysql_native_password USING PASSWORD('P@ssw0rd');
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'%';
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Создаём точку монтирования ISO и монтируем её туда:

```
mkdir -p /mnt/iso
mount /dev/sr0 /mnt/iso 2>/dev/null || mount -o loop /path/to/Additional.iso /mnt/iso
```

Импорт dump.sql

```
mariadb webdb < /mnt/iso/web/dump.sql
```

Проверка

```
mariadb -e "USE webdb; SHOW TABLES;"
```
Результат должен быть следующим:

<img width="442" height="126" alt="image" src="https://github.com/user-attachments/assets/778296e5-836c-44a1-83e0-310afe781dda" />

Далее копируем фалы в каталог apache и присваиваем права владельца:

```
cp /mnt/iso/web/index.php /var/www/html/

cp /mnt/iso/web/logo.png /var/www/html/

chown -R www-data:www-data /var/www/html/

chmod -R 755 /var/www/html/
```

Заходим в файл и редактируем его:

```
nano /var/www/html/index.php
```

```
<?php
$servername = "localhost";
$username = "web";
$password = "P@ssw0rd";
$dbname = "webdb";
```
Далее переходим в ещё один конфигурационный файл и происываем следующие значения:

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Меняем на 0.0.0.0 bind-address в файле

<code>bind-address = 0.0.0.0</code>

```
skip-name-resolve = ON
```

Перезапуск:

```
systemctl restart apache2
systemctl restart mariadb
```

</details>


<details>
<summary><strong> Проверка работы</strong></summary>

На клиенте:

```
http://192.168.111.15/index.php
```

или

```
http://au-team.irpo/index.php
```

На сервере

```
php /var/www/html/index.php | head -n 10
```
  
</details>

<br/>

## Задание 8. NAT (ПОРТЫ)

<details>
<summary><strong> Описание задания </strong></summary>

На маршрутизаторах сконфигурируйте статическую трансляцию портов: 

•	Пробросьте порт 8081в порт приложения testapp BR-SRV на маршрутизаторе BR-RTR, для обеспечения работы приложения testapp извне 

•	Пробросьте порт 8081в порт веб приложения на HQ-SRV на маршрутизаторе HQ-RTR, для обеспечения работы веб приложения извне 

•	Пробросьте порт 2011на маршрутизаторе HQ-RTR в порт 2026сервера HQ-SRV, для подключения к серверу по протоколу ssh из внешних сетей 

•	Пробросьте порт 2011на маршрутизаторе BR-RTR в порт 2026сервера BR-SRV, для подключения к серверу по протоколу ssh из внешних сетей. 
  
</details>



<br/>

<details>
<summary><strong> Настройка через IPTABLES </strong></summary>

### На HQ-RTR:

Настройка порта 8080:

```
iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 8081 -j DNAT --to-destination 192.168.111.15:8081
```

```
iptables -t nat -A POSTROUTING -o ens224 -p tcp --dport 8081 -d 192.168.111.15 -j MASQUERADE
```

Настройка порта 2026 (SSH):

```
iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 2011 -j DNAT --to-destination 192.168.111.15:2011
```

```
iptables -t nat -A POSTROUTING -o ens224 -p tcp --dport 2011 -d 192.168.111.15 -j MASQUERADE
```

### На BR-RTR

Настройка порта 8080:


```
iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 8081 -j DNAT --to-destination 192.168.0.2:8081
```

```
iptables -t nat -A POSTROUTING -o ens224 -p tcp --dport 8081 -d 192.168.0.2 -j MASQUERADE
```

Настройка порта 2026:

```
iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 2011 -j DNAT --to-destination 192.168.0.2:2011
```

```
iptables -t nat -A POSTROUTING -o ens224 -p tcp --dport 2011 -d 192.168.10.10 -j MASQUERADE
```

Сохраняем правила:

```
apt install iptables-persistent -y
```

```
netfilter-persistent save
```

```
netfilter-persistent reload
```

Проверка портов:

# Проверка веб-приложения на HQ-SRV

```
curl http://192.168.111.15:8081
```
# Проверка Docker-приложения на BR-SRV

```
curl http://192.168.0.2:8081
```

# Проверка SSH на HQ-SRV

```
ssh -p 2011 webuser@192.168.111.15
```

# Проверка SSH на BR-SRV

```
ssh -p 2011 webuser@192.168.0.2
```
  
</details>

<details>
<summary><strong> Настройка через NFTSBLES </strong></summary>

Открываенм конфиги nftables:

```
nnao /etc/nftables.conf
```

Прописывем конфиг на BR-RTR:

```
table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.0.2:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.0.2:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}


table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}

```

Прописываем конфиг на HQ-RTR:

```
table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        iifname "ens192" tcp dport 8081 dnat to 192.168.111.15:8081

        iifname "ens192" tcp dport 2011 dnat to 192.168.111.15:2011
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        oifname "ens192" masquerade
    }
}

table ip filter {
    chain forward {
        type filter hook forward priority filter; policy accept;
    }
}
```

Делаем файлы исполняемыми:

```
chmod +x /etc/nftables.conf
```

Применяем правила:

```
nft -f /etc/nftables.conf
```

Добавляем в автозагрузку:

```
systemctl enable nftables
```

</details>


<details>
<summary><strong> Проверка </strong></summary>

Проверку можно выполнить при помощи трёх способов, желательно проверить все:

Первый, обычное подключение по ssh по портам, на выбор у вас есть порт 2011 или 8081. Они должны переадресовывать на сервер.


На BR-RTR

```
ssh -p 2011 sshuser@172.16.20.2
```

```
ssh -p 8081 sshuser@172.16.20.2
```

На HQ-RTR

```
ssh -p 2011 sshuser@172.16.10.2
```

```
ssh -p 8081 sshuser@172.16.10.2
```

Второй:

```
apt install nmap
```

```
nmap -p 8081,2011 172.16.10.2
```

```
nmap -p 8081,2011 172.16.20.2
```

Или третий:

```
nc -zv 172.16.10.2 2011
```

```
nc -zv 172.16.20.2 2011
```
  
</details>

<br/>

## Задаие 9. NGINX PROXY (ISP)

<details>
<summary><strong> Описание задания </strong></summary>

Настройте веб-сервер nginx как обратный прокси-сервер на ISP 

•	При обращении по доменному имени web.au-team.irpo у клиента должно открываться веб приложение на HQ-SRV 

•	При обращении по доменному имени docker.au-team.irpo клиента должно открываться веб приложение testapp 

  
</details>


<br/>

<details>
<summary><strong> Настройка на ISP  </strong></summary>

## Преднастройка

Переход к хостам:

```
nano /etc/hosts
```

Обозначение маршрутов:

```
192.168.111.15 HQ-SRV
192.168.0.2 BR-SRV
```

## Оснавная настройка

Установка необходимых пакетов:

```
apt update && apt install nginx -y
```
Запуск nginx:

```
systemctl enable nginx

systemctl start nginx

systemctl status nginx
```

Создание конфига прокси:

```
nano /etc/nginx/sites-available/au-team
```

Конфиг:

```
# Reverse proxy for HQ-SRV web application
server {
    listen 8081;
    server_name web.au-team.irpo;

    location / {
        auth_basic "Restricted Content";   
        auth_basic_user_file /etc/nginx/.htpasswd;

        proxy_pass http://192.168.111.15:8081;  # Resolves via /etc/hosts
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# Reverse proxy for BR-SRV Docker web application
server {
    listen 81;
    server_name docker.au-team.irpo;

    location / {
        proxy_pass http://192.168.0.2:8081;  # Resolves via /etc/hosts
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Активайция конфига:

```
 ln -s /etc/nginx/sites-available/au-team /etc/nginx/sites-enabled/
```

Проверка на ошибки

```
nginx -t
```

Презапуск:

```
systemctl restart nginx
```
</details>

<details>
<summary><strong> Настройка на HQ-CLI </strong></summary>

Настройка хостов:

```
nano /etc/hosts
```

```
172.16.10.1 web.au-team.irpo
172.16.20.1 docker.au-team.irpo
```

## Проверка

```
apt install curl
```

Проверка адресации:

```
curl http://web.au-team.irpo
```

```
curl http://docker.au-team.irpo
```
</details>


<br/>

### Задание 10 web-based аутентификация

<details>
<summary><strong> Описание задания </strong></summary>

На маршрутизаторе ISP настройте web-based аутентификацию: 

•	При обращении к сайту web.au-team.irpo клиенту должно быть предложено ввести аутентификационные данные 

•	В качестве логина для аутентификации выберите Kharitonc с паролем P@ssw0rd 

•	Выберите файл /etc/nginx/.htpasswd в качестве хранилища учётных записей 

•	При успешной аутентификации клиент должен перейти на веб сайт. 

  
</details>


<details>
<summary><strong> Настройка на ISP </strong></summary>

Установка необходимых пакетов

```
apt update && apt install apache2-utils -y
```

Создаём файл усчётных записей

```
htpasswd -bc /etc/nginx/.htpasswd Kharitonc P@ssw0rd
```

Проверка результата:

```
cat /etc/nginx/.htpasswd
```

Добавляем дополнительные строки в /etc/nginx/sites-available/au-team:

```
nano /etc/nginx/sites-available/au-team
```


>server {
>   
>   listen 81;
>   
>  server_name web.au-team.irpo;
>
>  location / {
>   
>  ```
>    auth_basic "Restricted Area";           
>    
>     auth_basic_user_file /etc/nginx/.htpasswd; 
>  ```
>  
>  proxy_pass http://<ВНЕШНИЙ_IP_HQ-RTR>:8080;
>      
>  proxy_set_header Host $host;
>       
>  proxy_set_header X-Real-IP $remote_addr;
>        
>  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>   
>  }
>
> }

 Проверяем на ошибки:

```
 nginx -t
```

Перезагружаем 

```
systemctl reload nginx
```
  
</details>

<details>
<summary><strong> Проверка с HQ-CLI </strong></summary>

```
curl -I http://web.au-team.irpo
```

Авторизация через curl

```
curl -u WEB:P@ssw0rd http://web.au-team.irpo
```

  
</details>

## Задание 11. ЯНДЕКС БРАУЗЕР (HQ-CLI)

<details>
<summary><strong> Описание задания </strong></summary>

Удобным способом установите приложение Яндекс Браузер на HQ-CLI 
  
</details>


<br/>

<details>
<summary><strong> Установка браузера </strong></summary>

Указываем прямую ссылку:

```
wget "https://browser.yandex.ru/download/?banerid=6302000000&zih=1&beta=0&os=linux&x64=1&package=deb&full=1" -O yandex-stable.deb
```

или 

```
wget "https://browser.yandex.ru/download/?banerid=6302000000&zih=1&beta=1&os=linux&x64=1&package=deb&full=1" -O yandex.deb
```

Или

```
wget "https://browser.yandex.ru/download/?banerid=6302000000&zih=1&beta=1&os=linux&x64=1&package=deb&full=1"
```

Устанавливаем пакет

```
sudo dpkg -i yandex-browser.deb
```

# Если возникли ошибки:

Прописываем в resolv.conf dns сервера:

```
nameserver 77.88.8.8
nameserver 1.1.1.1
nameserver 8.8.8.8
```

Далее пишем apt update, для обновления информации о путях

```
apt update
```


```
sudo apt-get install -f -y
```

Проверяем установку:

```
yandex-browser --version
```
  
</details>

<details>
<summary><strong> Проверка заруска </strong></summary>

Через командную строку:

```
wich yandex-browser
```

Через ярылк:

```
nano ~/.local/share/applications/yandex-browser.desktop
```

```
[Desktop Entry]
Version=1.0
Name=Yandex Browser
Comment=Browse the Web
Exec=/usr/bin/yandex-browser %U
Icon=yandex-browser
Terminal=false
Type=Application
Categories=Network;WebBrowser;

```
</details>

<br/>

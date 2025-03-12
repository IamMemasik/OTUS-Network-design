# Проектирование адресного пространства

### Цель:
- Собрать схему CLOS;
- Распределить адресное пространство;


## Сбор топогии:
<img width="526" alt="image" src="https://github.com/user-attachments/assets/82b329a0-4bc5-43fe-be58-5e635c334b7f" />



## IP план


| Component       | IPv4 Subnet           | Example                 |
|---------------------|-------------------------|-----------------------------|
| Loopbacks  | 172.16.0.0/21     | 172.16.0.1/32 (leaf-01) <br>172.16.1.1/32 (spine-01)|
| Loopbacks (VRF)  | 172.16.96.0/20   | VRF A:<br>172.16.96.1/32 (leaf-01)<br>VRF B:<br>172.16.98.1/32 (leaf-01)|
| P2P Links          | 192.168.11.0/24<br>192.168.12.0/24   | 192.168.11.0/31 leaf-01, spine-01<br>192.168.12.0/31 leaf-01, spine-02 |
|Service Nerworks | 10.0.0.0/8 | 10.1.10.0/24 - DB segment DC1 <br>10.1.20.0/24 - web-apps segment DC 1<br>10.2.10.0/24 DB segment DC2<br>10.2.20.0/24 - web-apps segment DC 1

## Пояснение к таблице IP плана


### Loopback Links  

172.16.0.0/21, где если третий октет чётный для leaf, нечётный для spine.

### Loopbacks (VRF) Links 

Для Loopback в VRF берём 172.16.96.0/20, что больше чем диапазон для глобальных loopback'ов так как VRF может быть несколько.

### Point to point Links

Для underlay p2p соединений используется 192.168.Dn+Sn.X. 

Где:  
Dn - Номер ЦОДа  
Sn - Номер spine

Таким образом, для **DC1**: 

192.168.11.0/31 - leaf-01, spine-01
192.168.12.0/31 - leaf-01, spine-02

192.168.11.2/31 - leaf-02, spine-01
192.168.12.2/31 - leaf-02, spine-02

192.168.11.4/31 - leaf-03, spine-01
192.168.12.4/31 - leaf-03, spine-02

Для **DC2**:

192.168.21.0/31 - leaf-01, spine-01
192.168.22.0/31 - leaf-01, spine-02

192.168.21.2/31 - leaf-02, spine-01
192.168.22.2/31 - leaf-02, spine-02

192.168.21.4/31 - leaf-03, spine-01
192.168.22.4/31 - leaf-03, spine-02

И т.д.

### Service Networks

Сети сервисов внутри ЦОДа берутся из 10.Dn.0.0/16, разрезая /24 для каждого сегмента, где Dn - Номер ЦОДа.


#### Шаг 2. Выполним настройку leaf

**leaf-01**
```
hostname leaf-01
interface Ethernet1
   description r:spine-01
   no switchport
   ip address 192.168.11.0/31
interface Ethernet2
   description r:spine-02
   no switchport
   ip address 192.168.12.0/31
ip routing
```

**leaf-02**
```
hostname leaf-02
interface Ethernet1
   description r:spine-01
   no switchport
   ip address 192.168.11.2/31
interface Ethernet2
   description r:spine-02
   no switchport
   ip address 192.168.12.2/31
ip routing
```

**leaf-03**
```
hostname leaf-03
interface Ethernet1
   description r:spine-01
   no switchport
   ip address 192.168.11.4/31
interface Ethernet2
   description r:spine-02
   no switchport
   ip address 192.168.12.4/31
ip routing
```
#### Шаг 3. Выполним настройку spine

**spine-01**
```
hostname spine-01
interface Ethernet1
   description r:leaf-01
   no switchport
   ip address 192.168.11.1/31
!
interface Ethernet2
   description r:leaf-02
   no switchport
   ip address 192.168.11.3/31
!
interface Ethernet3
   description r:leaf-03
   no switchport
   ip address 192.168.11.5/31
ip routing
```
**spine-02**
```
hostname spine-02
interface Ethernet1
   description r:leaf-01
   no switchport
   ip address 192.168.12.1/31
interface Ethernet2
   description r:leaf-02
   no switchport
   ip address 192.168.12.3/31
interface Ethernet3
   description r:leaf-03
   no switchport
   ip address 192.168.12.5/31
ip routing
```


### Проверка связности

Проверим p2p линки:

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

### Настройка сетевых интерфейсов
 
 #### Шаг 1. Подпишем ip адреса на схеме в соответсвии с придуманым ip планом.
  
![image](https://github.com/user-attachments/assets/5393801a-1f0e-4870-9a48-297f5183d886)


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
interface Loopback0
   ip address 172.16.0.1/32
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
interface Loopback0
   ip address 172.16.0.2/32
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
interface Loopback0
   ip address 172.16.0.3/32
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
interface Loopback0
   ip address 172.16.1.1/32
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
interface Loopback0
   ip address 172.16.1.2/32
ip routing
```


### Проверка связности

Проверим p2p линки:
**Проверка со Spine-01**
```
spine-01#ping 192.168.11.0
PING 192.168.11.0 (192.168.11.0) 72(100) bytes of data.
80 bytes from 192.168.11.0: icmp_seq=1 ttl=64 time=12.4 ms
80 bytes from 192.168.11.0: icmp_seq=2 ttl=64 time=8.26 ms
80 bytes from 192.168.11.0: icmp_seq=3 ttl=64 time=8.66 ms
80 bytes from 192.168.11.0: icmp_seq=4 ttl=64 time=7.52 ms
80 bytes from 192.168.11.0: icmp_seq=5 ttl=64 time=6.93 ms

--- 192.168.11.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 58ms
rtt min/avg/max/mdev = 6.936/8.767/12.440/1.934 ms, ipg/ewma 14.611/10.504 ms
spine-01#ping 192.168.11.2
PING 192.168.11.2 (192.168.11.2) 72(100) bytes of data.
80 bytes from 192.168.11.2: icmp_seq=1 ttl=64 time=11.1 ms
80 bytes from 192.168.11.2: icmp_seq=2 ttl=64 time=11.5 ms
80 bytes from 192.168.11.2: icmp_seq=3 ttl=64 time=7.98 ms
80 bytes from 192.168.11.2: icmp_seq=4 ttl=64 time=8.11 ms
80 bytes from 192.168.11.2: icmp_seq=5 ttl=64 time=7.38 ms

--- 192.168.11.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 7.383/9.226/11.511/1.735 ms, ipg/ewma 13.328/10.067 ms
spine-01#ping 192.168.11.4
PING 192.168.11.4 (192.168.11.4) 72(100) bytes of data.
80 bytes from 192.168.11.4: icmp_seq=1 ttl=64 time=10.5 ms
80 bytes from 192.168.11.4: icmp_seq=2 ttl=64 time=6.91 ms
80 bytes from 192.168.11.4: icmp_seq=3 ttl=64 time=7.51 ms
80 bytes from 192.168.11.4: icmp_seq=4 ttl=64 time=7.54 ms
80 bytes from 192.168.11.4: icmp_seq=5 ttl=64 time=7.37 ms

--- 192.168.11.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 6.912/7.974/10.532/1.302 ms, ipg/ewma 15.287/9.218 ms
```


**Проверка со spinre-02**

```
spine-02#ping 192.168.12.0
PING 192.168.12.0 (192.168.12.0) 72(100) bytes of data.
80 bytes from 192.168.12.0: icmp_seq=1 ttl=64 time=9.19 ms
80 bytes from 192.168.12.0: icmp_seq=2 ttl=64 time=7.25 ms
80 bytes from 192.168.12.0: icmp_seq=3 ttl=64 time=12.0 ms
80 bytes from 192.168.12.0: icmp_seq=4 ttl=64 time=15.9 ms
80 bytes from 192.168.12.0: icmp_seq=5 ttl=64 time=18.8 ms

--- 192.168.12.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 7.256/12.677/18.853/4.267 ms, pipe 2, ipg/ewma 11.946/11.262 ms
spine-02#ping 192.168.12.2
PING 192.168.12.2 (192.168.12.2) 72(100) bytes of data.
80 bytes from 192.168.12.2: icmp_seq=1 ttl=64 time=9.78 ms
80 bytes from 192.168.12.2: icmp_seq=2 ttl=64 time=9.92 ms
80 bytes from 192.168.12.2: icmp_seq=3 ttl=64 time=10.2 ms
80 bytes from 192.168.12.2: icmp_seq=4 ttl=64 time=9.05 ms
80 bytes from 192.168.12.2: icmp_seq=5 ttl=64 time=8.24 ms

--- 192.168.12.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 8.249/9.454/10.261/0.725 ms, ipg/ewma 12.481/9.569 ms
spine-02#ping 192.168.12.4
PING 192.168.12.4 (192.168.12.4) 72(100) bytes of data.
80 bytes from 192.168.12.4: icmp_seq=1 ttl=64 time=8.87 ms
80 bytes from 192.168.12.4: icmp_seq=2 ttl=64 time=7.41 ms
80 bytes from 192.168.12.4: icmp_seq=3 ttl=64 time=9.56 ms
80 bytes from 192.168.12.4: icmp_seq=4 ttl=64 time=9.09 ms
80 bytes from 192.168.12.4: icmp_seq=5 ttl=64 time=8.75 ms

--- 192.168.12.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 7.418/8.739/9.568/0.727 ms, ipg/ewma 12.485/8.826 ms
```

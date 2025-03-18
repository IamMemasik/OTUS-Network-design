# Построение Underlay сети(OSPF)

## Цель
* Настроить OSPF для Underlay сети




## Топология 

![image](https://github.com/user-attachments/assets/0a2bea3d-3fb9-4b86-9658-adc8a7d92238)



Настройка ip адресов и проверка связности на point to point link'ах выполнена в [Лабораторной работе № 1](https://github.com/IamMemasik/OTUS-Network-design/tree/main/lab-01), поэтому сразу переходим в настройке OSPF

## Настройка ospf 

Выполним настройку ospf. 

**Leaf**
```
router ospf 1
   passive-interface default
   no passive-interface Ethernet1-2
exit
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
exit
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
exit
 interface Loopback0
   ip ospf area 0.0.0.0  
```


**Spine**
```
router ospf 1
   passive-interface default
   no passive-interface Ethernet1-3
exit
interface Ethernet1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
exit
interface Ethernet2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
exit
interface Ethernet3
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
exit
 interface Loopback0
   ip ospf area 0.0.0.0 
```



## Проверка ospf.
### Проверка установления соседства

На spine-01:
```
spine-01# show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
172.16.0.1      1        default  0   FULL                   00:00:36    192.168.11.0    Ethernet1
172.16.0.2      1        default  0   FULL                   00:00:36    192.168.11.2    Ethernet2
172.16.0.3      1        default  0   FULL                   00:00:29    192.168.11.4    Ethernet3
```

На spine-02:
```
spine-02#  sh ip ospf neighbor 
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
172.16.0.1      1        default  0   FULL                   00:00:29    192.168.12.0    Ethernet1
172.16.0.2      1        default  0   FULL                   00:00:38    192.168.12.2    Ethernet2
172.16.0.3      1        default  0   FULL                   00:00:29    192.168.12.4    Ethernet3
```

### Проверка маршрутной информации


На leaf-01
```
leaf-01#sh ip route ospf


 O        172.16.0.2/32 [110/30] via 192.168.11.1, Ethernet1
                                 via 192.168.12.1, Ethernet2
 O        172.16.0.3/32 [110/30] via 192.168.11.1, Ethernet1
                                 via 192.168.12.1, Ethernet2
 O        172.16.1.1/32 [110/20] via 192.168.11.1, Ethernet1
 O        172.16.1.2/32 [110/20] via 192.168.12.1, Ethernet2
 O        192.168.11.2/31 [110/20] via 192.168.11.1, Ethernet1
 O        192.168.11.4/31 [110/20] via 192.168.11.1, Ethernet1
 O        192.168.12.2/31 [110/20] via 192.168.12.1, Ethernet2
 O        192.168.12.4/31 [110/20] via 192.168.12.1, Ethernet2
```

На leaf-02
```
leaf-02(config)#sh ip route ospf

 O        172.16.0.1/32 [110/30] via 192.168.11.3, Ethernet1
                                 via 192.168.12.3, Ethernet2
 O        172.16.0.3/32 [110/30] via 192.168.11.3, Ethernet1
                                 via 192.168.12.3, Ethernet2
 O        172.16.1.1/32 [110/20] via 192.168.11.3, Ethernet1
 O        172.16.1.2/32 [110/20] via 192.168.12.3, Ethernet2
 O        192.168.11.0/31 [110/20] via 192.168.11.3, Ethernet1
 O        192.168.11.4/31 [110/20] via 192.168.11.3, Ethernet1
 O        192.168.12.0/31 [110/20] via 192.168.12.3, Ethernet2
 O        192.168.12.4/31 [110/20] via 192.168.12.3, Ethernet2
```

На leaf-03
```
leaf-03#sh ip route ospf

 O        172.16.0.1/32 [110/30] via 192.168.11.5, Ethernet1
                                 via 192.168.12.5, Ethernet2
 O        172.16.0.2/32 [110/30] via 192.168.11.5, Ethernet1
                                 via 192.168.12.5, Ethernet2
 O        172.16.1.1/32 [110/20] via 192.168.11.5, Ethernet1
 O        172.16.1.2/32 [110/20] via 192.168.12.5, Ethernet2
 O        192.168.11.0/31 [110/20] via 192.168.11.5, Ethernet1
 O        192.168.11.2/31 [110/20] via 192.168.11.5, Ethernet1
 O        192.168.12.0/31 [110/20] via 192.168.12.5, Ethernet2
 O        192.168.12.2/31 [110/20] via 192.168.12.5, Ethernet2
```

Заметим, что у всех лифов маршруты до loopback интерфейсов доступны по через spine-01 и spine-02 и для этих аноносов работает ECMP.


На spine-01
```
spine-01#sh ip route ospf

 O        172.16.0.1/32 [110/20] via 192.168.11.0, Ethernet1
 O        172.16.0.2/32 [110/20] via 192.168.11.2, Ethernet2
 O        172.16.0.3/32 [110/20] via 192.168.11.4, Ethernet3
 O        172.16.1.2/32 [110/30] via 192.168.11.0, Ethernet1
                                 via 192.168.11.2, Ethernet2
                                 via 192.168.11.4, Ethernet3
 O        192.168.12.0/31 [110/20] via 192.168.11.0, Ethernet1
 O        192.168.12.2/31 [110/20] via 192.168.11.2, Ethernet2
 O        192.168.12.4/31 [110/20] via 192.168.11.4, Ethernet3
```

На spine-02

```
spine-02#sh ip route ospf

 O        172.16.0.1/32 [110/20] via 192.168.12.0, Ethernet1
 O        172.16.0.2/32 [110/20] via 192.168.12.2, Ethernet2
 O        172.16.0.3/32 [110/20] via 192.168.12.4, Ethernet3
 O        172.16.1.1/32 [110/30] via 192.168.12.0, Ethernet1
                                 via 192.168.12.2, Ethernet2
                                 via 192.168.12.4, Ethernet3
 O        192.168.11.0/31 [110/20] via 192.168.12.0, Ethernet1
 O        192.168.11.2/31 [110/20] via 192.168.12.2, Ethernet2
 O        192.168.11.4/31 [110/20] via 192.168.12.4, Ethernet3

```

## Проверка связности 
Проверим связность до loopback адресов с leaf-01:

**Проверка с leaf-01**

До spine-01
```
leaf-01#ping 172.16.1.1
PING 172.16.1.1 (172.16.1.1) 72(100) bytes of data.
80 bytes from 172.16.1.1: icmp_seq=1 ttl=64 time=11.6 ms
80 bytes from 172.16.1.1: icmp_seq=2 ttl=64 time=16.6 ms
80 bytes from 172.16.1.1: icmp_seq=3 ttl=64 time=16.9 ms
80 bytes from 172.16.1.1: icmp_seq=4 ttl=64 time=12.9 ms
80 bytes from 172.16.1.1: icmp_seq=5 ttl=64 time=15.8 ms

--- 172.16.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 63ms
rtt min/avg/max/mdev = 11.675/14.802/16.973/2.114 ms, pipe 2, ipg/ewma 15.922/13.250 ms
```
До spine-02

```
leaf-01#ping 172.16.1.2
PING 172.16.1.2 (172.16.1.2) 72(100) bytes of data.
80 bytes from 172.16.1.2: icmp_seq=1 ttl=64 time=9.67 ms
80 bytes from 172.16.1.2: icmp_seq=2 ttl=64 time=9.05 ms
80 bytes from 172.16.1.2: icmp_seq=3 ttl=64 time=8.70 ms
80 bytes from 172.16.1.2: icmp_seq=4 ttl=64 time=11.4 ms
80 bytes from 172.16.1.2: icmp_seq=5 ttl=64 time=12.2 ms

--- 172.16.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 8.703/10.237/12.270/1.402 ms, pipe 2, ipg/ewma 12.318/10.053 ms
```

До leaf-02
```
leaf-01#ping 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 72(100) bytes of data.
80 bytes from 172.16.0.2: icmp_seq=1 ttl=63 time=41.8 ms
80 bytes from 172.16.0.2: icmp_seq=2 ttl=63 time=36.6 ms
80 bytes from 172.16.0.2: icmp_seq=3 ttl=63 time=153 ms
80 bytes from 172.16.0.2: icmp_seq=4 ttl=63 time=123 ms

--- 172.16.0.2 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 111ms
rtt min/avg/max/mdev = 36.626/88.693/153.134/50.606 ms, pipe 3, ipg/ewma 27.804/63.695 ms
```
До leaf-03

```
leaf-01#ping 172.16.0.3
PING 172.16.0.3 (172.16.0.3) 72(100) bytes of data.
80 bytes from 172.16.0.3: icmp_seq=1 ttl=63 time=26.6 ms
80 bytes from 172.16.0.3: icmp_seq=2 ttl=63 time=32.3 ms
80 bytes from 172.16.0.3: icmp_seq=3 ttl=63 time=34.8 ms
80 bytes from 172.16.0.3: icmp_seq=4 ttl=63 time=18.9 ms
80 bytes from 172.16.0.3: icmp_seq=5 ttl=63 time=23.3 ms

--- 172.16.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 86ms
rtt min/avg/max/mdev = 18.991/27.231/34.860/5.798 ms, pipe 3, ipg/ewma 21.588/26.644 ms
```


Связность между loopback адресами обеспечена.







# Построение Underlay сети(IS-IS)

## Цель
* Настроить IS-IS для Underlay сети




## Топология 

![image](https://github.com/user-attachments/assets/5226df16-300f-4ef3-a086-12ed5e198fd1)


Настройка ip адресов и проверка связности на point to point link'ах выполнена в [Лабораторной работе № 1](https://github.com/IamMemasik/OTUS-Network-design/tree/main/lab-01), поэтому сразу переходим в настройке IS-IS

## Настройка isis

Выполним настройку net для isis 49.0001.xxxx.xxxx.xxxx.00 - где:  
* 49 - AFI  
* 0001 - Area ID  
* xxxx.xxxx.xxxx -  ip адрес loopback'а перевёднный в 6 битный формат  
* 00 - selector  

Таким образом получаем:
|Устройство | isis net |
| ------------| --------------| 
| Leaf-01 | 49.0001.1720.1600.0001.00 |
| Leaf-02 | 49.0001.1720.1600.0002.00 |
| Leaf-03 | 49.0001.1720.1600.0003.00 |
| Spine-01 |  49.0001.1720.1600.1001.00 |
| Spine-02 | 49.0001.1720.1600.1002.00 |





### Настройка на Leaf
**Leaf-01**
```
router isis LAB
   net 49.0001.1720.1600.0001.00 ### Указываем зону и SysID
   is-type level-2               ### Используем 2 уровнь соседства 
   address-family ipv4 unicast   ### Включаем поддержку ipv4 


interface range Eth1-2 
   isis enable LAB               ### Включаем isis на интерфейсе
   isis network point-to-point   ### Указываем p2p режим
   isis bfd                      ### Включаем bfd

interface Loopback0
   isis enable LAB
   isis passive                  ### Делаем интерфейс пассивным для isis
```

**Leaf-02**
```
router isis LAB
   net 49.0001.1720.1600.0002.00
   is-type level-2
   address-family ipv4 unicast


interface range Eth1-2 
   isis enable LAB
   isis network point-to-point 
   isis bfd

interface Loopback0
   isis enable LAB
   isis passive
```

**Leaf-03**
```
router isis LAB
   net 49.0001.1720.1600.0003.00
   is-type level-2
   address-family ipv4 unicast
exit
interface range Eth1-2 
   isis enable LAB
   isis network point-to-point 
   isis bfd

interface Loopback0
   isis enable LAB
   isis passive
```


### Настройка на Spine
**Spine-01** 
```
router isis LAB
   net 49.0001.1720.1600.1001.00 ### Указываем зону и SysID
   is-type level-2               ### Используем 2 уровнь соседства 
   address-family ipv4 unicast   ### Включаем поддержку ipv4 


interface range Eth1-3
   isis enable LAB               ### Включаем isis на интерфейсе
   isis network point-to-point   ### Указываем p2p режим
   isis bfd                      ### Включаем bfd

interface Loopback0
   isis enable LAB
   isis passive                  ### Делаем интерфейс пассивным для isis

```
**Spine-02**
```
router isis LAB
   net 49.0001.1720.1600.1002.00 
   is-type level-2
   address-family ipv4 unicast


interface range Eth1-3
   isis enable LAB
   isis network point-to-point
   isis bfd

interface Loopback0
   isis enable LAB
   isis passive
```



## Проверка isis

### Проверка установления соседства

При настройке isis был запущен dump на 1 порту leaf-01

Просмотрим hello сообщение от leaf-01  

<img width="1362" alt="image" src="https://github.com/user-attachments/assets/a8e54393-fdbb-4b18-abf7-0b7a00367072" />


Просмотр TLV от spine-01  

<img width="1364" alt="image" src="https://github.com/user-attachments/assets/04b1ac0e-54c3-4bd5-b2e2-c489989a2c9c" />


После настройки на всех устойствах проверим соседство.  

На spine-01:
```
spine-01(config)#sh isis neighbors 
 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id          
LAB       default  leaf-01          L2   Ethernet1          P2P               UP    24          0D                  
LAB       default  leaf-02          L2   Ethernet2          P2P               UP    23          0D                  
LAB       default  leaf-03          L2   Ethernet3          P2P               UP    22          0D  
```

На spine-02:
```
spine-02(config)#sh isis neighbors
 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id          
LAB       default  leaf-01          L2   Ethernet1          P2P               UP    28          0E                  
LAB       default  leaf-02          L2   Ethernet2          P2P               UP    22          0E                  
LAB       default  leaf-03          L2   Ethernet3          P2P               UP    21          0E    
```

### Проверка маршрутной информации


На leaf-01
```
leaf-01#sh ip route isis detail 

VRF: default

 I L2     172.16.0.2/32 [115/30] via 192.168.11.1, Ethernet1 r:spine-01
                                 via 192.168.12.1, Ethernet2 r:spine-02
 I L2     172.16.0.3/32 [115/30] via 192.168.11.1, Ethernet1 r:spine-01
                                 via 192.168.12.1, Ethernet2 r:spine-02
 I L2     172.16.1.1/32 [115/20] via 192.168.11.1, Ethernet1 r:spine-01
 I L2     172.16.1.2/32 [115/20] via 192.168.12.1, Ethernet2 r:spine-02
 I L2     192.168.11.2/31 [115/20] via 192.168.11.1, Ethernet1 r:spine-01
 I L2     192.168.11.4/31 [115/20] via 192.168.11.1, Ethernet1 r:spine-01
 I L2     192.168.12.2/31 [115/20] via 192.168.12.1, Ethernet2 r:spine-02
 I L2     192.168.12.4/31 [115/20] via 192.168.12.1, Ethernet2 r:spine-02
```

На leaf-02
```
leaf-02#sh ip route isis detail 

VRF: default

 I L2     172.16.0.1/32 [115/30] via 192.168.11.3, Ethernet1 r:spine-01
                                 via 192.168.12.3, Ethernet2 r:spine-02
 I L2     172.16.0.3/32 [115/30] via 192.168.11.3, Ethernet1 r:spine-01
                                 via 192.168.12.3, Ethernet2 r:spine-02
 I L2     172.16.1.1/32 [115/20] via 192.168.11.3, Ethernet1 r:spine-01
 I L2     172.16.1.2/32 [115/20] via 192.168.12.3, Ethernet2 r:spine-02
 I L2     192.168.11.0/31 [115/20] via 192.168.11.3, Ethernet1 r:spine-01
 I L2     192.168.11.4/31 [115/20] via 192.168.11.3, Ethernet1 r:spine-01
 I L2     192.168.12.0/31 [115/20] via 192.168.12.3, Ethernet2 r:spine-02
 I L2     192.168.12.4/31 [115/20] via 192.168.12.3, Ethernet2 r:spine-02
```

На leaf-03
```
leaf-03#sh ip route isis detail 

VRF: default

 I L2     172.16.0.1/32 [115/30] via 192.168.11.5, Ethernet1 r:spine-01
                                 via 192.168.12.5, Ethernet2 r:spine-02
 I L2     172.16.0.2/32 [115/30] via 192.168.11.5, Ethernet1 r:spine-01
                                 via 192.168.12.5, Ethernet2 r:spine-02
 I L2     172.16.1.1/32 [115/20] via 192.168.11.5, Ethernet1 r:spine-01
 I L2     172.16.1.2/32 [115/20] via 192.168.12.5, Ethernet2 r:spine-02
 I L2     192.168.11.0/31 [115/20] via 192.168.11.5, Ethernet1 r:spine-01
 I L2     192.168.11.2/31 [115/20] via 192.168.11.5, Ethernet1 r:spine-01
 I L2     192.168.12.0/31 [115/20] via 192.168.12.5, Ethernet2 r:spine-02
 I L2     192.168.12.2/31 [115/20] via 192.168.12.5, Ethernet2 r:spine-02
```

Заметим, что у всех лифов маршруты до loopback интерфейсов доступны по через spine-01 и spine-02 и для этих аноносов работает ECMP.


На spine-01
```
spine-01(config)#sh ip route isis deta

VRF: default

 I L2     172.16.0.1/32 [115/20] via 192.168.11.0, Ethernet1 r:leaf-01
 I L2     172.16.0.2/32 [115/20] via 192.168.11.2, Ethernet2 r:leaf-02
 I L2     172.16.0.3/32 [115/20] via 192.168.11.4, Ethernet3 r:leaf-03
 I L2     172.16.1.2/32 [115/30] via 192.168.11.0, Ethernet1 r:leaf-01
                                 via 192.168.11.2, Ethernet2 r:leaf-02
                                 via 192.168.11.4, Ethernet3 r:leaf-03
 I L2     192.168.12.0/31 [115/20] via 192.168.11.0, Ethernet1 r:leaf-01
 I L2     192.168.12.2/31 [115/20] via 192.168.11.2, Ethernet2 r:leaf-02
 I L2     192.168.12.4/31 [115/20] via 192.168.11.4, Ethernet3 r:leaf-03
```

На spine-02

```
spine-02#sh ip route isis detail 

VRF: default

 I L2     172.16.0.1/32 [115/20] via 192.168.12.0, Ethernet1 r:leaf-01
 I L2     172.16.0.2/32 [115/20] via 192.168.12.2, Ethernet2 r:leaf-02
 I L2     172.16.0.3/32 [115/20] via 192.168.12.4, Ethernet3 r:leaf-03
 I L2     172.16.1.1/32 [115/30] via 192.168.12.0, Ethernet1 r:leaf-01
                                 via 192.168.12.2, Ethernet2 r:leaf-02
                                 via 192.168.12.4, Ethernet3 r:leaf-03
 I L2     192.168.11.0/31 [115/20] via 192.168.12.0, Ethernet1 r:leaf-01
 I L2     192.168.11.2/31 [115/20] via 192.168.12.2, Ethernet2 r:leaf-02
 I L2     192.168.11.4/31 [115/20] via 192.168.12.4, Ethernet3 r:leaf-03
```

### Просмотр информации по IS-IS

Просмотрим информацию по LSP и isis hostname
```
leaf-01#sh isis database 

IS-IS Instance: LAB VRF: default
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    leaf-01.00-00                10   6243   874    123 L2 <>
    leaf-02.00-00                 6  23829   937    123 L2 <>
    leaf-03.00-00                 6  39626   973    123 L2 <>
    spine-01.00-00               10  49835  1169    148 L2 <>
    spine-02.00-00                6    613   432    148 L2 <>
```

```
leaf-01#sh isis hostname 

IS-IS Instance: LAB VRF: default
Level  System ID           Hostname
L2     1720.1600.0001      leaf-01
L2     1720.1600.0002      leaf-02
L2     1720.1600.0003      leaf-03
L2     1720.1600.1001      spine-01
L2     1720.1600.1002      spine-02 
```


## Проверка bfd

**На Spine-01**

``` 
spine-01#show bfd peers 
VRF name: default
-----------------
DstAddr          MyDisc   YourDisc Interface/Transport    Type          LastUp 
------------ ---------- ---------- -------------------- ------- ---------------
192.168.11.0   32939573 1313961044       Ethernet1(13)  normal  03/22/25 15:13 
192.168.11.2 1603495595  577421980       Ethernet2(14)  normal  03/22/25 15:32 
192.168.11.4 3940786857 2857338463       Ethernet3(15)  normal  03/22/25 15:32 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```

**На spine-02**

```
spine-02#sh bfd peers 
VRF name: default
-----------------
DstAddr          MyDisc   YourDisc Interface/Transport    Type          LastUp 
------------ ---------- ---------- -------------------- ------- ---------------
192.168.12.0 2469237958 2948832207       Ethernet1(13)  normal  03/22/25 15:37 
192.168.12.2 1177117276 3381603212       Ethernet2(14)  normal  03/22/25 15:37 
192.168.12.4 3504996553 1143602547       Ethernet3(15)  normal  03/22/25 15:37 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```



## Проверка связности 
Проверим связность до loopback адресов с leaf-01:

**Проверка с leaf-01**

До spine-01
```
leaf-01#ping 172.16.1.1
PING 172.16.1.1 (172.16.1.1) 72(100) bytes of data.
80 bytes from 172.16.1.1: icmp_seq=1 ttl=64 time=9.22 ms
80 bytes from 172.16.1.1: icmp_seq=2 ttl=64 time=14.3 ms
80 bytes from 172.16.1.1: icmp_seq=3 ttl=64 time=20.0 ms
80 bytes from 172.16.1.1: icmp_seq=4 ttl=64 time=17.4 ms
80 bytes from 172.16.1.1: icmp_seq=5 ttl=64 time=17.5 ms

--- 172.16.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 66ms
rtt min/avg/max/mdev = 9.222/15.726/20.042/3.718 ms, pipe 2, ipg/ewma 16.521/12.629 ms
```
До spine-02

```
leaf-01#ping 172.16.1.2
PING 172.16.1.2 (172.16.1.2) 72(100) bytes of data.
80 bytes from 172.16.1.2: icmp_seq=1 ttl=64 time=16.0 ms
80 bytes from 172.16.1.2: icmp_seq=2 ttl=64 time=10.9 ms
80 bytes from 172.16.1.2: icmp_seq=3 ttl=64 time=10.7 ms
80 bytes from 172.16.1.2: icmp_seq=4 ttl=64 time=12.2 ms
80 bytes from 172.16.1.2: icmp_seq=5 ttl=64 time=10.9 ms

--- 172.16.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 10.777/12.192/16.000/1.979 ms, ipg/ewma 16.766/14.040 ms
```

До leaf-02
```
leaf-01#ping 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 72(100) bytes of data.
80 bytes from 172.16.0.2: icmp_seq=1 ttl=63 time=33.0 ms
80 bytes from 172.16.0.2: icmp_seq=2 ttl=63 time=50.6 ms
80 bytes from 172.16.0.2: icmp_seq=3 ttl=63 time=49.1 ms
80 bytes from 172.16.0.2: icmp_seq=4 ttl=63 time=51.5 ms
80 bytes from 172.16.0.2: icmp_seq=5 ttl=63 time=24.6 ms

--- 172.16.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 103ms
rtt min/avg/max/mdev = 24.612/41.806/51.583/10.967 ms, pipe 3, ipg/ewma 25.829/37.012 ms
```
До leaf-03

```
leaf-01#ping 172.16.0.3
PING 172.16.0.3 (172.16.0.3) 72(100) bytes of data.
80 bytes from 172.16.0.3: icmp_seq=1 ttl=63 time=40.6 ms
80 bytes from 172.16.0.3: icmp_seq=2 ttl=63 time=54.3 ms
80 bytes from 172.16.0.3: icmp_seq=3 ttl=63 time=50.3 ms
80 bytes from 172.16.0.3: icmp_seq=4 ttl=63 time=44.6 ms
80 bytes from 172.16.0.3: icmp_seq=5 ttl=63 time=19.2 ms

--- 172.16.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 85ms
rtt min/avg/max/mdev = 19.270/41.854/54.362/12.226 ms, pipe 4, ipg/ewma 21.305/40.508 ms
```


Связность между loopback адресами обеспечена.







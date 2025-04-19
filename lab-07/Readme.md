# VXLAN. Multihoming.

## Цель 
Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming

## Топология 

![alt text](image.png)

Настройка ip адресов и проверка связности на point to point link'ах выполнена в [Лабораторной работе № 1](https://github.com/IamMemasik/OTUS-Network-design/tree/main/lab-01)

В качетсве underlay сети используется eBGP, разбор и настройка выполнена в [Лабораторной работе № 4](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-04/Readme.md)

Настройка L2 сервиса выполнена в [Лабораторной работе № 5](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-05/readme.md)


Настройка L3 сервиса с Symmetric irb выполнена в  [Лабораторной работе № 6](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-06/readme.md)


## Настроойка EVPN Multihoming

### Подключение и настройка нового свитча

Подключим свитч в leaf-01 и leaf-02 и настроим на нем vlan, клиентские порты и portchannel

**switch**

Настроим обычный LACP в пассивном режиме.
```
vlan 10,20
!
interface Port-Channel1
   description r:leaf-01/02 c:evpn-esi
   switchport mode trunk
!
interface Ethernet1
   channel-group 1 mode passive
!
interface Ethernet2
   channel-group 1 mode passive
!
interface Ethernet3
   switchport access vlan 10
interface Ethernet4
   switchport access vlan 20
```


### Настройка EVPN active/active Mutltihoming 

**leaf-01 и leaf-02**
```
interface Port-Channel1
   description r:switch c:esi-0010
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0011:1111:1111:1111:1111
      route-target import 00:01:00:01:00:01
   lacp system-id dead.dead.0001
!
interface Ethernet4
   channel-group 1 mode active
```

ESI состоит из 10 байтов, 1 байт - это тип
00 значит, что метка сгенирована вручную, далее любые значения 

### Проверка Multihoming

#### Проверка LACP 

![alt text](image-1.png)


#### Проверка Route type 1 и 4
После настройки и того как LACP поднялся, можно заметить, как leaf-01 и 02 отправили в фабрику машруты 1 и 4 типа

Type 1:

![alt text](image-2.png)

Type 4:

![alt text](image-3.png)

Просмотрим эти маршруты:

На leaf-01 

Заметим, что есть маршруты VNI 10010 (vlan 10) и VNI 10020 (vlan 20) с одинаковой ESI меткой, через разные leaf (это понятно по RD)

![alt text](image-4.png)

На leaf-03 

![alt text](image-5.png)

Просмотрим, кто был выбран Designated forwarder для ES.
За выбор Designated forwarder отвечает маршрут типа 4 (ES) per EVI/ESI.
DF отвечает за форвардинг BUM трафика в сторону устройства подключенного по Multihoming к фабрике.

![alt text](image-6.png)


#### Проверка Type 2
Включим клиентские устройства и просмотрим type 2 машруты:

- vlan 10 

![alt text](image-7.png)

- vlan 20

![alt text](image-8.png)


Leaf-03 тоже получил эти машруты.
 
![alt text](image-9.png)

Посмотрим как выглядит UPDATE сообщение.

От leaf-02 до leaf-01

![alt text](image-10.png)

От leaf-01 до leaf-02

![alt text](image-11.png)


#### Проверка отказоустойчивости

Запустим бесконечный пинг с client4 (vlan 20 - 10.1.20.101) на VPC в 10 vlan (10.1.10.150)


![alt text](image-12.png)

И выключим на switch сначала один порт, потом поднимем и выключим другой


**switch**
```
interface Ethernet1
shutdown
```

После выключения интерфейса ниодного пакета не потерялось 
![alt text](image-13.png)

Leaf-01 выслал withdrawn для type 1 и type 4 сообщения
![alt text](image-14.png)

Всвязи с чем, теперь хосты подключенные по Multihoming доступны только через leaf-02

![alt text](image-15.png)


Поднимем первый линк, немного подождём, чтобы LACP увидел второй линк, машруты вернулись и выключим второй.

![alt text](image-16.png)

Машруты вернулись, выключаем второй линк:

**switch**
```
interface Ethernet2
shutdown
```

![alt text](image-17.png)

Наблюдаем 0 потерь.

Просмотрим маршруты - теперь хост находится только за leaf-01.

![alt text](image-18.png)



## Итоговая конфигурация в файлах:


[Leaf-01](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-07/ESI/leaf-01.txt)

[Leaf-02](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-07/ESI/leaf-02.txt)

[Switch](https://github.com/IamMemasik/OTUS-Network-design/blob/main/lab-07/ESI/switch.txt)




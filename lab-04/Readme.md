# Построение Underlay сети(BGP)

## Цель
Настроить BGP для Underlay сети




## Топология 

![image](https://github.com/user-attachments/assets/79c9d7b4-6d4c-405b-bc3e-03d3bd8d3e56)


Настройка ip адресов и проверка связности на point to point link'ах выполнена в [Лабораторной работе № 1](https://github.com/IamMemasik/OTUS-Network-design/tree/main/lab-01), поэтому сразу переходим в настройке eBGP

## Настройка ebgp соседства

Для построения underlay на eBGP будем использовать одинаковую AS на spine - 65000, и разную на leaf 65001, 65002, 65003 для leaf-01, 02 и 03 соответственно.


### Рассмотрим установление bgp соседства

Рассмотрим детально процесс уставления BGP сессии.
**Leaf-01**
```
ip prefix-list UNDERLAY_IP        # Создаём prefix-list с underlay ip
   seq 10 permit 192.168.11.0/31
   seq 20 permit 192.168.12.0/31
   seq 30 permit 172.16.0.1/32
!
route-map UNDERLAY_EXPORT permit 10         # Создаём Route-map и прикручиваем к ней созданный ранее prefix-list
   match ip address prefix-list UNDERLAY_IP 
!
router bgp 65001                 # Запускаем bgp процесс с номером AS - 65001
   router-id 172.16.0.1          # Делаем Router-id = ip loopback
   neighbor SPINE peer group     # Создаём peer-group для spine'ов
   neighbor SPINE remote-as 65000  # Указываем AS SPINE'ов
   neighbor 192.168.11.1 peer group SPINE # Запускаем соседство со spine-01 и 02 в peer group
   neighbor 192.168.12.1 peer group SPINE 
   redistribute connected route-map UNDERLAY_EXPORT # - Применяем Route-map для того чтобы отдать underlay сети на spine
!
```

Запустим dump трафика на Ether 1 и Ether 2 и просмотрим сообщения установления bgp-сессии от leaf-01 до spine-02.

![image](https://github.com/user-attachments/assets/d006ae50-7ef4-4744-a550-90f2a6139f72)


Заметим что spine шлёт reset. 
Это происходит потому что не запущен BGP процесс на spine-02.

Запустим BGP процесс на spine-02 и снова обратим внимание на dump


```
router bgp 65000
```

![image](https://github.com/user-attachments/assets/5fb6fae7-7132-44d5-92ce-e85bd4dd5db5)



Заметим, что spine отмалчивается, это происходит из-за того что spine не ожидает соединения от leaf-01.

Настроим соседа leaf-01 для spine-02, но ради эксперимента сделаем соседа выключенным.

**spine-02**
```
router bgp 65000
   router-id 172.16.1.2
   neighbor 192.168.12.0 remote-as 65001
   neighbor 192.168.12.0 shutdown

```
И снова обратим внимание на dump и на статусы соседей.

![image](https://github.com/user-attachments/assets/b729ae3c-dd53-4c89-95c3-495aa25bca5d)



В дампе видим, что tcp-сессия смогла установиться и leaf-01 послал hello-собощение на spine-02, но из-за того что на spine-02 сосед настроен в shutdown, то он закрывает сессию, а leaf-01 находиться в состояниях ACTIVE --> OpenSent --> Active --> OpenSent.


![image](https://github.com/user-attachments/assets/ea8a13a3-76c0-4d44-af8e-fa6dfd9a01a0)



Включим соседа spine-02 и посмотрим dump

```
 no neighbor 192.168.12.0 shutdown
```
![image](https://github.com/user-attachments/assets/a8c9317b-6f92-4f52-9132-1a49df963dff)


Видим успешно установленую TCP сессию (обратим внимание на TTL = 1 - значение пот умолчанию для eBGP), а затем и BGP.
leaf-01 и spine-01 успешно согласовали соседство и leaf-01 отдал свои маршруты (NLRI) на spine-02, а spine-02, в свою очередь, ничего не отдал, так как мы не указали что отдавать.

![image](https://github.com/user-attachments/assets/7b7142f4-464a-4ef0-8402-24446d1dcc95)


Просмотрим что отдаёт и получает spine-02

![image](https://github.com/user-attachments/assets/f4a294e4-e7d1-4de9-ad44-a1ed4d9df52a)



Настроим route-map на spine-02
```
ip prefix-list UNDERLAY_IP
   seq 10 permit 192.168.12.0/31
   seq 20 permit 192.168.12.2/31
   seq 30 permit 192.168.12.4/31
   seq 40 permit 172.16.1.2/32


route-map UNDERLAY_EXPORT permit 10
   match ip address prefix-list UNDERLAY_IP

router bgp 65000
   redistribute connected route-map UNDERLAY_EXPORT
```

После настройки сразу увидим update от spine-02 с префиксами в NLRI, которые мы проаносировали.

![image](https://github.com/user-attachments/assets/e4583a05-df66-4e4d-9d1d-c9d030ca2399)


И на самом spine-02 теперь видно какие маршруты он отдаёт

![image](https://github.com/user-attachments/assets/6ba6be80-c266-4afd-91bb-98e967aeb27a)


Просмотим таблицы маршрутизации на leaf-01 и spine-02.

**leaf-01**
![image](https://github.com/user-attachments/assets/226eed0e-2413-4ea2-ad31-47f62bfec201)


**spine-02**
![image](https://github.com/user-attachments/assets/e282abc7-11df-41b9-ae1a-46b3c1b1c8cb)


Между leaf-01 и spine-02 связность в uderlay поднята.


## Настрока BFD

Включим BFD на leaf-01 для peer group SPINE.

```
router bgp 65001
   neighbor SPINE bfd 
```

![image](https://github.com/user-attachments/assets/b2bb948b-dfbc-4748-8d63-973467197f44)


Пока BFD ниразу не поднялось, BGP сессия не будет разорвана.

![image](https://github.com/user-attachments/assets/9e878f9a-3f2c-4f4a-b2b7-e524d801d98c)



Включим BFD на Spine-02
```
router bgp 65000
   neighbor 192.168.12.0 bfd 
```

В дампе заметим, как bfd перешёл в состояние UP

![image](https://github.com/user-attachments/assets/e3474c8a-d3bf-4e8f-8b96-a3e67a3124cd)



![image](https://github.com/user-attachments/assets/42eb78ec-0f5a-4403-b3f8-24830655afa4)


Проверим, работу BFD, умышлено поменяем ip адрес spine-02 на интерфейсе Ethernet 1, после чего bfd должен моментально обнаружить разрыв связи и погасить bgp соседтво.

**spine-02**
```
int eth1
ip address 192.168.112.1/31 
```

![image](https://github.com/user-attachments/assets/6ce11636-68ef-4b19-89e3-5cd039b7506a)


Видно, что после потери 3 bfd сообщений, bgp сессия была разорвана.

![image](https://github.com/user-attachments/assets/23fc9a13-98bb-4e6d-9b5a-0fe21144b2b5)


Вернём правильный ip адрес и проверим bfd и bgp соседство.

**leaf-01**
```
int eth1
ip address 192.168.12.1/31 
```


![image](https://github.com/user-attachments/assets/f7a6f517-236a-4eef-b7ee-143ba86a1e6c)



BFD настроен и отрабатывает корректно.


## Настроим eBGP и BFD на всех устройтсвах

### Итоговая конфигурация underlay на leaf

**leaf-01**
```

```
**leaf-02**
```

```

### Итоговая конфигурация underlay на spine

**Spine-01** 
```

```
**Spine-02**
```

```



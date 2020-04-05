# Агрегация соединений
## Настройка EtherChannel

![](schema.png)

Таблица адресации
|Устройство|Интерфейс|IP-адрес	   |Маска подсети|
|----------|---------|-------------|-------------|
|S1		   |VLAN 99	 |192.168.99.11|255.255.255.0|
|S2		   |VLAN 99	 |192.168.99.12|255.255.255.0|
|S3		   |VLAN 99	 |192.168.99.13|255.255.255.0|
|PC-A  	   |NIC	     |192.168.10.1 |255.255.255.0|
|PC-B	   |NIC		 |192.168.10.2 |255.255.255.0|
|PC-C	   |NIC	     |192.168.10.3 |255.255.255.0|
--------------------------------------------------

#	Цели:

### Часть 1. Настройка базовых параметров коммутатора
### Часть 2. Настройка PAgP
### Часть 3. Настройка LACP

# Решения:

### Часть 1. Настройка базовых параметров коммутатора
#### Шаг 1. Создайте сеть согласно топологии.
#### Подключите устройства, как показано в топологии, и подсоедините необходимые кабели (схема в начале)
#### Шаг 2. Выполните инициализацию и перезагрузку коммутаторов.
#### Шаг 3. Настройте базовые параметры каждого коммутатора.

<details>
 <summary>Свитч S1</summary>

``` bash
Switch#conf t
Switch(config)#no ip domain-lookup 
Switch(config)#hostname S1
S1(config)#service password-encryption
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exec-timeout 5 0
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exec-timeout 5 0
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#int range e0/0-3,e1/1-3
S1(config-if-range)#shut
S1(config)#no logging console
S1(config)#vtp mode transparent
S1(config)#vlan 99
S1(config-vlan)#name Management
S1(config-vlan)#vlan 10
S1(config-vlan)#name Staff
S1(config-vlan)#exit
S1(config)#int e1/0
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 10
S1(config-if)#exit
S1(config)#int vlan 99
S1(config-if)#ip address 192.168.99.11 255.255.255.0
S1(config-if)#exit
S1(config)#exit
S1#copy running-config startup-config
```

</details>
S2 и S3 настраиваются также, за исключением:

<details>
 <summary>S2</summary>

``` bash
Switch(config)#hostname S2
S1(config-if)#ip address 192.168.99.12 255.255.255.0
```
</details>
<details>
 <summary>S3</summary>

``` bash
Switch(config)#hostname S3
S1(config-if)#ip address 192.168.99.13 255.255.255.0
```
</details>

##### Шаг 4: Настройте компьютеры.

``` bash
PC-1: ip 192.168.10.1/24
PC-2: ip 192.168.10.2/24
PC-3: ip 192.168.10.3/24
```

### Часть 2. Настройка протокола PAgP
##### Настройте PAgP на S1 и S3

<details>
 <summary>S1</summary>

``` bash
S1#conf t
S1(config)#interface range e0/2-3
S1(config-if-range)#channel-group 1 mode desirable
Creating a port-channel interface Port-channel 1

S1(config-if-range)#no shutdown
```
</details>

<details>
 <summary>S3</summary>

``` bash
S1#conf t
S1(config)#interface range e0/2-3
S1(config-if-range)#channel-group 1 mode auto
Creating a port-channel interface Port-channel 1

S1(config-if-range)#no shutdown
```
</details>

##### Проверить конфигурации на портах

<details>
 <summary>S1</summary>

``` bash
S1#sh run interface e0/2

interface Ethernet0/2
 channel-group 1 mode desirable
end
```

``` bash
S1#sh run interface e0/3

interface Ethernet0/3
 channel-group 1 mode desirable
end
```

``` bash
S1#sh int e0/2 switchport
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)
```

``` bash
S1#sh int e0/3 switchport
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)

```
</details>

<details>
 <summary>S3</summary>

``` bash
S3#sh run interface e0/2
interface Ethernet0/2
 channel-group 1 mode auto
```

``` bash
S3#sh run interface e0/3
interface Ethernet0/3
 channel-group 1 mode auto
```

``` bash
S3#sh int e0/2 switch
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)
```

``` bash
S3#sh int e0/3 switch
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)
```
</details>

##### Убедиться, что порты объединены
<details>
 <summary>Проверка объединения портов в EtherChannel S1:</summary>

``` bash
S1#show etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Et0/2(P) Et0/3(P)
```
</details>

<details>
 <summary>Проверка объединения портов в EtherChannel S3:</summary>

``` bash
S3#show etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Et0/2(P) Et0/3(P) 
```
</details>

##### Настройте транковые порты (S1 и S3)

</details>
<details>
 <summary>S1:</summary>
 
``` bash
S1(config)# interface port-channel 1
S1(config-if)# switchport trunk encapsulation dot1q
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
```

</details>

</details>
<details>
 <summary>S3:</summary>
 
``` bash
S1(config)# interface port-channel 1
S1(config-if)# switchport trunk encapsulation dot1q
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
```

</details>

##### Убедитесь в том, что порты настроены в качестве транковых
###### Выполните команды show run interface идентификатор-интерфейса на S1 и S3. Какие команды включены в список для интерфейсов e0/2 и e0/3 на обоих коммутаторах? Сравните результаты с текущей конфигурацией для интерфейса Po1. Запишите наблюдения.
<details>
 <summary>S1</summary>

``` bash
S1#sh run interface Po1
Building configuration...

Current configuration : 125 bytes
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
end

S1#sh int Po1 switchport
Name: Po1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 99 (Management)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001

Protected: false
Appliance trust: none
```

</details>

<details>
 <summary>S3</summary>
 
``` bash
S3#sh run interface Po1
Building configuration...

Current configuration : 125 bytes
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
end

S3#sh int Po1 switchport
Name: Po1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 99 (Management)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001

Protected: false
Appliance trust: none
```

</details>

###### Выполните команды show interfaces trunk и show spanning-tree на S1 и S3. Какой транковый порт включен в список? Какая используется сеть native VLAN? Какой вывод можно сделать на основе выходных данных?

<details>
 <summary>S1</summary>
 
``` bash
S1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Po1         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1-4094

Port        Vlans allowed and active in management domain
Po1         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         1,10,99

S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 56        128.65   P2p



VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et1/0               Desg FWD 100       128.5    P2p
Po1                 Desg FWD 56        128.65   P2p



VLAN0099
  Spanning tree enabled protocol ieee
  Root ID    Priority    32867
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32867  (priority 32768 sys-id-ext 99)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 56        128.65   P2pp
```

</details>

<details>
 <summary>S3</summary>
 
``` bash
S3#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Po1         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1-4094

Port        Vlans allowed and active in management domain
Po1         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         1,10,99


S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Root FWD 56        128.65   P2p



VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     aabb.cc00.1000
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et1/0               Desg FWD 100       128.5    P2p
Po1                 Root FWD 56        128.65   P2p



VLAN0099
  Spanning tree enabled protocol ieee
  Root ID    Priority    32867
             Address     aabb.cc00.1000
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32867  (priority 32768 sys-id-ext 99)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Root FWD 56        128.65   P2p

```

</details>

###### Ответы:
Порт Po1, Native vlan 99, S1 корневой во VLAN1, во VLAN10 оба свитча руты, в виду того, что анонс данного влана по транку не настрекн.


###### Какие значения стоимости и приоритета порта для агрегированного канала отображены в выходных данных команды show spanning-tree?

``` bash
Po1              Desg FWD 9         128.28   P2p
```

``` bash
Po1              Root BKN*9         128.28   P2p
```


### Часть 3. Настройка протокола LACP
##### Настройте LACP между S1 и S2

<details>
 <summary>S1</summary>
 
``` bash
S1(config)# interface range e0/0-1
S1(config-if-range)# switchport mode trunk
S1(config-if-range)# switchport trunk native vlan 99
S1(config-if-range)# channel-group 2 mode active
```

</details>

<details>
 <summary>S2</summary>
 
``` bash
S1(config)# interface range e0/0-1
S1(config-if-range)# switchport mode trunk
S1(config-if-range)# switchport trunk native vlan 99
S1(config-if-range)# channel-group 2 mode active
```

</details>

Для агрегирования каналов используется протокол LACP
Для образования Po2 используются порты Et0/0(P) Et0/1(P)

``` bash
S2#sh etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)         LACP      Et0/0(P)    Et0/1(P)
3      Po3(SU)         LACP      Et0/2(P)    Et0/3(P)
```

``` bash
S3#sh etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Et0/2(P)    Et0/3(P)
3      Po3(SU)         LACP      Et0/0(P)    Et0/1(P)
```

##### Настройте LACP между S2 и S3

<details>
 <summary>S2</summary>
 
``` bash
S2(config)# interface range e0/2-3
S2(config-if-range)# switchport mode trunk
S2(config-if-range)# switchport trunk native vlan 99
S2(config-if-range)# channel-group 3 mode active
Creating a port-channel interface Port-channel 3
S2(config-if-range)# no shutdown
```

</details>

<details>
 <summary>S3</summary>
 
``` bash
S3(config)# interface range e0/0-1
S3(config-if-range)# switchport mode trunk
S3(config-if-range)# switchport trunk native vlan 99
S3(config-if-range)# channel-group 3 mode passive
Creating a port-channel interface Port-channel 3

S3(config-if-range)# no shutdown
```

</details>

#### проверка работы PAgP и LACP

<details>
 <summary>S1</summary>
 
``` bash
S1#sh etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Et0/2(P)    Et0/3(P)
2      Po2(SU)         LACP      Et0/0(P)    Et0/1(P)
```
</details>

<details>
 <summary>S2</summary>
 
``` bash
S2#sh etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)         LACP      Et0/0(P)    Et0/1(P)
3      Po3(SU)         LACP      Et0/2(P)    Et0/3(P)
```

</details>

<details>
 <summary>S3</summary>
 
``` bash
S2#sh etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Et0/2(P)    Et0/3(P)
3      Po3(SU)         LACP      Et0/0(P)    Et0/1(P)
```

</details>

![](ping.png)
##### Проверка наличия сквозного соединения
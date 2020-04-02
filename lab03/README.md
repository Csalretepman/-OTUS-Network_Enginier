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

#№№ Часть 1. Настройка базовых параметров коммутатора
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
S1(config)#int range f0/1-4,f0/6-24
S1(config-if-range)#shut
S1(config)#no logging console
S1(config)#vtp mode transparent
S1(config)#vlan 99
S1(config-vlan)#name Management
S1(config-vlan)#vlan 10
S1(config-vlan)#name Staff
S1(config-vlan)#exit
S1(config)#int f0/5
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

### Часть 2. Настройка протокола PAgP
##### Настройте PAgP на S1 и S3

<details>
 <summary>S1</summary>

``` bash
S1#conf t
S1(config)#interface range f0/3-4
S1(config-if-range)#channel-group 1 mode desirable
Creating a port-channel interface Port-channel 1

S1(config-if-range)#no shutdown
```
</details>

<details>
 <summary>S3</summary>

``` bash
S1#conf t
S1(config)#interface range f0/3-4
S1(config-if-range)#channel-group 1 mode auto
Creating a port-channel interface Port-channel 1

S1(config-if-range)#no shutdown
```
</details>

##### Проверить конфигурации на портах

<details>
 <summary>S1</summary>

``` bash
S1#sh run interface f0/3

interface Ethernet0/3
 channel-group 1 mode desirable
end
```

``` bash
S1#sh run interface f0/4

interface Ethernet0/4
 channel-group 1 mode desirable
end
```

``` bash
S1#sh int f0/3 switchport
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)
```

``` bash
S1#sh int f0/4 switchport
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)

```
</details>

<details>
 <summary>S3</summary>

``` bash
S3#sh run interface f0/3
interface Ethernet0/1
 channel-group 1 mode auto
```

``` bash
S3#sh run interface f0/4
interface Ethernet0/2
 channel-group 1 mode auto
```

``` bash
S3#sh int f0/3 switch
Administrative Mode: dynamic auto
Operational Mode: static access (member of bundle Po1)
```

``` bash
S3#sh int f0/4 switch
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

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P)
```
</details>
<details>
 <summary>Проверка объединения портов в EtherChannel S3:</summary>

``` bash
S3#show etherchannel summary

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P) 
```
</details>

##### Настройте транковые порты (S1 и S3)

</details>
<details>
 <summary>S1:</summary>
``` bash
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
```
</details>

</details>
<details>
 <summary>S3:</summary>
``` bash
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
```
</details>


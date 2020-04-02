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

# Часть 1. Настройка базовых параметров коммутатора
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
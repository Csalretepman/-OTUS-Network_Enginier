---
typora-root-url: ./
---

# Лабораторная работа. Настройка OSPFv2 для нескольких областей

![Топология](work_dir/top.jpg)

Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети |
| ---------- | --------- | -------- | ------------- |
| R1       | lo/0 | 209.165.200.225 | 255.255.255.252 |
|  | lo1 | 192.168.1.1 | 255.255.255.0 |
|	   	     | lo2 | 192.168.2.1 | 255.255.255.0 |
|	| s1/0 (DCE) | 192.168.12.1 | 255.255.255.252 |
| R2         | lo6 | 192.168.6.1 | 255.255.255.0 |
|  | s1/0 | 192.168.12.2 | 255.255.255.252 |
|  | s1/1 (DCE) | 192.168.23.1 | 255.255.255.252 |
| R3       | lo4 | 192.168.4.1     | 255.255.255.0   |
|  | lo5 | 192.168.5.1     | 255.255.255.0   |
|  | s1/1 | 192.168.23.2    | 255.255.255.252 |

### Часть 1:   Создание сети и настройка основных параметров устройства

Произвести базовую настройку маршрутизаторов



<details>
 <summary>R1</summary>

``` bash

Router>en
Router#conf t
Router(config)#hostname R1
R1(config)#no logging console
R1(config)#no ip domain-lookup
R1(config)#service password-encryption 
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#int lo0
R1(config-if)#ip address 209.165.200.225 255.255.255.252
R1(config-if)#description internet
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#int lo1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#int lo2
R1(config-if)#ip address 192.168.2.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#int s1/0
R1(config-if)#ip address 192.168.12.1 255.255.255.252
R1(config-if)#clock rate 128000
R1(config-if)#no shutdown
R1(config-if)#end
R1#wr
Building configuration...
[OK]
R1#

```
</details>

<details>
 <summary>R2</summary>

``` bash

Router>en
Router#conf t
Router(config)#hostname R2
R2(config)#no logging console
R2(config)#no ip domain-lookup
R2(config)#service password-encryption 
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#int lo6
R2(config-if)#ip address 192.168.6.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#int s1/0
R2(config-if)#ip address 192.168.12.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#int s1/1
R2(config-if)#ip address 192.168.23.1 255.255.255.252
R2(config-if)#clock rate 128000
R2(config-if)#no shutdown
R2(config-if)#end
R2#wr
Building configuration...
[OK]
R2#

```
</details>

<details>
 <summary>R3</summary>

``` bash

Router>en
Router#conf t
Router(config)#hostname R3
R3(config)#no logging console
R3(config)#no ip domain-lookup
R3(config)#service password-encryption 
R3(config)#enable secret class
R3(config)#line console 0
R3(config-line)#password cisco
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#line vty 0 4
R3(config-line)#password cisco
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#int lo4
R3(config-if)#ip address 192.168.4.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#int lo5
R3(config-if)#ip address 192.168.5.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#int s1/1
R3(config-if)#ip address 192.168.23.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#end
R3#wr
Building configuration...
[OK]

```
</details>

Проверить  наличие подключения на уровне 3


<details>
 <summary>R2</summary>

``` bash
![R2sh-ip-int-bri+ping-R1-R3](/work_dir/R2sh-ip-int-bri+ping-R1-R3.jpg
```

</details>


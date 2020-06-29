
# Лабораторная работа №8. PBR

![TOP](https://raw.githubusercontent.com/Csalretepman/-OTUS-Network_Enginier/master/lab08/TOP.JPG)

### 1. Настроить политику маршрутизации для сетей офиса

Для маршрутизации нужно прописать адрес шлюза в каждой рабочей станции, адресом шлюза будет являться ip интерфейса роутера от которого идет линк к свитчам в сторону клиентов, в моем случае это интерфейс e0/2 28го роутера.

<details>
 <summary>VPC30</summary>

``` bash
set pc VPC30
ip 10.30.3.30/24 10.30.3.1
ip 2001:ABCD:0030:3::30/64 2001:ABCD:0030:3::1

```
</details>

<details>
 <summary>VPC31</summary>

``` bash
set pc VPC31
ip 10.30.4.31/24 10.30.4.1
ip 2001:ABCD:0030:4::31/64 2001:ABCD:0030:4::1

```
</details>

<details>
 <summary>Ping</summary>
![ping_vpc31](https://raw.githubusercontent.com/Csalretepman/-OTUS-Network_Enginier/master/lab08/ping_vpc31.JPG)
</details>


### 2. Распределить трафик между двумя линками с провайдером

Чтобы прокинуть интернет с ISP в сторону сети надо настроить маршруты по умолчанию на R28 со стороны провайдера с одинаковыми метриками, чтобы была равнозначность для возможности балансировки трафика



<details>
 <summary>R28</summary>

``` bash
conf t
 ip route 0.0.0.0 0.0.0.0 200.30.33.55 1 name R25_Triada
 ip route 0.0.0.0 0.0.0.0 200.30.30.36 1 name R26_Triada
 ipv6 route ::/0 2001:ABCD:0030:2528::25 1 name R25_Triada
 ipv6 route ::/0 2001:ABCD:0030:2628::26 1 name R26_Triada
 exit
```
</details>
<br>

``` bash
R28#sh ip route stat

Gateway of last resort is 200.30.33.55 to network 0.0.0.0

S*    0.0.0.0/0 [30/0] via 200.30.33.55
                [30/0] via 200.30.30.36
R28#sh ipv6 route stat

S   ::/0 [30/0]
     via 2001:ABCD:30:2528::25
     via 2001:ABCD:30:2628::26

```

Для настройки балансировки трафика между каналами ISP прибегнем к маршрутизации на основе политик Policy-based routing (PBR). Будем с четного номера подсети (10.30.4.0/24) выводить трафик через четный интерфейс e0/0 роутера R28, а с нечетного (10.30.3.0/24) соответственно через  e0/1. Для реализации нам понадобится ACL типа standart


<details>
 <summary>Настройка PBR на R28</summary>

``` bash
conf t

ip access-list standard ACL_PBR_TO_R25
  permit 10.30.1.0 0.0.254.255
  deny any
  exit
ipv6 access-list ACL_PBR_TO_R25-v6
  permit 2001:ABCD:0030:3::/64 any
  deny any any
  exit
ip access-list standard ACL_PBR_TO_R26
  permit 10.30.0.0 0.0.254.255
  deny any
  exit
ipv6 access-list ACL_PBR_TO_R26-v6
  permit 2001:ABCD:0030:4::/64 any
  deny any any
  exit
  
route-map PBR_TO_R25 permit 10
  match ip address ACL_PBR_TO_R25
  set ip next-hop 200.30.33.55
  exit
route-map PBR_TO_R25-v6 permit 10
  match ip address ACL_PBR_TO_R25-v6
  set ipv6 next-hop 2001:ABCD:0030:2528::25
  exit

route-map PBR_TO_R26 permit 10
  match ip address ACL_PBR_TO_R26
  set ip next-hop 200.30.30.36
  exit
route-map PBR_TO_R26-v6 permit 10
  match ip address ACL_PBR_TO_R26-v6
  set ipv6 next-hop 2001:ABCD:0030:2628::26
  exit

int e0/1
  ip policy route-map PBR_TO_R25
  ipv6 policy route-map PBR_TO_R25-v6
exit

int e0/0
  ip policy route-map PBR_TO_R26
  ipv6 policy route-map PBR_TO_R26-v6
exit

```
</details>

### 3. Настроить отслеживание линка через технологию IP SLA

<details>
 <summary>R28</summary>

``` bash

conf t

route-map PBR_TO_R25 permit 10
  match ip address ACL_PBR_TO_R25
  set ip next-hop verify-availability 200.30.33.55 1 track 25
  exit

route-map PBR_TO_R25-v6 permit 10
  match ipv6 address ACL_PBR_TO_R25-v6
  set ipv6 next-hop 2001:ABCD:0030:2528::25
  exit

route-map PBR_TO_R26 permit 10
  match ip address ACL_PBR_TO_R26
  set ip next-hop verify-availability 200.30.30.36 1 track 26
  exit

route-map PBR_TO_R26-v6 permit 10
  match ipv6 address ACL_PBR_TO_R26-v6
  set ipv6 next-hop 2001:ABCD:0030:2628::26
  exit



  ip sla 25
  icmp-echo 200.30.33.55 source-interface e0/1
  frequency 15
ip sla schedule 25 life forever start-time now
track 25 ip sla 25 reachability

ip sla 256
  icmp-echo 2001:ABCD:0030:2528::25 source-interface e0/1
  frequency 15
ip sla schedule 256 life forever start-time now
track 256 ip sla 256 reachability

ip sla 26
  icmp-echo 200.30.30.36 source-interface e0/0
  frequency 15
ip sla schedule 26 life forever start-time now
track 26 ip sla 26 reachability

ip sla 266
  icmp-echo 2001:ABCD:0030:2628::26 source-interface e0/0
  frequency 15
ip sla schedule 266 life forever start-time now
track 266 ip sla 266 reachability

```

</details>

### 4. Настроить для офиса Лабытнанги маршрут по-умолчанию

``` bash

conf t
 ip route 0.0.0.0 0.0.0.0 200.30.33.25 1 name R25_Triada
 ipv6 route ::/0 2001:ABCD:0040:2527::25 1 name R25_Triada
 exit
 
```

![R27(sh_ip_route_stat)](https://raw.githubusercontent.com/Csalretepman/-OTUS-Network_Enginier/master/lab08/R27(sh_ip_route_stat).JPG)
# Лабораторная работа №13. BGP. Выбор пути. Фильтрация.



![top](/top.jpg)

#### 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(Prefix-list)

![mos](/mos.jpg)

Для запрета транзитного трафика в офисе Москва фильтрацией prefix-list'ами сохдадим данный лист под названием NO_PASARAN, воспользуемся регулярным выражением __ge__ 32 (запретим все сети с 32 маской за исключением линка до R22 в трафике от данного bgp соседа), таким образом будет анонсироваться лишь подсети из своей автономной системы (1001)

<details>
 <summary>R14</summary>

``` bash

conf t

 #add prefix-list
ip prefix-list NO_PASARAN permit 100.10.10.0/27
ip prefix-list NO_PASARAN deny 0.0.0.0/0 ge 32
ipv6 prefix-list NO_PASARAN_V6 permit 2001:ABCD:10:1422::/64
ipv6 prefix-list NO_PASARAN_V6 deny 0::/0 ge 128

router bgp 1001
#ipv4
 address-family ipv4
 neighbor 100.10.10.10 prefix-list NO_PASARAN out
#ipv6
 address-family ipv6
 neighbor 2001:ABCD:10:1422::22 prefix-list NO_PASARAN_V6 out
 end
wr mem
```

</details>

<details>
 <summary>R15</summary>

``` bash

conf t
 #add prefix-list
ip prefix-list NO_PASARAN permit 100.11.11.0/27
ip prefix-list NO_PASARAN deny 0.0.0.0/0 ge 32
ipv6 prefix-list NO_PASARAN_V6 permit 2001:ABCD:10:1521::/64
ipv6 prefix-list NO_PASARAN_V6 deny 0::/0 ge 128


router bgp 1001
#ipv4
 address-family ipv4
 neighbor 100.11.11.11 prefix-list NO_PASARAN out
#ipv6
 address-family ipv6
 neighbor 2001:ABCD:10:1521::21 prefix-list NO_PASARAN_V6 out
 end
wr mem
```

</details>



#### 2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(As-path)



![spb](/spb.jpg)

Все тоже самое, только через as-path access-list. Будем использовать регулярные выражения:
^$ маршруты локальной AS -- permit
.* любая строка -- deny

<details>
 <summary>R18</summary>

``` bash

conf t
 
 ip as-path access-list 100 permit ^$
 ip as-path access-list 100 deny .*
 
 router bgp 2042
#ipv4
 address-family ipv4
  neighbor 200.20.20.24 filter-list 100 out
  neighbor 200.20.20.36 filter-list 100 out
#ipv6
 address-family ipv6
  neighbor 2001:ABCD:0020:1824::24 filter-list 100 out
  neighbor 2001:ABCD:0020:1826::26 filter-list 100 out


```

</details>

#### 3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию

Через as-path acl запретим все (.* ) eBGP соседу R14 кроме маршрута по умолчанию (default-originate)

<details>
 <summary>R22</summary>

``` bash

conf t

ip as-path access-list 1 deny .*

router bgp 101
#ipv4
 address-family ipv4
 neighbor 100.10.10.14 default-originate
 neighbor 100.10.10.14 filter-list 1 out
#ipv6
 address-family ipv6
 neighbor 2001:ABCD:10:1422::14 default-originate
 neighbor 2001:ABCD:10:1422::14 filter-list 1 out
 end
wr mem
 
 

```

</details>


#### 4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург

Все тоже самое что в Киторне, только добавим  в as-path access-list разрешение для трафика из Питерского офиса регулярным выражением (permit _2042_)

<details>
 <summary>R21</summary>

``` bash
conf t

ip as-path access-list 1 permit _2042_
ip as-path access-list 1 deny .*
router bgp 301
#ipv4
 address-family ipv4
 neighbor 100.11.11.15 default-originate
 neighbor 100.11.11.15 filter-list 1 out
#ipv6
 address-family ipv6
 neighbor 2001:ABCD:10:1521::15 default-originate
 neighbor 2001:ABCD:10:1521::15 filter-list 1 out
 end
wr mem

```

</details>


<details>
 <summary>R14#sh bgp all</summary>

``` bash

R14#sh bgp all
For address family: IPv4 Unicast

BGP table version is 17, local router ID is 1.1.10.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r i 0.0.0.0          100.0.10.15              0    100      0 301 i
 r>                   100.10.10.10                           0 101 i
 *>  100.10.10.0/27   0.0.0.0                  0         32768 i

For address family: IPv6 Unicast

BGP table version is 2, local router ID is 1.1.10.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r i ::/0             FC00::10:15              0    100      0 301 i
 r>                   2001:ABCD:10:1422::22
                                                              0 101 i

For address family: IPv4 Multicast

```
</details>

<details>
 <summary>R15#sh bgp all</summary>

``` bash

R15#sh bgp all
For address family: IPv4 Unicast

BGP table version is 34, local router ID is 1.1.10.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          100.11.11.11                           0 301 i
 r i                  100.0.10.14              0    100      0 101 i
 *>i 100.10.10.0/27   100.0.10.14              0    100      0 i

For address family: IPv6 Unicast

BGP table version is 5, local router ID is 1.1.10.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  ::/0             2001:ABCD:10:1521::21
                                                              0 301 i
 r i                  FC00::10:14              0    100      0 101 i

For address family: IPv4 Multicast

```

</details>
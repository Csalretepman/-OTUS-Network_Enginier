# Настройка базового протокола OSPFv2 для одной области
## Топология

![](scheme.jpg)

## Таблица адресации

![](sheet.JPG)

Базовая настройка маршрутизаторов.

<details>
 <summary>R1</summary>

``` bash
Router(config)#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no logging console
R1(config)#no ip domain-lookup
R1(config)#int s1/0
R1(config-if)#ip address 192.168.12.1 255.255.255.252
R1(config-if)#clock rate 128000
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#int s1/1
R1(config-if)#ip address 192.168.13.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#int e0/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#service password-encryption
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#exec-timeout 5 0
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#exec-timeout 5 0
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#exit
R1#wr
Building configuration...
[OK]
R1#
```
</details>

<details>
 <summary>R2</summary>

``` bash
Router(config)#hostname R2
R2(config)#no logging console
R2(config)#no ip domain-lookup
R2(config)#int s1/0
R2(config-if)#ip address 192.168.12.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#int s1/1
R2(config-if)#ip address 192.168.23.1 255.255.255.252
R2(config-if)#clock rate 128000
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#int e0/0
R2(config-if)#ip address 192.168.2.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#service password-encryption
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#exec-timeout 5 0
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#exec-timeout 5 0
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#exit
R2#wr
Building configuration...
[OK]
R2#
```
</details>

<details>
 <summary>R3</summary>

``` bash
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R3
R3(config)#no logging console
R3(config)#no ip domain-lookup
R3(config)#int s1/0
R3(config-if)#ip address 192.168.13.2 255.255.255.252
R3(config-if)#clock rate 128000
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#int s1/1
R3(config-if)#ip address 192.168.23.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#int e0/0
R3(config-if)#ip address 192.168.3.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#service password-encryption
R3(config)#enable secret class
R3(config)#line console 0
R3(config-line)#password cisco
R3(config-line)#exec-timeout 5 0
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#line vty 0 4
R3(config-line)#password cisco
R3(config-line)#exec-timeout 5 0
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#exit
R3#wr
Building configuration...
[OK]
R3#wr

```
</details>

<details>
 <summary>Настройка компьютеров</summary>

Настройка PC-1
``` bash
VPCS> set pcname PC-1
PC-1> ip 192.168.1.3/24 192.168.1.1
Checking for duplicate address...
PC1 : 192.168.1.3 255.255.255.0 gateway 192.168.1.1
```
Настройка PC-2
``` bash
VPCS> set pcname PC-2
PC-2> ip 192.168.2.3/24 192.168.2.1
Checking for duplicate address...
PC1 : 192.168.2.3 255.255.255.0 gateway 192.168.2.1
```
Настройка PC-3
``` bash
VPCS> set pcname PC-3
PC-3> ip 192.168.3.3/24 192.168.3.1
Checking for duplicate address...
PC1 : 192.168.3.3 255.255.255.0 gateway 192.168.3.1

```
</details>

Проверка доступности маршрутизаторов 


![](pingr1.jpg)
![](pingr2.jpg)

### Часть 2. Настройка и проверка маршрутизации OSPF

Настроить протокол OSPF на маршрутизаторах.

<details>
 <summary>R1</summary>

``` bash
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#router ospf 1
R1(config-router)#network 192.168.1.0 0.0.0.255 area 0
R1(config-router)#network 192.168.12.0 0.0.0.3 area 0
R1(config-router)#network 192.168.13.0 0.0.0.3 area 0
R1(config-router)#exit
R1(config)#exit
R1#wr
```
</details>
<details>
 <summary>R2</summary>

``` bash
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#router ospf 1
R2(config-router)#network 192.168.2.0 0.0.0.255 area 0
R2(config-router)#network 192.168.12.0 0.0.0.3 area 0
R2(config-router)#network 192.168.23.0 0.0.0.3 area 0
R2(config-router)#exit
R2(config)#exit
R2#wr
```
</details>

<details>
 <summary>R3</summary>

``` bash
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#router ospf 1
R3(config-router)#network 192.168.3.0 0.0.0.255 area 0
R3(config-router)#network 192.168.13.0 0.0.0.3 area 0
R3(config-router)#network 192.168.23.0 0.0.0.3 area 0
R3(config-router)#exit
R3(config)#exit
R3#wr
```
</details>

Проверить информацию о соседних устройствах и маршрутизации OSPF

<details>
 <summary>show ip ospf nei </summary>

![](R1show-ip-ospf-neighbor.JPG)
![](R2show-ip-ospf-neighbor.JPG)
![](R3show-ip-ospf-neighbor.JPG)

</details>

<details>
 <summary>show ip route </summary>

![](R1sh-ip-route.JPG)
![](R2sh-ip-route.JPG)
![](R3sh-ip-route.JPG)

</details>

<details>
 <summary>show ip protocols </summary>

![](R1show-ip-protocols.JPG)
![](R2show-ip-protocols.JPG)
![](R3show-ip-protocols.JPG)

</details>

Какую команду вы бы применили, чтобы просмотреть в таблице маршрутизации только маршруты OSPF?

*show ip route ospf*

Проверить данные процесса OSPF

<details>
 <summary>R1</summary>

``` bash
R1#show ip ospf
 Routing Process "ospf 1" with ID 192.168.13.1
 
    Area BACKBONE(0)
        Number of interfaces in this area is 3
        
        SPF algorithm last executed 01:12:23.892 ago
        
```
</details>

<details>
 <summary>R2</summary>

``` bash
R2#show ip ospf
 Routing Process "ospf 1" with ID 192.168.23.1
     Area BACKBONE(0)
        Number of interfaces in this area is 3
        SPF algorithm last executed 01:16:48.954 ago
```
</details>
<details>
 <summary>R3</summary>

``` bash
R3#show ip ospf
 Routing Process "ospf 1" with ID 192.168.23.2
 
    Area BACKBONE(0)
        Number of interfaces in this area is 3
        SPF algorithm last executed 01:21:25.001 ago
```
</details>

Проверить параметры интерфейса OSPF

<details>
 <summary>show ip ip ospf int bri </summary>

![](R1show-ip-ospf-int-bri.JPG)
![](R2show-ip-ospf-int-bri.JPG)
![](R3show-ip-ospf-int-bri.JPG)

</details>

Наличие сквозного соединения

<details>
 <summary>PC ping </summary>

![](PC-1-ping.JPG)
![](PC-2-ping.JPG)
![](PC-3-ping.JPG)

</details>

### Часть 3. Изменение назначенных идентификаторов маршрутизаторов

##### Шаг 1. Изменить идентификаторы loopback-интерфейсов

Настроим значения loopback-интерфейсов маршрутизаторов

<details>
 <summary>R1</summary>

``` bash
R1#conf t
R1(config)#interface lo0
R1(config-if)#ip address 1.1.1.1 255.255.255.255
R1(config-if)#end
```
</details>

<details>
 <summary>R2</summary>

``` bash
R2#conf t
R2(config)#interface lo0
R2(config-if)#ip address 2.2.2.2 255.255.255.255
R2(config-if)#end
```
</details>

<details>
 <summary>R3</summary>

``` bash
R3#conf t
R3(config)#interface lo0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
R3(config-if)#end
```
</details>

<details>
 <summary>R1# (after change lo0) show ip ospf neighbor</summary>

``` bash
R1#sh ip ospf nei

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:34    192.168.13.2    Serial1/1
2.2.2.2           0   FULL/  -        00:00:35    192.168.12.2    Serial1/0
```
</details>

<details>
 <summary>R2# (after change lo0) show ip ospf neighbor</summary>

``` bash
R2#sh ip ospf nei

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:30    192.168.23.2    Serial1/1
3.3.3.3           0   FULL/  -        00:00:39    192.168.12.1    Serial1/0
```
</details>

<details>
 <summary>R3# (after change lo0) show ip ospf neighbor</summary>

``` bash
R3#sh ip ospf nei

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:38    192.168.13.1    Serial1/0
2.2.2.2           0   FULL/  -        00:00:32    192.168.23.1    Serial1/1
```
</details>

##### Шаг 2. Изменить идентификатор маршрутизаторов с помощью команды router-id.

Настроим значения router-id маршрутизаторов


<details>
 <summary>R1</summary>

``` bash
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#router ospf 1
R1(config-router)#router-id 11.11.11.11
% OSPF: Reload or use "clear ip ospf process" command, for this to take effect
R1(config-router)#end
R1#wr
R1#reload
```
</details>

<details>
 <summary>R2</summary>

``` bash
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#router ospf 1
R2(config-router)#router-id 22.22.22.22
% OSPF: Reload or use "clear ip ospf process" command, for this to take effect
R2(config-router)#end
R2#wr
R2#reload
```
</details>

<details>
 <summary>R3</summary>

``` bash
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#router ospf 1
R3(config-router)#router-id 33.33.33.33
% OSPF: Reload or use "clear ip ospf process" command, for this to take effect
R3(config-router)#end
R3#wr
R3#reload
```
</details>

<details>
 <summary>R1# (after change router-id) show ip ospf neighbor.txt</summary>

``` bash
R1#sh ip ospf nei

Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:30    192.168.13.2    Serial1/1
22.22.22.22       0   FULL/  -        00:00:37    192.168.12.2    Serial1/0
```
</details>

### Часть 4. Настройка пассивных интерфейсов OSPF

##### Шаг 1. Настроим интерфейсы e0/0 на всех роутерах, чтобы через них не пересылались обновления маршрутов.

Отключение обновлений на e0/0 интерфейсах

<details>
 <summary>R1</summary>

``` bash
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#router ospf 1
R1(config-router)#passive-interface e0/0
R1(config-router)#exit
R1(config)#exit
R1#wr

```
</details>

<details>
 <summary>R2</summary>

``` bash
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#router ospf 1
R2(config-router)#passive-interface e0/0
R2(config-router)#exit
R2(config)#exit
R2#wr

```
</details>

<details>
 <summary>R3</summary>

``` bash
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#router ospf 1
R3(config-router)#passive-interface e0/0
R3(config-router)#exit
R3(config)#exit
R3#wr

```
</details>

<details>
 <summary>Проверка passive-interface на e0/0</summary>

``` bash
R1#show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 192.168.1.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 10
  
    No Hellos (Passive interface)

```
</details>

Проверим, что у роутеров R2 и R3 не пропали маршруты до сети 192.168.1.0/24

<details>
 <summary>R2</summary>

``` bash
R2#sh ip route ospf

O     192.168.1.0/24 [110/74] via 192.168.12.1, 00:32:02, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.23.2, 00:29:41, Serial1/1
      192.168.13.0/30 is subnetted, 1 subnets
O        192.168.13.0 [110/128] via 192.168.23.2, 00:29:41, Serial1/1
                      [110/128] via 192.168.12.1, 00:32:02, Serial1/0
```
</details>

<details>
 <summary>R3</summary>

``` bash
R3#sh ip route ospf

O     192.168.1.0/24 [110/74] via 192.168.13.1, 00:29:59, Serial1/0
O     192.168.2.0/24 [110/74] via 192.168.23.1, 00:29:59, Serial1/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/128] via 192.168.23.1, 00:29:59, Serial1/1
                      [110/128] via 192.168.13.1, 00:29:59, Serial1/0

```
</details>

##### Шаг 2. Настроить на маршрутизаторе пассивный интерфейс в качестве интерфейса по умолчанию

<details>
 <summary>R1# show ip ospf neighbor</summary>

![](R1show-ip-ospf-neighbor-(with-router-id).JPG)

</details>

Сделаем все интерфейсы на R2 пассивными по-умолчанию.
``` bash
R2#conf t
R2(config)#router ospf 1
R2(config-router)#passive-interface default
```
Роутер R2 больше не является соседом R1 и R3, т.к. не делает рассылку маршрутов. Сейчас у маршрутизаторов R1 и R3 отсутствует маршрут к сети 192.168.2.0/24.
<details>
 <summary>Проверка отсутствия соседства R2 c R3</summary>

``` bash
R1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:37    192.168.13.2    Serial1/1
```
</details>

<details>
 <summary>Проверка отсутствия маршрута до 192.168.2.0/24</summary>

``` bash
R1#show ip route ospf

O     192.168.3.0/24 [110/74] via 192.168.13.2, 17:12:27, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 17:12:27, Serial1/1
```
</details>
Включим рассылку маршрутов на интерфейсе S1/0 у R2.

``` bash
R2#conf t
R2(config)#router ospf 1
R2(config-router)#no passive-interface s1/0
```
Теперь роутеры R1, R3 узнают про маршрут до 192.168.2.0/24

<details>

<details>
 <summary>R1#show ip route ospf</summary>

``` bash
R1#show ip route ospf

O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:01:17, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 23:14:10, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 23:14:10, Serial1/1
                      [110/128] via 192.168.12.2, 00:01:17, Serial1/0
```
</details>

#### Ответы на вопросы:

1. Какой интерфейс использует R3 для маршрута к сети 192.168.2.0/24?
   > s1/0
   
2. Чему равна суммарная метрика стоимости для сети 192.168.2.0/24 на R3?
    > 138 
3. Отображается ли маршрутизатор R2 как соседнее устройство OSPF на маршрутизаторе R1?
   > Да.  R1 подключён к s1/0 на R2, который анонсирует маршруты и отправляет hello-пакеты.
4. Отображается ли маршрутизатор R2 как соседнее устройство OSPF на маршрутизаторе R3?
   > Нет.  R3 подключён к s1/1 на R2. Интерфейс s1/1 в R2 в пассивном режиме и из-за этого не отправляет hello-пакеты.
5. Что дает вам эта информация?
   > Сведения о существующих подсетях и путь до них.
6. Настройте интерфейс S0/0/1 маршрутизатора R2 так, чтобы разрешить ему объявлять маршруты OSPF. Ниже запишите используемые команды.
    > R2#conf t
R2(config)#router ospf 1
R2(config-router)#no passive-interface s1/1

7. Какой интерфейс использует R3 для маршрута к сети 192.168.2.0/24?
   > s1/1
8. Отображается ли маршрутизатор R2 как сосед OSPF для маршрутизатора R3?
   > Отображается

### Часть 5. Изменение метрик OSPF

##### Шаг 1. Изменить заданную пропускную способность для маршрутизаторов

<details>
 <summary>Настройка стоимости маршрута относительно скорости порта</summary>

``` bash
R1#conf t
R1(config)#router ospf 1
R1(config-router)#auto-cost reference-bandwidth 10000
```
</details>

Теперь стоимость маршрутизации для s1/1 на R1 равна **6476**.

Вопрос: Почему может понадобиться изменить эталонную пропускную способность OSPF по умолчанию?

> Чтобы при расчёте маршрутов учитывать метрику скорости интерфейсов. Иначе, для всех интерфейсов она равна 1.

##### Шаг 2. Изменить пропускную способность для интерфейса

Изначально скорость порта s1/0 на R1 **1544 Kbit/sec**
<details>
 <summary>Стоимость пути до 192.168.23.0 на R1</summary>

``` bash
R1#show ip route ospf | begin 192.168.23.0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:09:36, Serial1/1
                      [110/128] via 192.168.12.2, 00:09:36, Serial1/0
```
</details>

Изменим значение метрики скорости интерфейса s1/0 R1 на 128. Таблица маршрутов до сети 23.х измениться. Останется только наилучший маршрут через s1/1 - 192.168.13.2 

<details>
 <summary>Останется только наилучший маршрут через s1/1 - 192.168.13.2</summary>

``` bash
R1#show ip route ospf | begin 192.168.23.0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:03:37, Serial1/1
```
</details>

Если изменить метрику скорости порта s1/1 на R1, то в таблице маршрутизации вновь будет два маршрута до сети 23.x, в виду того, что их суммарные стоимости будут равны.

<details>
 <summary>Маршруты OSPF на R1</summary>

``` bash
R1#show ip route ospf
O     192.168.2.0/24 [110/791] via 192.168.12.2, 00:02:30, Serial1/0
O     192.168.3.0/24 [110/791] via 192.168.13.2, 00:02:30, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/845] via 192.168.13.2, 00:02:30, Serial1/1
                      [110/845] via 192.168.12.2, 00:02:30, Serial1/0
```
</details>

Объясните, как были рассчитаны стоимости маршрутов от маршрутизатора R1 для сетей 192.168.3.0/24 и 192.168.23.0/30.

> Стоимость маршрута до сети 192.168.3.0/24 равна сумме стоимостей интерфейсов s1/1 на R1 и R3 = 791 (781+10)
Стоимость маршрута до сети 192.168.23.0/24 равна сумме стоимостей интерфейсов s1/1 R1 и e0/0 на R3 = 845 (781+64)

Изменили значения метрики стоимости всех последовательных интерфейсов на 128.
В результате новая суммарная стоимость для сети 192.168.23.0/24 на R1 равна 1562 (781+781).

##### Шаг 3. Измените стоимость маршрута.

Сейчас стоимость пути до сети 192.168.3.0/24 равна 791
<details>
 <summary>R1#sh ip route ospf</summary>
 
 ``` bash
R1#show ip route ospf | begin 192.168.3.0
O     192.168.3.0/24 [110/791] via 192.168.13.2, 00:32:33, Serial1/1

```
</details>
Изменим значение метрики стоимости порта с помощью команды _ip ospf cost_ на 1565

<details>
 <summary>R1</summary>
 
``` bash
R1#conf t
R1(config)#interface s1/1
R1(config-if)#ip ospf cost 1565
```

</details>

Теперь маршрут до 3.x/24 находится за s1/0 -192.168.12.2

<details>
 <summary>R1</summary>

``` begin

R1#show ip route ospf

O     192.168.2.0/24 [110/791] via 192.168.12.2, 00:44:06, Serial1/0
O     192.168.3.0/24 [110/1572] via 192.168.12.2, 00:01:02, Serial1/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/1562] via 192.168.12.2, 00:22:28, Serial1/0
```

</details>

Почему маршрут к сети 192.168.3.0/24 от маршрутизатора R1 теперь проходит через R2?
> Суммарная стоимость маршрута меньше, чем через R3.

### Вопросы для повторения

1. Почему так важно управлять назначением идентификатора маршрутизатора при использовании протокола OSPF?
    > Индентификатор маршрутизатора нужен для выбора DR и BDR. Для выставления приоритета тому или иному роутеру можно изменить один из параметров, влияющих на идентификацию роутеров в ospf.

2. Почему в этой лабораторной работе не рассматривается процесс выбора DR/BDR?
    > Роутеры подключены по схеме "точка-точка". Выбор DR и BDR нужен для уменьшения мультикаста в сетях с множественным доступом.

3. Почему рекомендуется настраивать интерфейс OSPF как пассивный?
    > Чтобы не делать рассылку рабочей информации о маршрутизации в сторону локальной сети.
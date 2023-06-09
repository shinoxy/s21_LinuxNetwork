# s21_LinuxNetwork

Практическая работа по настройке сетей в Linux на виртуальных машинах.

# Part 1 Инструмент ipcalc

## 1.1 Сети и маски

1. Адрес сети для 192.167.38.54/13 - **192.160.0.0/13**  

2. Маска подсети 255.255.255.0 в префиксной записи: **/24**  
   Маска подсети 255.255.255.0 в двоичной записи: **11111111.11111111.11111111.00000000**  
   Префикс /15 в обычной записи: **255.254.0.0**  
   Префикс /15 в двоичной записи: **11111111.11111110.00000000.00000000**  
   Маска подсети 11111111.11111111.11111111.11110000 в обычной записи: **255.255.255.240**  
   Маска подсети 11111111.11111111.11111111.11110000 в префиксной записи: **/28**  

3. Минимальный и максимальный хост в сети 12.167.38.4 при масках:  
		/8 - **12.0.0.1** и **12.255.255.254**  
		11111111.11111111.00000000.00000000 - **12.167.0.1** и **12.167.255.254**  
		255.255.254.0 - **12.167.38.1** и **12.167.39.254**  
		/4 - *0.0.0.1* *15.255.255.254*  

## 1.2. localhost

Можно ли обратиться к приложению, работающему на localhost, со следующими IP:  
		194.34.23.100 - **Нельзя**  
		127.0.0.2 - **Можно, loopback**  
		127.1.0.1 - **Можно, loopback**  
		128.0.0.1 - **Нельзя**  

## 1.3. Диапазоны и сегменты сетей

1. В качестве публичных можно использовать:  
		134.43.0.2  
		192.172.0.1  
		172.68.0.2  
		192.169.168.1  
		  
   В качестве частных можно использовать:  
		10.0.0.45  
		192.168.4.2  
		172.20.250.4  
		172.0.2.1  
		172.16.255.255  
		10.10.10.10  

2. У сети 10.10.0.0/18 могут быть следующие IP-адреса:  
		10.10.0.2  
		10.10.10.10  
		10.10.1.255  


# Part 2. Статическая маршрутизация между двумя машинами

## 2.0 Смотрим существующие сетевые интерфейсы

Используем **ip a**

![ip a](./images/2.1.jpg)

Задаем следующие адреса и маски: 
		ws1 - **192.168.100.10/16**, 
		ws2 - **172.24.116.8/12**

И перезапускаем сервис сети с помощью **netplan apply**

![set addresses](./images/2.2.jpg)

## 2.1 Добавление статического маршрута вручную

Добавляем статический маршрут от одной машины до другой и обратно

**sudo ip r add 192.168.100.10 via 172.24.116.8 dev enp0s3**
**sudo ip r add 172.24/11.116.8 via 192.168.100.10 dev enp0s3**

![set routes manually](./images/2.3.jpg)

## 2.2 Добавление статического маршрута с сохранением

На данном этапе машина уже перезапущена.

Добавляем статический маршрут от одной машины до другой с помощью файла etc/netplan/00-installer-config.yaml

![set routes via netplan](./images/2.4.jpg)

Пингуем соединение между машинами

![ping machines](./images/2.5.jpg)


# Part 3. Утилита iperf3

## 3.1. Скорость соединения

8 MB/s - это **1 MB/s**
100 MB/s - это **800000 KB/s**
1 GB/s - это **1000 MB/s**

## 3.2. Утилита iperf3

Чтобы измерить скорость соединения:

1. Запускаем iperf3 командой **iperf3 -s**
2. Проверяем скорость от 192.168.100.10 к 172.24.116.8 командой **iperf3 -c 172.24.116.8**

![iperf3 report](./images/3.1.jpg)


# Part 4. Сетевой экран

## 4.1. Утилита iptables

Создаем правила согласно заданию, запускаем файлы на обеих машинах командами chmod +x /etc/firewall.sh и /etc/firewall.sh

![iptables](./images/4.1.jpg)

Разница между стратегиями заключается в том, что в первом файле первым подходящим правилом для пакета является запрет, а во втором - разрешение. Применяется только первое подходящее правило, остальные игнорируются.

## 4.2. Утилита nmap 

Пингуем обе машины. 

![ping machines](./images/4.2.jpg)

Видим, что ws2 не пингуется с ws1, проверяем утилитой nmap. 

![nmap](./images/4.3.jpg)

Видим, что Host is up


# Part 5. Статическая маршрутизация сети  

Cоздаем сеть согласно заданному условию (3 рабочие станции (ws11, ws21, ws22) и 2 роутера (r1, r2)):

![network](./images/5.1.jpg)

## 5.1. Настройка адресов машин 

Создаем по два адаптера для обоих роутеров - под сетевой мост и под отдельные подсети net1 и net2.

![net1](./images/5.2.jpg)

![net2](./images/5.3.jpg)

Настраиваем рабочие станции в их **/etc/netplan/00-installer-config.yaml** и сохраняем результат с помощью **netplan apply**:

![ws11](./images/5.4.jpg)

![ws21](./images/5.5.jpg)

![ws22](./images/5.6.jpg)

Настраиваем роутеры в их **/etc/netplan/00-installer-config.yaml** и сохраняем результат с помощью **netplan apply**:

![r1 and r2](./images/5.7.jpg)

В нашем случае, чтобы не менять лишнего, возьмем за eth0 сеть enp0s3, а за eth1 - enp0s8

Проверяем соединения с помощью **ip -4 a**

![ws11 ip a](./images/5.8.jpg)

![ws21 ip a](./images/5.9.jpg)

![ws22 ip a](./images/5.10.jpg)

![r1 and r2 ip a](./images/5.11.jpg)

Пингуем ws22 с ws21 и r1 с ws11

![ping ws22 from ws21](./images/5.12.jpg)

![ping r1 from ws11](./images/5.13.jpg)

## 5.2. Включение переадресации IP-адресов.

Вызов команды **sudo sysctl -w net.ipv4.ip_forward=1** для машин r1 и r2.
При таком подходе переадресация не будет работать после перезагрузки системы.

![forwarding IP 1](./images/5.14.jpg)

Открываем файл **/etc/sysctl.conf** и добавляем в него **net.ipv4.ip_forward = 1**. 
При использовании этого подхода, IP-переадресация включена на постоянной основе.

![forwarding IP 2](./images/5.15.jpg)

## 5.3. Установка маршрута по-умолчанию 

Настраиваем маршрут по-умолчанию (шлюз) для рабочих станций в файле конфигураций **etc/netplan/00-installer-config.yaml** 

![ws11 default route](./images/5.16.jpg)

![ws21 default route](./images/5.17.jpg)

![ws22 default route](./images/5.18.jpg)

Пингуем r2 с ws11 и проверяем, что пинг доходит командой **tcpdump -tn -i eth1**

![ping r2 from ws11](./images/5.19.jpg)

## 5.4. Добавление статических маршрутов 

Добавляем в роутеры r1 и r2 статические маршруты в файле конфигураций.

![config r1 and r2](./images/5.20.jpg)

Вызываем **ip r** чтобы увидеть таблицы с маршрутами на обоих роутерах

![ip r r1 and r2](./images/5.21.jpg)

Запускаем на ws11 **ip r list 10.10.0.0/[маска сети]** и **ip r list 0.0.0.0/0**

![ws11 ip r list](./images/5.22.jpg)

Для адреса **10.10.0.0/[порт сети]** был выбран маршрут, отличный от **0.0.0.0/0**, потому что порт **/18** описывает маршрут к сети точнее, в отличие от порта **/0**

## 5.5. Построение списка маршрутизаторов

Вывод команды traceroute от ws11 до ws21:

![ws11 traceroute](./images/5.23.png)

Захват трафика на r1 c с помощью **tcpdump -tnv -i eth0** при выполнении трассировки (команда **traceroute**) на ws11:

![r1 tcpdump](./images/5.24.png)

Хост, с которого выполняется трассировка, отправляет пакеты на адрес назначения с разными показателями "времени жизни" TTL, начиная с 1 и постепенно его увеличивая. Каждый хост на пути к назначению отправляет пакет **ICMP time exceeded in-transit**, показывая, что пакет ещё не дошёл до адреса назначения. Адрес отправителя в данном пакете фиксируется в трассировке, как промежуточное звено на пути к адресу назначения. По умолчанию запросы от хоста-источника отправляются с помощью "проб"-пакетов (probes) по протоколу UDP, но с помощью ключей **-I** и **-T** можно заменить протокол на ICMP или TCP соответственно.

## 5.6. Использование протокола ICMP при маршрутизации  

Вывод команд **ping -c 1 10.30.0.111** на ws11 и **tcpdump -n -i eth0 icmp** на r1

![ICMP](./images/5.25.jpg)


# Part 6. Динамическая настройка IP с помощью DHCP  

Для r2 настроим в **/etc/dhcp/dhcpd.conf** конфигурацию службы DHCP:

1) указываем адрес маршрутизатора по-умолчанию, DNS-сервер и адрес внутренней сети

![r2 dhcpd.conf](./images/6.1.jpg)

2) в файле **/etc/resolv.conf** прописываем **nameserver 8.8.8.8**

![r2 resolv.conf](./images/6.2.jpg)

Перезагружаем службу DHCP командой **systemctl restart isc-dhcp-server**

![r2 restart dhcp](./images/6.3.jpg)

Машину ws21 перезагружаем при помощи **reboot** и через **ip a** видим, что она получила адрес

![ws11 ip a](./images/6.4.jpg)

Пингуем ws22 с ws21.

![ping ws11 from ws22](./images/6.5.jpg)

Указываем MAC адрес у ws11, для этого в **etc/netplan/00-installer-config.yaml** добавляем строки: **macaddress: 10:10:10:10:10:BA**, **dhcp4: true**

![MAC address ws11](./images/6.6.jpg)

Настраиваем r1 аналогично r2, но делаем выдачу адресов с жесткой привязкой к MAC-адресу (ws11). Проводим аналогичные тесты

![r1 dhcpd.conf](./images/6.7.jpg)

![r1 resolv.conf](./images/6.8.jpg)

![r1 restart dhcp](./images/6.9.jpg)

![ws11 ip a](./images/6.10.jpg)

Запрашиваем с ws21 обновление ip адреса

![ws21 ip a](./images/6.11.jpg)


# Part 7. NAT 

Делаем сервер Apache2 общедоступным: в файле **/etc/apache2/ports.conf** на ws22 и r1 меняем строку **Listen 80** на **Listen 0.0.0.0:80**

![ports.conf](./images/7.1.jpg)

Запускаем веб-сервер Apache командой **service apache2 start** на ws22 и r1

![apache start](./images/7.2.jpg)

Добавляем в файервол **/etc/firewall.sh** на r2 следующие правила:

1) удаление правил в таблице filter - **iptables -F**

2) удаление правил в таблице "NAT" - **iptables -F -t nat**

3) отбрасывать все маршрутизируемые пакеты - **iptables --policy FORWARD DROP**

Запускаем файл

![firewall r2](./images/7.3.jpg)

Проверяем соединение между ws22 и r1. При запуске файла с этими правилами, ws22 не должна пинговаться с r1

![ping ws22 r1](./images/7.4.jpg)

Добавляем в файл ещё одно правило:

4) разрешить маршрутизацию всех пакетов протокола ICMP
   
Запускаем файл

![firewall r2](./images/7.5.jpg)

Проверяем соединение между ws22 и r1. При запуске файла с этими правилами, ws22 должна пинговаться с r1

![ping ws22 r1](./images/7.6.jpg)

Добавляем в файл ещё два правила:

5) включить SNAT, а именно маскирование всех локальных ip из локальной сети, находящейся за r2 (по обозначениям из Части 5 - сеть **10.20.0.0**)

6) включить DNAT на 8080 порт машины r2 и добавить к веб-серверу Apache, запущенному на ws22, доступ извне сети

![DNAT and SNAT](./images/7.7.jpg)

Запускаем файл

Проверяем соединение по TCP для SNAT, для этого с ws22 подключаемся к серверу Apache на r1 командой **telnet [адрес] [порт]**

![SNAT check](./images/7.8.jpg)

Проверяем соединение по TCP для DNAT, для этого с r1 подключаемся к серверу Apache на ws22 командой **telnet [адрес r2] [порт 8080]**

![DNAT check](./images/7.9.jpg)

# Part 8. Знакомство с SSH Tunnels

Запускаем на r2 фаервол с правилами из Части 7

![DNAT and SNAT](./images/8.1.jpg)

Запускаем Apache на ws22 только на localhost (то есть в файле **/etc/apache2/ports.conf** меняем строку **Listen 80** на **Listen localhost:80**)

![apache localhost](./images/8.2.jpg)

Воспользуeмся **Local TCP forwarding** с ws21 до ws22, чтобы получить доступ к веб-серверу на ws22 с ws21

![Local TCP forwarding 1](./images/8.3.jpg)

![Local TCP forwarding 2](./images/8.4.jpg)

Воспользуемся Remote TCP forwarding c ws11 до ws22, чтобы получить доступ к веб-серверу на ws22 с ws11

![Remote TCP forwarding](./images/8.5.jpg)

Для проверки, сработало ли подключение в обоих предыдущих пунктах, выполняем **telnet 127.0.0.1 [локальный порт]**

![telnet report](./images/8.6.jpg)




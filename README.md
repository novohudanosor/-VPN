# -VPN
Мосты, туннели и VPN 
**Домашнее задание VPN**
* Создать домашнюю сетевую лабораторию. Научится настраивать VPN-сервер в Linux-based системах.
* Описание домашнего задания
Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ

1. Поднимаем 2 виртуальные машины ``` vagrant up ```
2. Нужно настроить VPN между двумя хостами. Устанавливаем на хостах пакет OpenVPN и программу для измерения пропускной способности сети **iperf3**
```
sudo su -
apt update
apt upgrade
apt install -y openvpn iperf3
```
3. Генерируем ключи на сервере
```
openvpn --genkey secret /etc/openvpn/static.key
```
4. В конфигурационный файл OpenVPN внесем настройки
```
root@server:~# nano /etc/openvpn/server.conf
root@server:~# cat /etc/openvpn/server.conf
dev tap
ifconfig 10.10.10.1 255.255.255.0
topology subnet
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
```
5. На сервере подготовим модуль **systemd** для запуска OpenVPN
```
root@server:~# nano /etc/systemd/system/openvpn@.service
root@server:~# cat /etc/systemd/system/openvpn@.service
[Unit]
Description=OpenVPN Tunneling Application On %I
After=network.target
[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf
[Install]
WantedBy=multi-user.target
root@server:~# systemctl start openvpn@server
root@server:~# systemctl enable openvpn@server
Created symlink /etc/systemd/system/multi-user.target.wants/openvpn@server.service → /etc/systemd/system/openvpn@.service.
```
6. На хосте клиент настраиваем конфигурационный файл для **OpenVPN**
```
root@client:~# nano /etc/openvpn/server.conf
root@client:~# cat /etc/openvpn/server.conf
dev tap
remote 192.168.56.10
ifconfig 10.10.10.2 255.255.255.0
topology subnet
route 192.168.56.0 255.255.255.0
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
```
7. Сгенерированный файл на сервере **/etc/openvpn/static.key**  копируем на клиента в директорию /etc/openvpn/
8. На клиенте создаем такой же systemd модуль, и запускаем.
```
nano /etc/systemd/system/openvpn@.service
cat /etc/systemd/system/openvpn@.service
[Unit]
Description=OpenVPN Tunneling Application On %I
After=network.target
[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf
[Install]
WantedBy=multi-user.target

systemctl start openvpn@server
systemctl enable openvpn@server
```
9.  При помощи iperf замеряем скорость
10.  на сервере запускаем iperf d ht;bvt cthdthf
```
iperf3 -s &
```
* запускаем iperf на клиенте
```
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 60482 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  12.5 MBytes  20.9 Mbits/sec    0    577 KBytes
[  5]   5.00-10.00  sec  11.2 MBytes  18.7 Mbits/sec    0   1.10 MBytes
[  5]  10.00-15.00  sec  11.1 MBytes  18.6 Mbits/sec   51   1.11 MBytes
[  5]  15.00-20.00  sec  11.2 MBytes  18.8 Mbits/sec   17    971 KBytes
[  5]  20.00-25.00  sec  11.1 MBytes  18.6 Mbits/sec    0   1.07 MBytes
[  5]  25.00-30.00  sec  10.0 MBytes  16.8 Mbits/sec    0   1.10 MBytes
[  5]  30.00-35.00  sec  11.2 MBytes  18.8 Mbits/sec   34    933 KBytes
[  5]  35.00-40.00  sec  10.0 MBytes  16.8 Mbits/sec    0   1.24 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec  88.3 MBytes  18.5 Mbits/sec  102             sender
[  5]   0.00-41.14  sec  88.0 MBytes  17.9 Mbits/sec                  receiver

iperf Done.
```


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
10.  на сервере запускаем iperf и проверяем скорость 
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
* TUN и TAP — виртуальные сетевые драйверы ядра системы. Они представляют собой программные сетевые устройства, которые отличаются от обычных аппаратных сетевых карт.
TAP эмулирует Ethernet-устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet. TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP-пакетами. TAP используется для создания сетевого моста, тогда как TUN — для маршрутизации.
11.  На клиенет и на сервере правим файл **/etc/openvpn/server.conf** меняем tap на tun и перезапускаем сервис ``` systemctl restart openvpn@server.service ```  Запускаем iperf и проверяем скорость:
```
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 57012 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  12.3 MBytes  20.6 Mbits/sec    0    633 KBytes
[  5]   5.00-10.00  sec  12.1 MBytes  20.3 Mbits/sec   30    453 KBytes
[  5]  10.00-15.00  sec  11.1 MBytes  18.7 Mbits/sec    0    551 KBytes
[  5]  15.00-20.00  sec  11.9 MBytes  19.9 Mbits/sec    0    587 KBytes
[  5]  20.00-25.00  sec  11.4 MBytes  19.1 Mbits/sec   13    603 KBytes
[  5]  25.00-30.00  sec  12.0 MBytes  20.2 Mbits/sec    1    536 KBytes
[  5]  30.00-35.00  sec  10.4 MBytes  17.5 Mbits/sec    0    554 KBytes
[  5]  35.00-40.00  sec  11.6 MBytes  19.4 Mbits/sec    9    506 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec  92.8 MBytes  19.5 Mbits/sec   53             sender
[  5]   0.00-40.27  sec  91.6 MBytes  19.1 Mbits/sec                  receiver

iperf Done.
```
12. В рамках данной задачи получили примерно одинаковые скорости. на практике TUN должен иметь более высокую производительность, т.к. не обрабатывает широковещательный трафик, характерных для режима TAP.
## Настройка RAS на базе OpenVPN
1. Устанавливаем на сервер пакет easy-rsa для развёртывания центра серитификации:
```
apt update
apt install -y openvpn easy-rsa
```
2. Инициализация PKI:
```
cd /etc/openvpn
/usr/share/easy-rsa/easyrsa init-pki
```
3. Генерируем необходимые ключи и сертификаты для сервера:
```
echo 'rasvpn' | /usr/share/easy-rsa/easyrsa gen-req server nopass
echo 'yes' | /usr/share/easy-rsa/easyrsa sign-req server server 
/usr/share/easy-rsa/easyrsa gen-dh
openvpn --genkey secret ca.key
```
4. Генерируем необходимые ключи и сертификаты для клиента
```
echo 'client' | /usr/share/easy-rsa/easyrsa gen-req client nopass
echo 'yes' | /usr/share/easy-rsa/easyrsa sign-req client client
```
5. Создаем конфигурационный файл сервера ```nano /etc/openvpn/server.conf   ```
```
port 1207
proto udp
dev tun
ca /etc/openvpn/pki/ca.crt
cert /etc/openvpn/pki/issued/server.crt
key /etc/openvpn/pki/private/server.key
dh /etc/openvpn/pki/dh.pem
server 10.10.10.0 255.255.255.0
push "route 192.168.56.0 255.255.255.0"
ifconfig-pool-persist ipp.txt
client-to-client
client-config-dir /etc/openvpn/client
keepalive 10 120
comp-lzo
persist-key
persist-tun
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3

```
6. Задание параметра iroute для клиента:  ``` echo 'iroute 192.168.56.0 255.255.255.0' > /etc/openvpn/client/client   ```
7. Запускаем сервис
```
systemctl start openvpn@server
systemctl enable openvpn@server
```
* Копирование указанных ниже файлов сертификатов и ключа на клентскую машину в **/etc/openvpn**:
```
/etc/openvpn/pki/ca.crt
/etc/openvpn/pki/issued/client.crt
/etc/openvpn/pki/private/client.key
```
* Конфигурационный файл на клиенте ``` etc/openvpn/client.conf ```
```
dev tun
proto udp
remote 192.168.56.10 1207
client
resolv-retry infinite
ca ./ca.crt
cert ./client.crt
key ./client.key
persist-key
persist-tun
comp-lzo
verb 3
```
* подключаемся с хостовой машины: 

```
cd /etc/openvpn/
openvpn --config client.conf
```
```
2024-08-11 13:00:14 WARNING: Compression for receiving enabled. Compression has been used in the past to break encryption. Sent packets are not compressed unless "allow-compression yes" is also set.
2024-08-11 13:00:14 Note: --cipher is not set. OpenVPN versions before 2.5 defaulted to BF-CBC as fallback when cipher negotiation failed in this case. If you need this fallback please add '--data-ciphers-fallback BF-CBC' to your configuration and/or add BF-CBC to --data-ciphers.
2024-08-11 13:00:14 OpenVPN 2.6.9 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [MH/PKTINFO] [AEAD]
2024-08-11 13:00:14 library versions: OpenSSL 3.0.13 30 Jan 2024, LZO 2.10
2024-08-11 13:00:14 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
2024-08-11 13:00:14 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.56.10:1207
2024-08-11 13:00:14 Socket Buffers: R=[212992->212992] S=[212992->212992]
2024-08-11 13:00:14 UDPv4 link local: (not bound)
2024-08-11 13:00:14 UDPv4 link remote: [AF_INET]192.168.56.10:1207
2024-08-11 13:00:14 TLS: Initial packet from [AF_INET]192.168.56.10:1207, sid=c5707599 e1e34c53
2024-08-11 13:00:14 VERIFY OK: depth=1, CN=rasvpn
2024-08-11 13:00:14 VERIFY OK: depth=0, CN=rasvpn
2024-08-11 13:00:14 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, peer certificate: 2048 bits RSA, signature: RSA-SHA256, peer temporary key: 253 bits X25519
2024-08-11 13:00:14 [rasvpn] Peer Connection Initiated with [AF_INET]192.168.56.10:1207
2024-08-11 13:00:14 TLS: move_session: dest=TM_ACTIVE src=TM_INITIAL reinit_src=1
2024-08-11 13:00:14 TLS: tls_multi_process: initial untrusted session promoted to trusted
2024-08-11 13:00:14 PUSH: Received control message: 'PUSH_REPLY,route 10.10.10.0 255.255.255.0,topology net30,ping 10,ping-restart 120,ifconfig 10.10.10.6 10.10.10.5,peer-id 0,cipher AES-256-GCM'
2024-08-11 13:00:14 OPTIONS IMPORT: --ifconfig/up options modified
2024-08-11 13:00:14 OPTIONS IMPORT: route options modified
2024-08-11 13:00:14 net_route_v4_best_gw query: dst 0.0.0.0
2024-08-11 13:00:14 net_route_v4_best_gw result: via 192.168.55.1 dev wlp2s0
2024-08-11 13:00:14 ROUTE_GATEWAY 192.168.55.1/255.255.255.0 IFACE=wlp2s0 HWADDR=4c:d5:77:7b:e2:87
2024-08-11 13:00:14 TUN/TAP device tun0 opened
2024-08-11 13:00:14 net_iface_mtu_set: mtu 1500 for tun0
2024-08-11 13:00:14 net_iface_up: set tun0 up
2024-08-11 13:00:14 net_addr_ptp_v4_add: 10.10.10.6 peer 10.10.10.5 dev tun0
2024-08-11 13:00:14 net_route_v4_add: 10.10.10.0/24 via 10.10.10.5 dev [NULL] table 0 metric -1
2024-08-11 13:00:14 Initialization Sequence Completed
2024-08-11 13:00:14 Data Channel: cipher 'AES-256-GCM', peer-id: 0, compression: 'lzo'
2024-08-11 13:00:14 Timers: ping 10, ping-restart 120
```
```
ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.389 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.616 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.327 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.577 ms

--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3030ms
rtt min/avg/max/mdev = 0.327/0.477/0.616/0.122 ms
```

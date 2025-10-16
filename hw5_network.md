
## Задание 1

Находим сокет с http сервером
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ ss -tlnp | grep 8080
LISTEN 0      5            0.0.0.0:8080       0.0.0.0:*    users:(("python3",pid=5189,fd=3))
```


```bash
denisden@LAPTOP-RMQ088BV:~/vk$ curl http://localhost:8080/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".git/">.git/</a></li>
<li><a href="hw5_network.md">hw5_network.md</a></li>
<li><a href="solution.md">solution.md</a></li>
<li><a href="task_3/">task_3/</a></li>
</ul>
<hr>
</body>
</html>
```


```bash
denisden@LAPTOP-RMQ088BV:~/vk$ ss -tan | grep 8080
LISTEN    0      5             0.0.0.0:8080        0.0.0.0:*           
TIME-WAIT 0      0           127.0.0.1:8080      127.0.0.1:59168       

```

TIME-WAIT - это состояние, в котором сокет находится после закрытия TCP-соединения

TIME-WAIT - обеспечивает гаранитию того, что старые пакеты не повлияют на новое соединение
- Обеспечивает надежное закрыти сокета
Основные проблема большого количества сокетов исчерпание портов - система не может создавать новые соединения. Каждый сокет потребляет ресурсы ядра



## Задание 2

Создаем dummy интерфейсы
```bash

sudo ip link add service_0 type dummy
sudo ip addr add 192.168.14.88/32 dev service_0
sudo ip link set service_0 up


sudo ip link add service_1 type dummy
sudo ip addr add 192.168.14.1/30 dev service_1
sudo ip link set service_1 up

sudo ip link add service_2 type dummy
sudo ip addr add 192.168.10.4/32 dev service_2
sudo ip link set service_2 up

sudo ip link add srv_1 type dummy
sudo ip addr add 192.168.14.4/32 dev srv_1
sudo ip link set srv_1 up
```


Проверяем что интерфейсы работают

```basdenisden@LAPTOP-RMQ088BV:/etc$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:34:47:87 brd ff:ff:ff:ff:ff:ff
    inet 172.25.166.217/20 brd 172.25.175.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe34:4787/64 scope link 
       valid_lft forever preferred_lft forever
3: service_0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 8e:cf:c4:33:85:fc brd ff:ff:ff:ff:ff:ff
    inet 192.168.14.88/32 scope global service_0
       valid_lft forever preferred_lft forever
    inet6 fe80::8ccf:c4ff:fe33:85fc/64 scope link 
       valid_lft forever preferred_lft forever
4: service_1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 02:fb:88:c7:40:85 brd ff:ff:ff:ff:ff:ff
    inet 192.168.14.1/30 scope global service_1
       valid_lft forever preferred_lft forever
    inet6 fe80::fb:88ff:fec7:4085/64 scope link 
       valid_lft forever preferred_lft forever
5: service_2: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether e6:22:f4:fe:b0:27 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.4/32 scope global service_2
       valid_lft forever preferred_lft forever
    inet6 fe80::e422:f4ff:fefe:b027/64 scope link 
       valid_lft forever preferred_lft forever
6: srv_1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 8a:7b:44:a6:7f:a8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.14.4/32 scope global srv_1
       valid_lft forever preferred_lft forever
    inet6 fe80::887b:44ff:fea6:7fa8/64 scope link 
       valid_lft forever preferred_lft forever
denisden@LAPTOP-RMQ088BV:/etc$ ip route show
default via 172.25.160.1 dev eth0 proto kernel 
172.25.160.0/20 dev eth0 proto kernel scope link src 172.25.166.217 
192.168.14.0/30 dev service_1 proto kernel scope link src 192.168.14.1 
```


Создаем файл bird.conf
```bash
router id 192.168.14.88;

protocol kernel {
    ipv4 {
        import all;
        export where source = RTS_STATIC;
    };
    learn;
    persist;
}

protocol device {
    scan time 10;
}

protocol rip {
    ipv4 {
        import all;
        export where (net ~ 192.168.14.0/24) && (net.len = 32) && (ifname ~ "service_*");
    };
    
    interface "eth0" {  
        version 2;
        update time 10;
        timeout time 30;
        authentication none;
    };
    
    interface "service_*" {
        mode passive;
    };
}
```


```bash
denisden@LAPTOP-RMQ088BV:/etc$ sudo systemctl status bird
● bird.service - BIRD Internet Routing Daemon
     Loaded: loaded (/lib/systemd/system/bird.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-10-16 18:31:38 MSK; 2min 33s ago
    Process: 44515 ExecStartPre=/usr/lib/bird/prepare-environment (code=exited, status=0/SUCCESS)
    Process: 44521 ExecStartPre=/usr/sbin/bird -p (code=exited, status=0/SUCCESS)
   Main PID: 44522 (bird)
      Tasks: 1 (limit: 5890)
     Memory: 796.0K
        CPU: 51ms
     CGroup: /system.slice/bird.service
             └─44522 /usr/sbin/bird -f -u bird -g bird
```

## Задание 3

Устанавливаем правило фаервола
```bash
sudo tcpdump -i any port 8080
```

Наблюдаем за tcpdump
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo tcpdump -i any port 8080 
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
17:54:56.009026 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217779284 ecr 0,nop,wscale 7], length 0
17:54:56.210493 lo    In  IP6 ip6-localhost.48822 > ip6-localhost.http-alt: Flags [S], seq 4192534456, win 65476, options [mss 65476,sackOK,TS val 1291837752 ecr 0,nop,wscale 7], length 0
17:54:56.210506 lo    In  IP6 ip6-localhost.http-alt > ip6-localhost.48822: Flags [R.], seq 0, ack 4192534457, win 0, length 0
17:54:57.020362 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217780296 ecr 0,nop,wscale 7], length 0
17:54:58.044344 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217781320 ecr 0,nop,wscale 7], length 0
17:54:59.068417 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217782344 ecr 0,nop,wscale 7], length 0
17:55:00.092305 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217783368 ecr 0,nop,wscale 7], length 0
17:55:01.116287 lo    In  IP localhost.50272 > localhost.http-alt: Flags [S], seq 3331405815, win 65495, options [mss 65495,sackOK,TS val 4217784392 ecr 0,nop,wscale 7], length 0
```
Видим флаг [R] - RST означающий немедленный сброс соединения

Подключиться к серверу не получается
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ curl -v http://localhost:8080 
*   Trying 127.0.0.1:8080...
*   Trying ::1:8080...
* connect to ::1 port 8080 failed: Connection refused
* connect to 127.0.0.1 port 8080 failed: Connection timed out
* Failed to connect to localhost port 8080 after 133652 ms: Connection timed out
* Closing connection 0
curl: (28) Failed to connect to localhost port 8080 after 133652 ms: Connection timed out
```


## Задание 4

Проверяем доступные интерфейсы и иследование offload  возможностей адаптера

```bash
denisden@LAPTOP-RMQ088BV:~/vk$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:34:47:87 brd ff:ff:ff:ff:ff:ff
denisden@LAPTOP-RMQ088BV:~/vk$ ethtool -k eth0
Features for eth0:
rx-checksumming: on
tx-checksumming: on
        tx-checksum-ipv4: on
        tx-checksum-ip-generic: off [fixed]
        tx-checksum-ipv6: on
        tx-checksum-fcoe-crc: off [fixed]
        tx-checksum-sctp: off [fixed]
scatter-gather: on
        tx-scatter-gather: on
        tx-scatter-gather-fraglist: off [fixed]
tcp-segmentation-offload: on
        tx-tcp-segmentation: on
        tx-tcp-ecn-segmentation: off [fixed]
        tx-tcp-mangleid-segmentation: off
        tx-tcp6-segmentation: on
generic-segmentation-offload: on
generic-receive-offload: on
large-receive-offload: on
rx-vlan-offload: on [fixed]
tx-vlan-offload: on [fixed]
ntuple-filters: off [fixed]
receive-hashing: on
highdma: on [fixed]
rx-vlan-filter: off [fixed]
vlan-challenged: off [fixed]
tx-lockless: off [fixed]
netns-local: off [fixed]
tx-gso-robust: off [fixed]
tx-fcoe-segmentation: off [fixed]
tx-gre-segmentation: off [fixed]
tx-gre-csum-segmentation: off [fixed]
tx-ipxip4-segmentation: off [fixed]
tx-ipxip6-segmentation: off [fixed]
tx-udp_tnl-segmentation: off [fixed]
tx-udp_tnl-csum-segmentation: off [fixed]
tx-gso-partial: off [fixed]
tx-tunnel-remcsum-segmentation: off [fixed]
tx-sctp-segmentation: off [fixed]
tx-esp-segmentation: off [fixed]
tx-udp-segmentation: off [fixed]
tx-gso-list: off [fixed]
fcoe-mtu: off [fixed]
tx-nocache-copy: off
loopback: off [fixed]
rx-fcs: off [fixed]
rx-all: off [fixed]
tx-vlan-stag-hw-insert: off [fixed]
rx-vlan-stag-hw-parse: off [fixed]
rx-vlan-stag-filter: off [fixed]
l2-fwd-offload: off [fixed]
hw-tc-offload: off [fixed]
esp-hw-offload: off [fixed]
esp-tx-csum-hw-offload: off [fixed]
rx-udp_tunnel-port-offload: off [fixed]
tls-hw-tx-offload: off [fixed]
tls-hw-rx-offload: off [fixed]
rx-gro-hw: off [fixed]
tls-hw-record: off [fixed]
rx-gro-list: off
macsec-hw-offload: off [fixed]
rx-udp-gro-forwarding: off
hsr-tag-ins-offload: off [fixed]
hsr-tag-rm-offload: off [fixed]
hsr-fwd-offload: off [fixed]
hsr-dup-offload: off [fixed]
```

Как можно видеть по строчкам:
```bash
tcp-segmentation-offload: on
        tx-tcp-segmentation: on
```
TCP Segmentation Offload  включен 

TCP Segmentation Offload переносит задачу сегментации TCP-пакетов с центрального процессора на сетевой адаптер.
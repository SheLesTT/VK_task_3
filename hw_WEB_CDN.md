## Задание 1_

Запуск сервера 1
```bash
denisden@LAPTOP-RMQ088BV:~/backend1$ Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
denisden@LAPTOP-RMQ088BV:~/backend1$ k192.168.100.1 - - [29/Oct/2025 14:14:20] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:20] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:50] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:52] "GET / HTTP/1.1" 200 -
denisden@LAPTOP-RMQ088BV:~/backend1$ cat index.html 
<h1>Response from Backend Server 1</h1>
denisden@LAPTOP-RMQ088BV:~/backend1$ 

```

Запуск сервера 2
```bash
denisden@LAPTOP-RMQ088BV:~/backend2$ python3 -m http.server 8082 &
[1] 6428
denisden@LAPTOP-RMQ088BV:~/backend2$ Serving HTTP on 0.0.0.0 port 8082 (http://0.0.0.0:8082/) ...
denisden@LAPTOP-RMQ088BV:~/backend2$ 192.168.100.1 - - [29/Oct/2025 14:14:20] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:20] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:50] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [29/Oct/2025 14:14:50] "GET / HTTP/1.1" 200 -
denisden@LAPTOP-RMQ088BV:~/backend2$ cat index.html 
<h2>*** Response from Backend Server 2 ***</h2>
```


## Задание 2
В конфигурации dnsmasq добавляем 2 записи 
address=/my-awesome-highload-app.local/127.0.0.1
address=/my-awesome-highload-app.local/127.0.0.2

Запускам dnsmasq
```bash
sudo systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/lib/systemd/system/dnsmasq.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-10-29 14:11:48 MSK; 26ms ago
    Process: 10781 ExecStartPre=/etc/init.d/dnsmasq checkconfig (code=exited, status=0/SUCCESS)
    Process: 10789 ExecStart=/etc/init.d/dnsmasq systemd-exec (code=exited, status=0/SUCCESS)
    Process: 10798 ExecStartPost=/etc/init.d/dnsmasq systemd-start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 10797 (dnsmasq)
      Tasks: 1 (limit: 5890)
     Memory: 844.0K
        CPU: 89ms
     CGroup: /system.slice/dnsmasq.service
             └─10797 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -7 /etc/dnsmasq.d,.dpkg-dist,.dpkg-old,.dpkg-new --local-service --trust-anchor=.,>

Oct 29 14:11:48 LAPTOP-RMQ088BV systemd[1]: Starting dnsmasq - A lightweight DHCP and caching DNS server...
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: started, version 2.90 cachesize 150
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: DNS service limited to local subnets
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipset no-nftset a>
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: reading /etc/resolv.conf
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: ignoring nameserver 127.0.0.1 - local interface
Oct 29 14:11:48 LAPTOP-RMQ088BV dnsmasq[10797]: read /etc/hosts - 13 names
Oct 29 14:11:48 LAPTOP-RMQ088BV systemd[1]: Started dnsmasq - A lightweight DHCP and caching DNS server.
```

При проверке чере dig dnsmasq возвращает оба Ip адреса.
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ dig @127.0.0.1 my-awesome-highload-app.local

; <<>> DiG 9.18.39-0ubuntu0.22.04.1-Ubuntu <<>> @127.0.0.1 my-awesome-highload-app.local
; (1 server found)
;; global options: +cmd
;; Got answer:_
;; WARNING: .local is reserved for Multicast DNS_
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40165
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;my-awesome-highload-app.local. IN      A

;; ANSWER SECTION:
my-awesome-highload-app.local. 0 IN     A       127.0.0.2
my-awesome-highload-app.local. 0 IN     A       127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Wed Oct 29 14:11:56 MSK 2025
;; MSG SIZE  rcvd: 90
```

Если сервер на  127.0.0.2 перестанет отвечать, dnsmasq этого не узначет, потому что DNS не отслеживает состояние серверов.

При резолве через DNS ответ останется тем же. Клиент может получить адрес неработающего backend, и запрос туда отвалиться по timeout.

## Задание 3

Создаем dummy интерфейс
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ ip addr show dummy1
3: dummy1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 1a:97:cb:4f:02:bc brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/32 scope global dummy1
       valid_lft forever preferred_lft forever
    inet6 fe80::1897:cbff:fe4f:2bc/64 scope link 
```
Создаем виртуалный сервер (VIP)
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo ipvsadm -A -t 192.168.100.1:80 -s rr

```

Добавляем два реальных сервера (REAL)
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8081 -m
denisden@LAPTOP-RMQ088BV:~/vk$ sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8082 -m

```
После 8 запросов на адрес видим, что нагрузка распределилась равномерно
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ curl http://192.168.100.1
```
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo ipvsadm -Ln --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.100.1:80                    8       52       46     3384     4296
  -> 127.0.0.1:8081                      4       25       23     1640     2132
  -> 127.0.0.1:8082                      4       27       23     1744     2164
```

## Задание 4

Конфигураци nginx
```bash

  1 http {
  2     upstream backend {
  3         server 127.0.0.1:8081 max_fails=7 fail_timeout=30s;
  4         server 127.0.0.1:8082 backup;
  5     }
  6 
  7     server {
  8         listen 8888;
  9         server_name 127.0.0.1;
 10 
 11         location / {
 12             proxy_pass http://backend;
 13             proxy_set_header X-high-load-test 123;
 14         }
 15     }
 16 }
~                                                                                                           
~          
```
Запуск tshark
```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo tshark -i lo -Y 'http' -O http
```

Трафик пойманный через tshark при активном сервер1

В пакете передаваемом от nginx до сервера есть заголовок  X-high-load-test: 
```bash
Frame 174: 144 bytes on wire (1152 bits), 144 bytes captured (1152 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 34550, Dst Port: 8888, Seq: 1, Ack: 1, Len: 78
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.1\r\n]
            [GET / HTTP/1.1\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.1
    Host: 127.0.0.1:8888\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://127.0.0.1:8888/]
    [HTTP request 1/1]

Frame 179: 184 bytes on wire (1472 bits), 184 bytes captured (1472 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 54318, Dst Port: 8081, Seq: 1, Ack: 1, Len: 118
Hypertext Transfer Protocol
    GET / HTTP/1.0\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.0\r\n]
            [GET / HTTP/1.0\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.0
    X-high-load-test: 123\r\n
    Host: backend_pool\r\n
    Connection: close\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://backend_pool/]
    [HTTP request 1/1]

Frame 183: 106 bytes on wire (848 bits), 106 bytes captured (848 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8081, Dst Port: 54318, Seq: 187, Ack: 119, Len: 40
[2 Reassembled TCP Segments (226 bytes): #181(186), #183(40)]
Hypertext Transfer Protocol
    HTTP/1.0 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.0 200 OK\r\n]
            [HTTP/1.0 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.0
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: SimpleHTTP/0.6 Python/3.10.12\r\n
    Date: Wed, 29 Oct 2025 16:06:21 GMT\r\n
    Content-type: text/html\r\n
    Content-Length: 40\r\n
        [Content length: 40]
    Last-Modified: Wed, 29 Oct 2025 11:02:38 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002151600 seconds]
    [Request in frame: 179]
    [Request URI: http://backend_pool/]
    File Data: 40 bytes
Line-based text data: text/html (1 lines)

Frame 187: 308 bytes on wire (2464 bits), 308 bytes captured (2464 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8888, Dst Port: 34550, Seq: 1, Ack: 79, Len: 242
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.1 200 OK\r\n]
            [HTTP/1.1 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.1
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: nginx/1.18.0 (Ubuntu)\r\n
    Date: Wed, 29 Oct 2025 16:06:21 GMT\r\n
    Content-Type: text/html\r\n
    Content-Length: 40\r\n
        [Content length: 40]
    Connection: keep-alive\r\n
    Last-Modified: Wed, 29 Oct 2025 11:02:38 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.003693900 seconds]
    [Request in frame: 174]
    [Request URI: http://127.0.0.1:8888/]
    File Data: 40 bytes
Line-based text data: text/html (1 lines)


```
Ответ приходит от сервера 1
```bash
denisden@LAPTOP-RMQ088BV:~$ curl -v http://127.0.0.1:8888/
*   Trying 127.0.0.1:8888...
* Connected to 127.0.0.1 (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8888
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Date: Wed, 29 Oct 2025 16:06:21 GMT
< Content-Type: text/html
< Content-Length: 40
< Connection: keep-alive
< Last-Modified: Wed, 29 Oct 2025 11:02:38 GMT
< 
<h1>Response from Backend Server 1</h1>
* Connection #0 to host 127.0.0.1 left intact
```

Останавливаем сервер 1
```bash
denisden@LAPTOP-RMQ088BV:~/backend1$ python3 -m http.server 8081 --bind 127.0.0.1 &
[1] 62832
denisden@LAPTOP-RMQ088BV:~/backend1$ Serving HTTP on 127.0.0.1 port 8081 (http://127.0.0.1:8081/) ...
127.0.0.1 - - [29/Oct/2025 18:58:58] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [29/Oct/2025 19:04:21] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [29/Oct/2025 19:04:44] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [29/Oct/2025 19:06:21] "GET / HTTP/1.0" 200 -
denisden@LAPTOP-RMQ088BV:~/backend1$ kill 62832
[1]+  Terminated              python3 -m http.server 8081 --bind 127.0.0.1
```

Ответ теперь приходит с сервера 2
```bash
denisden@LAPTOP-RMQ088BV:~$ curl -v http://127.0.0.1:8888/
*   Trying 127.0.0.1:8888...
* Connected to 127.0.0.1 (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8888
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Date: Wed, 29 Oct 2025 16:07:54 GMT
< Content-Type: text/html
< Content-Length: 48
< Connection: keep-alive
< Last-Modified: Wed, 29 Oct 2025 11:02:47 GMT
< 
<h2>*** Response from Backend Server 2 ***</h2>
* Connection #0 to host 127.0.0.1 left intact
```
Трафик соотвествующий запросу

```bash
Frame 1406: 144 bytes on wire (1152 bits), 144 bytes captured (1152 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 49152, Dst Port: 8888, Seq: 1, Ack: 1, Len: 78
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.1\r\n]
            [GET / HTTP/1.1\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.1
    Host: 127.0.0.1:8888\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://127.0.0.1:8888/]
    [HTTP request 1/1]

Frame 1413: 184 bytes on wire (1472 bits), 184 bytes captured (1472 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 32952, Dst Port: 8082, Seq: 1, Ack: 1, Len: 118
Hypertext Transfer Protocol
    GET / HTTP/1.0\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.0\r\n]
            [GET / HTTP/1.0\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.0
    X-high-load-test: 123\r\n
    Host: backend_pool\r\n
    Connection: close\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://backend_pool/]
    [HTTP request 1/1]

Frame 1417: 114 bytes on wire (912 bits), 114 bytes captured (912 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8082, Dst Port: 32952, Seq: 187, Ack: 119, Len: 48
[2 Reassembled TCP Segments (234 bytes): #1415(186), #1417(48)]
Hypertext Transfer Protocol
    HTTP/1.0 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.0 200 OK\r\n]
            [HTTP/1.0 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.0
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: SimpleHTTP/0.6 Python/3.10.12\r\n
    Date: Wed, 29 Oct 2025 16:07:54 GMT\r\n
    Content-type: text/html\r\n
    Content-Length: 48\r\n
        [Content length: 48]
    Last-Modified: Wed, 29 Oct 2025 11:02:47 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002701800 seconds]
    [Request in frame: 1413]
    [Request URI: http://backend_pool/]
    File Data: 48 bytes
Line-based text data: text/html (1 lines)

Frame 1420: 316 bytes on wire (2528 bits), 316 bytes captured (2528 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8888, Dst Port: 49152, Seq: 1, Ack: 79, Len: 250
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.1 200 OK\r\n]
            [HTTP/1.1 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.1
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: nginx/1.18.0 (Ubuntu)\r\n
    Date: Wed, 29 Oct 2025 16:07:54 GMT\r\n
    Content-Type: text/html\r\n
    Content-Length: 48\r\n
        [Content length: 48]
    Connection: keep-alive\r\n
    Last-Modified: Wed, 29 Oct 2025 11:02:47 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.004768100 seconds]
    [Request in frame: 1406]
    [Request URI: http://127.0.0.1:8888/]
    File Data: 48 bytes
Line-based text data: text/html (1 lines)


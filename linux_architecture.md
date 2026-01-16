

##Задание 1

версия ядра:  6.6.87.2
```bash
root@ASUSZB:~# uname -a
Linux ASUSZB 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

Список всех загруженных модулей ядра:
```bash
root@ASUSZB:~/VK_task_3# lsmod
Module                  Size  Used by
btrfs                1581056  0
blake2b_generic        16384  0
xor                    20480  1 btrfs
raid6_pq              110592  1 btrfs
msdos                  16384  0
cfg80211              958464  0
intel_rapl_msr         16384  0
intel_rapl_common      32768  1 intel_rapl_msr
kvm_intel             356352  0
crc32c_intel           16384  0
ac                     16384  0
battery                20480  0
kvm                   970752  1 kvm_intel
irqbypass              12288  1 kvm
sch_fq_codel           16384  1
configfs               53248  0
autofs4                45056  0
br_netfilter           28672  0
bridge                282624  1 br_netfilter
stp                    12288  1 bridge
llc                    12288  2 bridge,stp
ip_tables              28672  0
tun                    53248  0
```



Создадим файл конфигурации blacklist
```bash
vi /etc/modprobe.d/blacklist-cdrom.conf

```
с единственной строкой:
```bash
blacklist cdrom

```
Обновим initramfs
```bash
root@ASUSZB:~/VK_task_3# update-initramfs -u
```




Поддержка файловой системы XFS встроена в ядро

```bash
root@ASUSZB:~/VK_task_3# zcat /proc/config.gz | grep CONFIG_XFS_FS
CONFIG_XFS_FS=y
```


##Задание 2

В логах трассировки видно, что есть один вызов write, записывающий содержание файла /etc/os-release в стандартный вывод.

/etc/os-release файл в Linux, который содержит информацию о об операционной системы: название, версию, id..
```bash
root@ASUSZB:~/VK_task_3#  strace -e trace=openat,read,write,close cat /etc/os-release > /dev/null
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/locale.alias", O_RDONLY|O_CLOEXEC) = 3
read(3, "# Locale name alias data base.\n#"..., 4096) = 2996
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NAME", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_NAME", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_PAPER", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_PAPER", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TIME", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_TIME", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3
read(3, "PRETTY_NAME=\"Ubuntu 22.04.5 LTS\""..., 131072) = 386
write(1, "PRETTY_NAME=\"Ubuntu 22.04.5 LTS\""..., 386) = 386
read(3, "", 131072)                     = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
+++ exited with 0 +++

```
##Задание 4

Извлекаем из /proc модель CPU и объём памяти (KiB).
```bash
root@ASUSZB:~/VK_task_3# grep "model name" /proc/cpuinfo
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
model name      : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
```
```bash
root@ASUSZB:~/VK_task_3# grep "MemTotal" /proc/meminfo
MemTotal:        7975136 kBgrep "MemTotal" /proc/meminfo

```

$$ - означает process_id текущего shell
```bash
root@ASUSZB:~/VK_task_3# grep PPid /proc/$$/status
PPid:   479
```
Определим настройки I/O scheduler
```bash
root@ASUSZB:~/VK_task_3# cat /sys/block/sda/queue/scheduler
[none] mq-deadline kyber

```
Видим что планировщик не выбран. Так как задание выполянется на wsl, и доступ к диску происхдит через файловую систему windows


Определим основной интерфейс
```bash
ip link
root@ASUSZB:~/VK_task_3# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:50:56:c0:00:01 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:55:66:95 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:98:46:08 brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:50:56:c0:00:08 brd ff:ff:ff:ff:ff:ff
6: loopback0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:be:6c:a5 brd ff:ff:ff:ff:ff:ff
7: eth4: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:58:bc:71 brd ff:ff:ff:ff:ff:ff
8: eth5: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:e1:78:d2 brd ff:ff:ff:ff:ff:ff
9: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether f0:57:a6:df:43:f0 brd ff:ff:ff:ff:ff:ff
13: eth7: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:00:27:00:00:58 brd ff:ff:ff:ff:ff:ff
14: eth8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:2c:c8:2e brd ff:ff:ff:ff:ff:ff
root@ASUSZB:~/VK_task_3# ip addr show eth6

```

видим, что кроме внутренних loopback интерфейсов запущены два eth6 и eth8


eth8 имеет ip адрес докер подсети и mtu 9000. eth6 имеет ip локальной сети и mtu 1500
```bash
root@ASUSZB:~/VK_task_3# ip addr show eth6
ip addr show eth8
9: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether f0:57:a6:df:43:f0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.113/24 brd 192.168.1.255 scope global noprefixroute eth6
       valid_lft forever preferred_lft forever
    inet6 fde1:4093:4380:0:e020:bad8:789c:8037/128 scope global nodad noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fde1:4093:4380:0:5ca5:7983:4515:e228/64 scope global nodad deprecated noprefixroute
       valid_lft forever preferred_lft 0sec
    inet6 fe80::3a48:1463:e80c:620f/64 scope link nodad noprefixroute
       valid_lft forever preferred_lft forever
14: eth8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:2c:c8:2e brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/28 brd 172.19.0.15 scope global noprefixroute eth8
       valid_lft forever preferred_lft forever
```


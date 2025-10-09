## Задание 1
systemd unit 

```bash
[Unit]
Description=My Persistent Script Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/homework_service.sh
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target                    
```

#### Сервис самостоятельно запускается после остановки

```bash
denisden@LAPTOP-RMQ088BV:~/vk$ sudo systemctl kill homework.service
denisden@LAPTOP-RMQ088BV:~/vk$ sudo systemctl status homework.service
● homework.service - My Persistent Script Service
     Loaded: loaded (/etc/systemd/system/homework.service; disabled; vendor preset: enabled)
     Active: activating (auto-restart) since Tue 2025-10-07 11:39:30 MSK; 3s ago
    Process: 354361 ExecStart=/usr/local/bin/homework_service.sh (code=killed, signal=TERM)
   Main PID: 354361 (code=killed, signal=TERM)
        CPU: 109ms
denisden@LAPTOP-RMQ088BV:~/vk$ sudo systemctl status homework.service
● homework.service - My Persistent Script Service
     Loaded: loaded (/etc/systemd/system/homework.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-10-07 11:39:45 MSK; 6s ago
   Main PID: 354721 (homework_servic)
      Tasks: 2 (limit: 5890)
     Memory: 620.0K
        CPU: 18ms
     CGroup: /system.slice/homework.service
             ├─354721 /bin/bash /usr/local/bin/homework_service.sh
             └─354723 sleep 15

Oct 07 11:39:45 LAPTOP-RMQ088BV systemd[1]: Started My Persistent Script Service.
Oct 07 11:39:45 LAPTOP-RMQ088BV homework_service.sh[354721]: My custom service has started.
```
#### топ-5 systemd unit`ов стартующих дольше всего.

```bash
denisden@LAPTOP-RMQ088BV:/tmp$ systemd-analyze blame | head -n 5
4min 23.445s apt-daily.service
     25.575s motd-news.service
     10.088s man-db.service
      5.121s logrotate.service
      4.270s apt-daily-upgrade.service
```


## Задание 2
```bash
denisden@LAPTOP-RMQ088BV:~/vk/task_3$ ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      
0x0000b0c6 0          postgres   600        56         6                       
0x4130f375 1          denisden   666        1024       0

Сегмент создан с ключом 0x4130f375, но ещё никто его не использует (nattch=0). Права разрешают писать и читать любому.
```

## Задание 3
```bash
denisden@LAPTOP-RMQ088BV:~$ ps -o pid,user,%mem,rss,vsz,comm -p 410635
    PID USER     %MEM   RSS    VSZ COMMAND
 410635 denisden  5.2 263552 269552 python3
```

RSS ~250Mb так как python алоцирует память для строки размером 250 Mb и пишет туда данные. Следовательно эти данные должны появиться непосредственно в оперативной памяти. VSZ это общий размер зарезервированного виртуального пространства процесса. Часть этой памяти может быть зарезервирована, но еще никак не использована. Но VSZ включает в себя и все данные RSS. Так как RSS это та часть виртуального пространства, которая сейчас лежит в оперативной памяти. 


## Задание 4
#### Показать количество NUMA нод

numactl --hardware

```bash
denisden@LAPTOP-RMQ088BV:~$ numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7
node 0 size: 4918 MB
node 0 free: 1275 MB
node distances:
node   0
  0:  10
```

#### Детальное описание памяти на кажной ноде

```bash
denisden@LAPTOP-RMQ088BV:~$ numastat -m

Per-node system memory usage (in MBs):
Token SwapCached not in hash table.
Token SecPageTables not in hash table.
Token FileHugePages not in hash table.
Token FilePmdMapped not in hash table.
Token Unaccepted not in hash table.
                          Node 0           Total
                 --------------- ---------------
MemTotal                 4918.22         4918.22
MemFree                  1278.48         1278.48
MemUsed                  3639.74         3639.74
Active                    152.97          152.97
Inactive                 3023.75         3023.75
Active(anon)               36.54           36.54
Inactive(anon)           2938.76         2938.76
Active(file)              116.44          116.44
Inactive(file)             84.99           84.99
Unevictable                 0.00            0.00
Mlocked                     0.00            0.00
Dirty                       0.13            0.13
Writeback                   0.00            0.00
FilePages                 242.62          242.62
Mapped                    187.96          187.96
AnonPages                2938.69         2938.69
Shmem                       8.24            8.24
KernelStack                 7.62            7.62
PageTables                 47.52           47.52
NFS_Unstable                0.00            0.00
Bounce                      0.00            0.00
WritebackTmp                0.00            0.00
Slab                      164.85          164.85
SReclaimable               90.52           90.52
SUnreclaim                 74.33           74.33
AnonHugePages               0.00            0.00
ShmemHugePages              0.00            0.00
ShmemPmdMapped              0.00            0.00
HugePages_Total             0.00            0.00
HugePages_Free              0.00            0.00
HugePages_Surp              0.00            0.00
KReclaimable               90.52           90.52
```

#### При запуске теста, использование памяти никогда не превыет 150 MB

```bash
sudo systemd-run --unit=highload-stress-test --slice=testing.slice \

--property="MemoryMax=150M" \

--property="CPUWeight=100" \

stress --cpu 1 --vm 1 --vm-bytes 300M --timeout 30s
```
```bash
Control Group                                                                Tasks   %CPU   Memory  Input/s Output/s
/                                                                                -  168.9        -    63.7M    70.7M
testing.slice                                                                    3  150.1   149.9M        -        -
testing.slice/highload-stress-test.service                                       3  150.1   149.8M        -        -
init.scope                                                                       1    1.2    10.1M        -        -
```

MemoryMax - yстанавливает абсолютный лимит памяти для службы/процесса
используется для предотвращение исчерпания памяти одним процессом

CPUWeight - определяет относительный приоритет CPU между службами (от 1 до 10000)

используется для :

- Распределение CPU времени между конкурирующими процессами
- Приоритизация важных служб
- Балансировка нагрузки 

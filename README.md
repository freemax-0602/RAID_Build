**Домашнее задание №3**

**Тема** ***"Дисковая подсистема"***

**Задача**
Подготовить научиться использовать утилиту для управления программными RAID-массивами в Linux:

- добавить в Vagrantfile еще дисков;
- собрать R0/R5/R10 на выбор;
- сломать/починить raid;
- прописать собранный рейд в конф, чтобы рейд собирался при загрузке;
- создать GPT раздел и 5 партиций.
---
**Результат выполнения задания**
1. Создан `Vagrantfile`. В `Vagrantfile` добавлены еще диски:
```
:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 250, # Megabytes
			:port => 2
		},
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 250,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
                }

	}
```
Вывод команды `fdisk -l` на хосте:
```
vagrant@raids ~]$ sudo fdisk -l

Disk /dev/sda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ef1a

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    83886079    41942016   83  Linux

Disk /dev/sde: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

2. Создана роль ansible:
```
├── playbooks
│   └── raids.yml
├── raids
│   ├── defaults
│   │   └── main.yml
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── staging
│   │   └── hosts
│   │       └── inventory
│   ├── tasks
│   │   ├── build.yml
│   │   ├── install_debian.yml
│   │   ├── install_RedHat.yml
│   │   └── main.yml
│   ├── templates
│   │   └── etc
│   │       └── mdadm
│   │           └── mdadm.conf.j2
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
├── README.md
├── sata1.vdi
├── sata2.vdi
├── sata3.vdi
├── sata4.vdi
└── Vagrantfile

```

3. Роль выполняет следующие функции:
- Создаёт RAID-массив
- Создает конфигурационный файл mdadm.conf, для того, чтобы RAID-массив собирался после перезагрузки
- Эмулирует "поломку" RAID-массива
- Чинит RAID-массив, путём замены "сломанного" диска
- Создает таблицу разделов gpt
- Создает 5 партиций, устанавливает на них файловую систему и монтирует в дирректории

4. В результате выполнения playbook: 
``` 
ansible-playbook playbooks/raids.yml                                                                                        ─╯

PLAY [raids] *********************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************
ok: [raids]

TASK [raids : include_tasks] *****************************************************************************************************
skipping: [raids]

TASK [raids : include_tasks] *****************************************************************************************************
included: /home/admsys/study/RAID_Build/raids/tasks/install_RedHat.yml for raids

TASK [raids : RedHat | Installing mdadm] *****************************************************************************************
ok: [raids]

TASK [raids : arrays | Setting distribution-specific facts] **********************************************************************
ok: [raids]

TASK [raids : include_tasks] *****************************************************************************************************
included: /home/admsys/study/RAID_Build/raids/tasks/build.yml for raids

TASK [raids : arrays | Checking Status Of Array(s)] ******************************************************************************
ok: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Creating Array(s)] ****************************************************************************************
changed: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Capturing Array Details] **********************************************************************************
ok: [raids]

TASK [raids : arrays | Create mdadm] *********************************************************************************************
changed: [raids]

TASK [raids : arrays | Add /etc/mdadm/mdadm.conf] ********************************************************************************
changed: [raids]

TASK [raids : arrays | Updating /etc/mdadm/mdadm.conf] ***************************************************************************
changed: [raids] => (item=ARRAY /dev/md0 metadata=1.2 spares=1 name=raids:0 UUID=12185378:1c1ae2f6:27892bdc:40c5b13e)

TASK [raids : arrays | Degradet Array] *******************************************************************************************
changed: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Remove disk on Array] *************************************************************************************
changed: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Add disk to Array(s)] *************************************************************************************
changed: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Updating /etc/mdadm/mdadm.conf] ***************************************************************************
ok: [raids] => (item=ARRAY /dev/md0 metadata=1.2 spares=1 name=raids:0 UUID=12185378:1c1ae2f6:27892bdc:40c5b13e)

TASK [raids : arrays | Create gpt partition] *************************************************************************************
changed: [raids] => (item={'name': 'md0', 'devices': ['/dev/sdb', '/dev/sdc', '/dev/sdd', '/dev/sde'], 'filesystem': 'ext4', 'level': '5', 'mountpoint': '/mnt/md0', 'state': 'present', 'opts': 'noatime', 'files': '/var/log', 'number': '1', 'table': ['0% 20%', '20% 40%', '40% 60%', '60% 80%', '80% 100%'], 'name_partition': ['1', '2', '3', '4', '5']})

TASK [raids : arrays | Create a new ext4 primary partition] **********************************************************************
changed: [raids] => (item=0% 20%)
changed: [raids] => (item=20% 40%)
changed: [raids] => (item=40% 60%)
changed: [raids] => (item=60% 80%)
changed: [raids] => (item=80% 100%)

TASK [raids : arrays | Create partition dir] *************************************************************************************
changed: [raids] => (item=1)
changed: [raids] => (item=2)
changed: [raids] => (item=3)
changed: [raids] => (item=4)
changed: [raids] => (item=5)

TASK [raids : arrays | Install filesystem on partition] **************************************************************************
changed: [raids] => (item=1)
changed: [raids] => (item=2)
changed: [raids] => (item=3)
changed: [raids] => (item=4)
changed: [raids] => (item=5)

TASK [raids : arrays | Mounting partition] ***************************************************************************************
changed: [raids] => (item=1)
changed: [raids] => (item=2)
changed: [raids] => (item=3)
changed: [raids] => (item=4)
changed: [raids] => (item=5)
```

Хост raids **до запуска** Playbook:
```
Last login: Sun Feb 25 22:13:21 2024 from 10.0.2.2
[vagrant@raids ~]$ sudo lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk 
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  250M  0 disk 
sdc      8:32   0  250M  0 disk 
sdd      8:48   0  250M  0 disk 
sde      8:64   0  250M  0 disk 
[vagrant@raids ~]$ cat /proc/mdstat 
Personalities : 
unused devices: <none>
[vagrant@raids ~]$ 
```

Хост **после выполнения** Playbook:
```
[vagrant@raids ~]$ sudo cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sde[3] sdc[1] sdb[0]
      507904 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
[vagrant@raids ~]$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Feb 25 22:49:39 2024
        Raid Level : raid5
        Array Size : 507904 (496.00 MiB 520.09 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun Feb 25 22:51:13 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : raids:0  (local to host raids)
              UUID : 4a75f10b:b69ef4ca:47dc8899:f73e5ac5
            Events : 115

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       3       8       64        2      active sync   /dev/sde
[vagrant@raids ~]$ sudo lsblk                  
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0   40G  0 disk  
`-sda1      8:1    0   40G  0 part  /
sdb         8:16   0  250M  0 disk  
`-md0       9:0    0  496M  0 raid5 
  |-md0p1 259:0    0   98M  0 md    /raid/part1
  |-md0p2 259:1    0   99M  0 md    /raid/part2
  |-md0p3 259:2    0  100M  0 md    /raid/part3
  |-md0p4 259:3    0   99M  0 md    /raid/part4
  `-md0p5 259:4    0   98M  0 md    /raid/part5
sdc         8:32   0  250M  0 disk  
`-md0       9:0    0  496M  0 raid5 
  |-md0p1 259:0    0   98M  0 md    /raid/part1
  |-md0p2 259:1    0   99M  0 md    /raid/part2
  |-md0p3 259:2    0  100M  0 md    /raid/part3
  |-md0p4 259:3    0   99M  0 md    /raid/part4
  `-md0p5 259:4    0   98M  0 md    /raid/part5
sdd         8:48   0  250M  0 disk  
sde         8:64   0  250M  0 disk  
`-md0       9:0    0  496M  0 raid5 
  |-md0p1 259:0    0   98M  0 md    /raid/part1
  |-md0p2 259:1    0   99M  0 md    /raid/part2
  |-md0p3 259:2    0  100M  0 md    /raid/part3
  |-md0p4 259:3    0   99M  0 md    /raid/part4
  `-md0p5 259:4    0   98M  0 md    /raid/part5
```
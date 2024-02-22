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
── ansible.cfg
├── playbooks
│   └── raids.yml
├── raids
│   ├── defaults
│   │   └── main.yml
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── staging
│   │   └── hosts
│   │       └── inventory
│   ├── tasks
│   │   └── main.yml
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
├── README.md
├── sata1.vdi
├── sata2.vdi
├── sata3.vdi
├── sata4.vdi
└── Vagrantfile

```

3. Функции Raids:
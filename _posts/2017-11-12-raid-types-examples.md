---
layout: post
title: RAID 실습해보기
tags: [linux, raid]
---

[이전 포스트]({{ site.url }}/raid-types) 에서 RAID 구성에 대해 이론만 확인했었는데, 실제 만들어보면 더 잘 이해가 된다. 한 번 직접 해보자.

---

* 목차
{:toc}

---

## RAID 1 위에 OS 설치하기

OS 를 구축하는데, 보통 OS 의 파일들이 서비스 구성 상 가장 중요하다. OS 가 날라가면 그 위의 SW 는 아무 것도 못하기 때문이다. 그래서 2개의 디스크로 RAID 1 를 구성한 뒤, 이 위에 OS 를 설치해서 OS 데이터를 2중화한다.

그런데 보통은 Linux OS 가 설치된 이후 mdadm 명령어를 통해 RAID 를 구성하는데, 그 전에는 어떻게 해야할까? 그래서 CentOS 는 설치 시에 디스크 구성 옵션에서 이런 부분들을 설정할 수가 있다.

아래 순서대로 따라해보면서 RAID 1 디스크 위에 OS 를 설치해보자.

### 설치 환경 구축하기

설치 실습은 Virtual Box 위에 CentOS 7 을 통해 수행해보자. Virtual Box 는 [여기](https://www.virtualbox.org/wiki/Downloads)서, CentOS 는 [여기](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso)서 다운로드 받고, Virtual Box 는 설치해두자.

### CentOS 설치하기

Virtual Box 에서 New 버튼을 눌러서 Redhat Linux 를 선택하자. (CentOS 는 Redhat 과 동일하기 때문에) 그리고 스토리지 옵션에 들어가서 아래와 같이 디스크 2개를 만들어서 넣고, CD-ROM 에는 위에서 다운받은 CentOS 의 iso 이미지를 넣자.

![OS Install #1]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-01.png){: .center-image}

그리고 시작 버튼을 누르면 설치를 시작한다.

![OS Install #2]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-02.png){: .center-image}

Install CentOS 7 을 클릭하고, Installation Destination 을 빼고 나머지 옵션들은 적당히 선택한다.

![OS Install #3]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-03.png){: .center-image}

그리고 Installation Destination 을 선택하고, 우리가 꼽은 디스크 2개가 보이는데 모두 선택한다. 그리고 Partitioning 은 I will configure partitioning. 을 선택하자.

![OS Install #4]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-04.png){: .center-image}

다음 화면에서는 아래와 같이 Click here to create them automatically 를 선택한다. 마운트 포인트를 여기에 있는 템플릿대로 하겠다는 뜻이다.

![OS Install #5]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-05.png){: .center-image}

그러면 마운트포인트 /boot, /, swap 영역에 대해 파티션 옵션을 지정할 수 있다. /boot 와 / 는 중요한 데이터들이 들어가므로 RAID 1 로 구성하고, swap 영역은 중요한 데이터가 아니기 때문에 RAID 0 으로 아래처럼 구성해보자.

![OS Install #6]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-06.png){: .center-image}

![OS Install #7]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-07.png){: .center-image}

![OS Install #8]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-08.png){: .center-image}

그리고 확인 버튼을 누르면 아래와 같이 디스크 파티셔닝을 하겠다는 확인 창이 뜬다.

![OS Install #9]({{ site.url }}/images/2017-11-12-raid-types-examples/os-install-09.png){: .center-image}

여기서 중요한 것들을 확인할 수 있다. 우선 Linux 에서 SATA 나 SAS 로 디스크를 꼽으면 /dev 안에 sda, sdb, sdc .. 와 같은 이름으로 표시된다. 이 이름들은 물리디스크 자체를 말하는 것이다.

그리고 이 물리디스크를 파티션으로 나누면 OS 가 이 파티션을 1개의 쓰기영역으로 인식한다. sda 물리디스크를 1개로 파티셔닝하면 /dev 안에 sda1 라는 1개의 파티션이 생긴다. 2개로 파티셔닝하면 sda1, sda2 라는 2개의 파티션이 생긴다. 그리고 이 파티션들끼리 RAID 를 구성하는 것이다.

위의 그림에서보면 우리가 꼽은 2개의 물리디스크가 sda, sdb 라는 이름으로 표시된다. 그리고 이 디스크를 포맷하고 난뒤, 각 디스크를 3개의 파티션으로 파티셔닝하는 것이 보인다. 그래서 총 6개의 파티션(sda1, sda2, sda3, sdb1, sdb2, sdb3)이 생긴다.

그리고 아마도 sda1, sda2 를 RAID 1 로 묶어서 /boot 를 만드는 식으로 RAID 가 구성될 것이다.

여튼 여기서 일어나는 작업은 대충 이렇고, 자세히는 확인 버튼을 눌러서 OS 설치를 마무리하고 확인해보자.

### OS 의 RAID 구성 확인하기

부팅 후 파일시스템과 Swap 영역을 확인해보자.

```
[root@raid ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/md127      5.8G  885M  4.6G  16% /
devtmpfs        2.0G     0  2.0G   0% /dev
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           2.0G  8.4M  2.0G   1% /run
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/md126      976M   91M  819M  10% /boot
tmpfs           396M     0  396M   0% /run/user/0

[root@raid ~]# swapon -s
Filename				Type		Size	Used	Priority
/dev/md125                             	partition	2099196	0	-1
```

/ 는 /dev/md127 이라는 파일시스템으로, /boot 는 /dev/md126 으로, Swap 은 /dev/md125 로 구성되어 있다.

Linux 에서 RAID 를 구성하면 /dev 안에 md 라는 Prefix 가 붙은 파일로 파일시스템이 구성된다. 즉, 이것들은 RAID 구성한 디스크그룹이 할당되었다는 뜻이다.

그럼 이 md125, md126, md127 이 우리가 처음 설정한대로 잘 구성되었는지 확인해보자.

우선 디스크와 파티션들은 아래와 같다.

```
[root@raid ~]# ls -al /dev/sd*
brw-rw----. 1 root disk 8,  0 Nov 12 23:09 /dev/sda
brw-rw----. 1 root disk 8,  1 Nov 12 23:09 /dev/sda1
brw-rw----. 1 root disk 8,  2 Nov 12 23:09 /dev/sda2
brw-rw----. 1 root disk 8,  3 Nov 12 23:09 /dev/sda3
brw-rw----. 1 root disk 8, 16 Nov 12 23:09 /dev/sdb
brw-rw----. 1 root disk 8, 17 Nov 12 23:09 /dev/sdb1
brw-rw----. 1 root disk 8, 18 Nov 12 23:09 /dev/sdb2
brw-rw----. 1 root disk 8, 19 Nov 12 23:09 /dev/sdb3
```

RAID 들은 아래와 같다.

> RAID 를 구성하면 /dev/md/ 디렉토리 안에 매핑된 것들이 나오고, 이것들은 /dev/md* 로 link 되어있는 것을 알 수 있다.

```
[root@raid ~]# ls -al /dev/md*
brw-rw----. 1 root disk 9, 125 Nov 12 23:09 /dev/md125
brw-rw----. 1 root disk 9, 126 Nov 12 23:09 /dev/md126
brw-rw----. 1 root disk 9, 127 Nov 12 23:09 /dev/md127

/dev/md:
total 0
drwxr-xr-x.  2 root root  100 Nov 12 23:09 .
drwxr-xr-x. 19 root root 3200 Nov 12 23:09 ..
lrwxrwxrwx.  1 root root    8 Nov 12 23:09 boot -> ../md126
lrwxrwxrwx.  1 root root    8 Nov 12 23:09 root -> ../md127
lrwxrwxrwx.  1 root root    8 Nov 12 23:09 swap -> ../md125
```

이제 RAID 구성을 확인해보자. 아래와 같이 2개의 디스크들을 통해서 RAID 0, RAID 1 이 구성되었다는 것을 알 수 있다.

```
[root@raid ~]# mdadm --detail /dev/md125
/dev/md125:
           Version : 1.2

   (중략)

    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sda1
       1       8       17        1      active sync   /dev/sdb1

[root@raid ~]# mdadm --detail /dev/md126
/dev/md126:
           Version : 1.2

   (중략)

    Number   Major   Minor   RaidDevice State
       0       8        2        0      active sync   /dev/sda2
       1       8       18        1      active sync   /dev/sdb2

[root@raid ~]# mdadm --detail /dev/md127
/dev/md127:
           Version : 1.2

   (중략)

    Number   Major   Minor   RaidDevice State
       0       8        3        0      active sync   /dev/sda3
       1       8       19        1      active sync   /dev/sdb3
```

즉, 우리는 디스크 중 1개가 고장나더라도 / 와 /boot 에는 영향이 없는 OS 를 설치하였다.

## RAID 5 구성하기

이제 우리가 위에서 설치한 OS 위에 명령어를 통해 직접 RAID 5 를 구성해보자. 이전 글에서 설명했다시피 아래와 같은 구성이다.

![RAID 5]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5.png){: .center-image}

설치 전에 먼저 생각해보자.

- 디스크 1개당 크기가 2 GiB 라면 RAID 후에 가용크기는 얼마가 되어있을까?
- 디스크 몇 개 까지 오류를 견딜 수 있을까?
- 디스크 1개당 I/O 쓰기속도가 1 GiB/s 라면 이론상 쓰기속도는 얼마일까?

### 디스크 추가하기

Virtual Box 의 Guest OS 를 종료하고, 아래와 같이 2GiB 크기로 3개의 디스크를 추가해보자.

![RAID 5 Install #1]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-install-01.png){: .center-image}

그리고 OS 를 시작하자.

### 디스크 확인 및 파티셔닝 하기

나는 스토리지 추가 시에 NVMe 컨트롤러에 디스크를 추가해서 디스크 파일들이 /dev/nvme0n1, /dev/nvme0n2, /dev/nvme0n3 으로 표시된다. SATA 나 SAS 컨트롤러에 추가했다면 /dev/sdb, /dev/sdc, /dev/sdd 로 나타날 것이다. 파일명만 다를 뿐이므로 아래 작업들은 모두 파일명만 바꾸어서 동일하게 진행하면 된다.

우선 디스크들이 모두 정상적으로 꼽혀서 인식되는지 확인하자.

```
[root@raid ~]# ls -al /dev/nvme*
crw-------. 1 root root 247, 0 Nov 13 00:19 /dev/nvme0
brw-rw----. 1 root disk 259, 0 Nov 13 00:19 /dev/nvme0n1
brw-rw----. 1 root disk 259, 1 Nov 13 00:19 /dev/nvme0n2
brw-rw----. 1 root disk 259, 2 Nov 13 00:19 /dev/nvme0n3
```

3개 모두 정상적으로 보인다.

우리는 1개 물리 디스크를 1개의 파티션으로 만들어서 총 3개의 파티션을 만들 것이다. 아래 작업을 각 물리 디스크별로 수행하자.

```
[root@raid ~]# fdisk /dev/nvme0n1
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x27a304ad.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-4194303, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
Using default value 4194303
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT
1e  Hidden W95 FAT1 80  Old Minix
Hex code (type L to list all codes): fd
Changed type of partition 'Linux' to 'Linux raid autodetect'

Command (m for help): p

Disk /dev/nvme0n1: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x27a304ad

        Device Boot      Start         End      Blocks   Id  System
/dev/nvme0n1p1            2048     4194303     2096128   fd  Linux raid autodetect

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

파티셔닝 시에 System 을 Linux raid autodetect 로 설정해야한다. 나머지 2개 디스크도 위와 동일하게 진행하고 난 뒤, 다시 파티션 파일들을 확인하자.

```
[root@raid ~]# ls -al /dev/nvme*
crw-------. 1 root root 247, 0 Nov 13 00:19 /dev/nvme0
brw-rw----. 1 root disk 259, 0 Nov 13 00:25 /dev/nvme0n1
brw-rw----. 1 root disk 259, 3 Nov 13 00:25 /dev/nvme0n1p1
brw-rw----. 1 root disk 259, 1 Nov 13 00:27 /dev/nvme0n2
brw-rw----. 1 root disk 259, 4 Nov 13 00:27 /dev/nvme0n2p1
brw-rw----. 1 root disk 259, 2 Nov 13 00:27 /dev/nvme0n3
brw-rw----. 1 root disk 259, 5 Nov 13 00:27 /dev/nvme0n3p1
```

NVMe 컨트롤러의 디스크는 파티셔닝하면 뒤에 p1, p2 .. 이런 식으로 붙는다. SATA 나 SAS 컨트롤러의 디스크들은 1, 2 .. 이런 식으로 Postfix 가 붙는다.

3개 모두 파티셔닝이 잘 된 것을 확인할 수 있다.

### RAID 구성 및 확인하기

이제 3개의 파티셔닝된 디스크들로부터 RAID 5 를 구성해보자. 사실 명령어를 통해서 level 옵션에 넣는 숫자만 달라질 뿐, 모든 RAID 구성의 명령어는 동일하다.

```
[root@raid ~]# mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/nvme0n1p1 /dev/nvme0n2p1 /dev/nvme0n3p1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md5 started.
```

위의 명령어는 3개의 파티션 디스크 (/dev/nvme0n1p1, /dev/nvme0n2p1, /dev/nvme0n3p1) 를 사용해서 /dev/md5 라는 파일명으로 RAID 를 구성하라고 하는 것이다. 이제 OS 는 /dev/md5 를 하나의 디스크로 인식한다.

> md5 라는 이름은 내 마음대로 정할 수 있다. 보통 md 는 RAID 구성한 파일의 네이밍으로서 앞에 붙여주고, 뒤에는 내가 인식하기 쉬운 이름으로 붙이면 된다.

RAID 파일이 잘 생성되었는지 확인하자.

```
[root@raid ~]# ls -al /dev/md*
brw-rw----. 1 root disk 9, 125 Nov 13 00:19 /dev/md125
brw-rw----. 1 root disk 9, 126 Nov 13 00:19 /dev/md126
brw-rw----. 1 root disk 9, 127 Nov 13 00:19 /dev/md127
brw-rw----. 1 root disk 9,   5 Nov 13 00:30 /dev/md5

/dev/md:
total 0
drwxr-xr-x.  2 root root  100 Nov 13 00:19 .
drwxr-xr-x. 19 root root 3360 Nov 13 00:30 ..
lrwxrwxrwx.  1 root root    8 Nov 13 00:19 boot -> ../md127
lrwxrwxrwx.  1 root root    8 Nov 13 00:19 root -> ../md126
lrwxrwxrwx.  1 root root    8 Nov 13 00:19 swap -> ../md125
```

```
[root@raid ~]# mdadm --detail /dev/md5
/dev/md5:
           Version : 1.2
     Creation Time : Mon Nov 13 00:30:53 2017
        Raid Level : raid5
        Array Size : 4190208 (4.00 GiB 4.29 GB)
     Used Dev Size : 2095104 (2046.00 MiB 2145.39 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Mon Nov 13 00:35:42 2017
             State : clean
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : raid:5  (local to host raid)
              UUID : dfd6185a:5d2d29e3:0008c03e:65a1c54b
            Events : 18

    Number   Major   Minor   RaidDevice State
       0     259        3        0      active sync   /dev/nvme0n1p1
       1     259        4        1      active sync   /dev/nvme0n2p1
       3     259        5        2      active sync   /dev/nvme0n3p1
```

### 포맷 및 마운트 하기

이제 /dev/md5 라는 이름으로 파티션 디스크가 준비된 것과 동일하다. ext4 로 포맷하자.

```
[root@raid ~]# mkfs -t ext4 /dev/md5
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=128 blocks, Stripe width=256 blocks
262144 inodes, 1047552 blocks
52377 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1073741824
32 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

그리고 우리는 이 디스크를 /data 로 마운트하자.

```
[root@raid ~]# mkdir /data

[root@raid ~]# mount /dev/md5 /data

[root@raid ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/md126      5.8G  885M  4.6G  16% /
devtmpfs        2.0G     0  2.0G   0% /dev
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           2.0G  8.4M  2.0G   1% /run
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/md127      976M   91M  819M  10% /boot
tmpfs           396M     0  396M   0% /run/user/0
/dev/md5        3.9G   16M  3.7G   1% /data
```

정상적으로 마운트된 것을 확인할 수 있다. 파일시스템의 크기도 예상 대로 4 GiB 로 나온 것을 알 수 있다.

이제 fstab 에 등록해서 리부팅 시에도 동일하게 마운트되도록 하자.

```
[root@raid ~]# vi /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sun Nov 12 22:53:38 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=ca276cda-1a5f-4b91-ae3d-77680a41bce6 /                       ext4    defaults        1 1
UUID=60391e67-f9a0-4b0f-a023-432272f1ef06 /boot                   ext4    defaults        1 2
UUID=cf15ed7c-9801-41b1-aab6-e9db15143dbd swap                    swap    defaults        0 0

/dev/md5                                  /data                   ext4    defaults        1 2
```

사실 RAID 구조만 이해하면, 명령어로 SW RAID 를 구성하는 것은 위와 같이 매우 쉽다.

## RAID 5 에서 장애 발생 및 조치하기

자 이제 여기서 장애를 발생시키고, 정말 1개 디스크가 고장났을 때 복구가 가능한지, 2개 디스크가 고장났을 때 복구가 불가능한지 실습해보자.

우선 적당한 사이즈의 데이터를 /data 로 넣어놓고 아래를 진행하자.

```
[root@raid data]# ls -alh /data
total 88M
drwxr-xr-x.  3 root root 4.0K Nov 13 00:45 .
dr-xr-xr-x. 19 root root 4.0K Nov 13 00:35 ..
-rw-r--r--.  1 root root  88M Nov 13 00:45 boot.tar
drwx------.  2 root root  16K Nov 13 00:34 lost+found
```

### 디스크 1개 제거하기

Guest OS 를 종료시키고 우리가 추가했던 디스크 3개 중 1개를 아래와 같이 제거하자.

![RAID 5 Problem #1]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-problem-01.png){: .center-image}

그리고 Guest OS 를 시작시켜보자. 근데 뭔가 문제가 생길 줄 알았는데, 정상적으로 부팅이 된다. /dev/md5 RAID 의 상태를 한 번 확인해보자.

```
[root@raid ~]# mdadm --detail /dev/md5
/dev/md5:
           Version : 1.2
     Creation Time : Mon Nov 13 00:30:53 2017
        Raid Level : raid5
        Array Size : 4190208 (4.00 GiB 4.29 GB)
     Used Dev Size : 2095104 (2046.00 MiB 2145.39 MB)
      Raid Devices : 3
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Mon Nov 13 00:50:06 2017
             State : clean, degraded
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : raid:5  (local to host raid)
              UUID : dfd6185a:5d2d29e3:0008c03e:65a1c54b
            Events : 22

    Number   Major   Minor   RaidDevice State
       0     259        1        0      active sync   /dev/nvme0n1p1
       1     259        3        1      active sync   /dev/nvme0n2p1
       -       0        0        2      removed
```

원래 3개의 디스크들 중 1개가 removed 상태로 바뀌고 2개로만 운영되고 있는 것을 확인할 수 있다. 그럼 /data 안에 파일들은 정상적으로 동작하고 있을까? 확인해보자.

```
[root@raid ~]# cd /data

[root@raid data]# ls
boot.tar  lost+found

[root@raid data]# tar -xf boot.tar

[root@raid data]# ls
boot  boot.tar  lost+found

[root@raid data]# ls boot
System.map-3.10.0-693.el7.x86_64                         initramfs-3.10.0-693.el7.x86_64.img
config-3.10.0-693.el7.x86_64                             initrd-plymouth.img
efi                                                      lost+found
grub                                                     symvers-3.10.0-693.el7.x86_64.gz
grub2                                                    vmlinuz-0-rescue-3291d16e80fc46dba5c0e221b64c5594
initramfs-0-rescue-3291d16e80fc46dba5c0e221b64c5594.img  vmlinuz-3.10.0-693.el7.x86_64
```

파일들이 모두 정상적으로 존재하고, 압축 해제 등의 명령도 정상적으로 수행된다. 즉, 운영 상에 문제는 없다. 하지만 현재 상태에서 디스크가 1개 더 고장난다면 복구가 불가능할 것이다.

### 디스크 교체 및 복구하기

우선 이 상태에서 고장난 디스크를 얼른 교체해서 복구를 수행하기 위해 Guest OS 를 종료하고, 아래와 같이 새로운 디스크를 꼽아보자.

![RAID 5 Problem #2]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-problem-02.png){: .center-image}

이제 Guest OS 를 시작시키자. 당연하지만 새로 추가한 디스크가 자동으로 RAID 그룹에 추가되지 않는다. 새로 추가한 디스크를 확인해보고 파티셔닝하자.

```
[root@raid ~]# ls -al /dev/nvme*
crw-------. 1 root root 247, 0 Nov 13 00:57 /dev/nvme0
brw-rw----. 1 root disk 259, 0 Nov 13 00:57 /dev/nvme0n1
brw-rw----. 1 root disk 259, 1 Nov 13 00:57 /dev/nvme0n1p1
brw-rw----. 1 root disk 259, 2 Nov 13 00:57 /dev/nvme0n2
brw-rw----. 1 root disk 259, 3 Nov 13 00:57 /dev/nvme0n2p1
brw-rw----. 1 root disk 259, 4 Nov 13 00:57 /dev/nvme0n3

[root@raid ~]# fdisk /dev/nvme0n3
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xe34db38a.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-4194303, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
Using default value 4194303
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): fd
Changed type of partition 'Linux' to 'Linux raid autodetect'

Command (m for help): p

Disk /dev/nvme0n3: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xe34db38a

        Device Boot      Start         End      Blocks   Id  System
/dev/nvme0n3p1            2048     4194303     2096128   fd  Linux raid autodetect

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

이 새로운 파티션 디스크를 /dev/md5 RAID 에 추가하자.

```
[root@raid ~]# mdadm /dev/md5 --add /dev/nvme0n3p1
mdadm: added /dev/nvme0n3p1
```

그리고 바로 /dev/md5 의 상태를 확인해보자.

```
[root@raid ~]# mdadm --detail /dev/md5
/dev/md5:
           Version : 1.2
     Creation Time : Mon Nov 13 00:30:53 2017
        Raid Level : raid5
        Array Size : 4190208 (4.00 GiB 4.29 GB)
     Used Dev Size : 2095104 (2046.00 MiB 2145.39 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Mon Nov 13 01:02:45 2017
             State : clean, degraded, recovering
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 4% complete

              Name : raid:5  (local to host raid)
              UUID : dfd6185a:5d2d29e3:0008c03e:65a1c54b
            Events : 40

    Number   Major   Minor   RaidDevice State
       0     259        1        0      active sync   /dev/nvme0n1p1
       1     259        3        1      active sync   /dev/nvme0n2p1
       3     259        5        2      spare rebuilding   /dev/nvme0n3p1
```

새로 추가한 디스크가 보인다. 그리고 상태가 spare rebuilding 이다. 현재 존재하는 데이터를 분산시키고 Parity Bit 을 계산해서 넣는 중인 것이다. 잠시 후에 다시 상태를 확인하면 모두 active sync 인 것을 확인할 수 있다.

### 디스크 2개 제거하기

이제 완전한 장애 상황을 확인하기 위해 Guest OS 를 종료하고 RAID 5 를 위해 생성했던 3개의 디스크 중 2개를 제거하고, 다시 Guest OS 를 기동시켜보자.

아마도 이상하게 평소와는 부팅시간이 오래 걸리면서 Emergency Mode 로 들어오게 된다. RAID 를 구성한 디스크들 중에 정상적으로 데이터를 가져올 수 없는 상황이면 정상적인 부팅이 이루어지지 않는 것이다.

RAID 상태를 확인해보자.

(Emergency Mode 에서는 sshd 가 기동이 안되어서 SSH 를 붙을 수가 없다. 그래서 쉘의 내용을 복사할 수가 없어서 캡쳐로 대신한다.)

![RAID 5 Problem #3]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-problem-03.png){: .center-image}

State 가 inactive 이다. 즉 사용 불능 상태이다. OS 를 정상적으로 구동시키려면 이 문제를 해결시켜줘야 한다. 이미 디스크 2개가 없어졌으므로 데이터의 2/3 는 날아간 상태이다. 데이터의 복구는 불가능하다고 생각하는 것이 맞다.

이럴 때는 어쩔 수 없이 아래와 같이 이 RAID 를 제거해주고 OS 를 구동시켜야 한다.

![RAID 5 Problem #4]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-problem-04.png){: .center-image}

![RAID 5 Problem #5]({{ site.url }}/images/2017-11-12-raid-types-examples/raid-5-problem-05.png){: .center-image}

다시 시작하면 정상적으로 시작되는 것을 확인할 수 있다. 이후에는 디스크를 추가해서 다시 RAID 를 구성하던지 하는 추가 작업이 필요할 것이다.

## 마치며

모든 RAID 구성은 위와 같은 명령어와 장애 시뮬레이션을 통해 동작을 확인할 수 있다. 자기가 필요한 것을 직접 구성해보면 이해하는데 많은 도움이 될 것이다.

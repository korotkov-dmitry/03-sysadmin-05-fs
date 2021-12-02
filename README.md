# Домашнее задание к занятию "3.5. Файловые системы"

## 1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
Выполнено.
## 2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
`hardlink` - это ссылка на один объект, имеющий тот же inode, т.е права будут одинаковые.

        `vagrant@vagrant:~$ ls -l
        total 0
        -rw-rw-r-- 1 vagrant vagrant 0 Nov 28 12:50 test_fs
        vagrant@vagrant:~$ ln test_fs test_fs_link
        vagrant@vagrant:~$ ls -l
        total 0
        -rw-rw-r-- 2 vagrant vagrant 0 Nov 28 12:50 test_fs
        -rw-rw-r-- 2 vagrant vagrant 0 Nov 28 12:50 test_fs_link
        vagrant@vagrant:~$ chmod 0755 test_fs
        vagrant@vagrant:~$ ls -l
        total 0
        -rwxr-xr-x 2 vagrant vagrant 0 Nov 28 12:50 test_fs
        -rwxr-xr-x 2 vagrant vagrant 0 Nov 28 12:50 test_fs_link
        vagrant@vagrant:~$ stat test_fs
          File: test_fs
          Size: 0               Blocks: 0          IO Block: 4096   regular empty file
        Device: fd00h/64768d    Inode: 131087      Links: 2
        Access: (0755/-rwxr-xr-x)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
        Access: 2021-11-28 12:50:16.259108017 +0000
        Modify: 2021-11-28 12:50:16.259108017 +0000
        Change: 2021-11-28 12:53:51.006439169 +0000
         Birth: -
        vagrant@vagrant:~$ stat test_fs_link
          File: test_fs_link
          Size: 0               Blocks: 0          IO Block: 4096   regular empty file
        Device: fd00h/64768d    Inode: 131087      Links: 2
        Access: (0755/-rwxr-xr-x)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
        Access: 2021-11-28 12:50:16.259108017 +0000
        Modify: 2021-11-28 12:50:16.259108017 +0000
        Change: 2021-11-28 12:53:51.006439169 +0000
        Birth: -`        
## 3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end```

Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

Результат:

       `vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        sdc                    8:32   0  2.5G  0 disk`
## 4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
        `vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        ├─sdb1                 8:17   0    2G  0 part
        └─sdb2                 8:18   0  500M  0 part
        sdc                    8:32   0  2.5G  0 disk
        ├─sdc1                 8:33   0    2G  0 part
        └─sdc2                 8:34   0  500M  0 part
        vagrant@vagrant:~$ sudo fdisk -l /dev/sdb
        Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
        Disk model: VBOX HARDDISK
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disklabel type: dos
        Disk identifier: 0x6e80328c
        
        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdb1          2048 4218879 4216832    2G 83 Linux
        /dev/sdb2       4218880 5242879 1024000  500M 83 Linux
        vagrant@vagrant:~$ sudo fdisk -l /dev/sdc
        Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
        Disk model: VBOX HARDDISK
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disklabel type: dos
        Disk identifier: 0xa9286681
        
        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdc1          2048 4218879 4216832    2G 83 Linux
        /dev/sdc2       4218880 5242879 1024000  500M 83 Linux`
## 5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
        `root@vagrant:~# sfdisk -d /dev/sdb|sfdisk --force /dev/sdc
        Checking that no-one is using this disk right now ... OK

        Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
        Disk model: VBOX HARDDISK
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes

        >>> Script header accepted.
        >>> Script header accepted.
        >>> Script header accepted.
        >>> Script header accepted.
        >>> Created a new DOS disklabel with disk identifier 0x6e80328c.
        /dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
        /dev/sdc2: Created a new partition 2 of type 'Linux' and of size 500 MiB.
        /dev/sdc3: Done.

        New situation:
        Disklabel type: dos
        Disk identifier: 0x6e80328c

        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdc1          2048 4218879 4216832    2G 83 Linux
        /dev/sdc2       4218880 5242879 1024000  500M 83 Linux

        The partition table has been altered.
        Calling ioctl() to re-read partition table.
        Syncing disks.`

Результат:

        `vagrant@vagrant:~$ sudo fdisk -l /dev/sdb
        Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
        Disk model: VBOX HARDDISK
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disklabel type: dos
        Disk identifier: 0x6e80328c

        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdb1          2048 4218879 4216832    2G 83 Linux
        /dev/sdb2       4218880 5242879 1024000  500M 83 Linux
        vagrant@vagrant:~$ sudo fdisk -l /dev/sdc
        Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
        Disk model: VBOX HARDDISK
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disklabel type: dos
        Disk identifier: 0x6e80328c

        Device     Boot   Start     End Sectors  Size Id Type
        /dev/sdc1          2048 4218879 4216832    2G 83 Linux
        /dev/sdc2       4218880 5242879 1024000  500M 83 Linux`
## 6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

        `vagrant@vagrant:~$ sudo mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b1,c1}
        mdadm: Note: this array has metadata at the start and
            may not be suitable as a boot device.  If you plan to
            store '/boot' on this device please ensure that
            your boot-loader understands md/v1.x metadata, or use
            --metadata=0.90
        mdadm: size set to 2105344K
        Continue creating array? y
        mdadm: Defaulting to version 1.2 metadata
        mdadm: array /dev/md1 started.
        vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part  /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        ├─sdb1                 8:17   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdb2                 8:18   0  500M  0 part
        sdc                    8:32   0  2.5G  0 disk
        ├─sdc1                 8:33   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdc2                 8:34   0  500M  0 part`
## 7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
        
        `vagrant@vagrant:~$ sudo mdadm --create --verbose /dev/md0 -l 0 -n 2 /dev/sd{b2,c2}
        mdadm: chunk size defaults to 512K
        mdadm: /dev/sdb2 appears to be part of a raid array:
               level=raid1 devices=2 ctime=Thu Dec  2 04:47:08 2021
        mdadm: /dev/sdc2 appears to be part of a raid array:
               level=raid1 devices=2 ctime=Thu Dec  2 04:47:08 2021
        Continue creating array? y
        mdadm: Defaulting to version 1.2 metadata
        mdadm: array /dev/md0 started.
        vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part  /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        ├─sdb1                 8:17   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdb2                 8:18   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0
        sdc                    8:32   0  2.5G  0 disk
        ├─sdc1                 8:33   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdc2                 8:34   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0`
## 8. Создайте 2 независимых PV на получившихся md-устройствах.
        `vagrant@vagrant:~$ sudo pvcreate /dev/md1 /dev/md0
          Physical volume "/dev/md1" successfully created.
          Physical volume "/dev/md0" succe`
## 9. Создайте общую volume-group на этих двух PV.
        `vagrant@vagrant:~$ sudo vgcreate vg1 /dev/md1 /dev/md0
        Volume group "vg1" successfully created
        vagrant@vagrant:~$ sudo vgdisplay
          --- Volume group ---
          VG Name               vgvagrant
          System ID
          Format                lvm2
          Metadata Areas        1
          Metadata Sequence No  3
          VG Access             read/write
          VG Status             resizable
          MAX LV                0
          Cur LV                2
          Open LV               2
          Max PV                0
          Cur PV                1
          Act PV                1
          VG Size               <63.50 GiB
          PE Size               4.00 MiB
          Total PE              16255
          Alloc PE / Size       16255 / <63.50 GiB
          Free  PE / Size       0 / 0
          VG UUID               PaBfZ0-3I0c-iIdl-uXKt-JL4K-f4tT-kzfcyE

          --- Volume group ---
          VG Name               vg1
          System ID
          Format                lvm2
          Metadata Areas        2
          Metadata Sequence No  1
          VG Access             read/write
          VG Status             resizable
          MAX LV                0
          Cur LV                0
          Open LV               0
          Max PV                0
          Cur PV                2
          Act PV                2
          VG Size               2.97 GiB
          PE Size               4.00 MiB
          Total PE              761
          Alloc PE / Size       0 / 0
          Free  PE / Size       761 / 2.97 GiB
          VG UUID               FbOFow-ZUil-2ewY-l0Lo-mheX-GH9M-2Gvf8d`
## 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
        `vagrant@vagrant:~$ sudo lvcreate -L 100M vg1 /dev/md0
          Logical volume "lvol0" created.
          vagrant@vagrant:~$ sudo vgs
          VG        #PV #LV #SN Attr   VSize   VFree
          vg1         2   1   0 wz--n-   2.97g <2.88g
          vgvagrant   1   2   0 wz--n- <63.50g     0
          vagrant@vagrant:~$ sudo lvs
          LV     VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
          lvol0  vg1       -wi-a----- 100.00m
          root   vgvagrant -wi-ao---- <62.54g
          swap_1 vgvagrant -wi-ao---- 980.00m`
## 11. Создайте `mkfs.ext4` ФС на получившемся LV.
        `vagrant@vagrant:~$ sudo mkfs.ext4 /dev/vg1/lvol0
        mke2fs 1.45.5 (07-Jan-2020)
        Creating filesystem with 25600 4k blocks and 25600 inodes

        Allocating group tables: done
        Writing inode tables: done
        Creating journal (1024 blocks): done
        Writing superblocks and filesystem accounting information: done`
## 12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
        `vagrant@vagrant:~$ mkdir /tmp/new
        vagrant@vagrant:~$ sudo mount /dev/vg1/lvol0 /tmp/new`
## 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
        `vagrant@vagrant:~$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
        --2021-12-02 07:04:17--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
        Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
        Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
        HTTP request sent, awaiting response... 200 OK
        Length: 22743572 (22M) [application/octet-stream]
        Saving to: ‘/tmp/new/test.gz’
        
        /tmp/new/test.gz   100%[================>]  21.69M   824KB/s    in 30s

        2021-12-02 07:04:48 (740 KB/s) - ‘/tmp/new/test.gz’ saved [22743572/22743572]`
## 14. Прикрепите вывод `lsblk`.
        `vagrant@vagrant:~$ lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part  /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        ├─sdb1                 8:17   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdb2                 8:18   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0
            └─vg1-lvol0      253:2    0  100M  0 lvm   /tmp/new
        sdc                    8:32   0  2.5G  0 disk
        ├─sdc1                 8:33   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        └─sdc2                 8:34   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0
            └─vg1-lvol0      253:2    0  100M  0 lvm   /tmp/new`
## 15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
    
        `vagrant@vagrant:~$ gzip -t /tmp/new/test.gz
        vagrant@vagrant:~$  echo $?
        0`
## 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
        `vagrant@vagrant:~$ sudo pvmove /dev/md0
          /dev/md0: Moved: 100.00%
        vagrant@vagrant:~lsblk
        NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
        sda                    8:0    0   64G  0 disk
        ├─sda1                 8:1    0  512M  0 part  /boot/efi
        ├─sda2                 8:2    0    1K  0 part
        └─sda5                 8:5    0 63.5G  0 part
          ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
          └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
        sdb                    8:16   0  2.5G  0 disk
        ├─sdb1                 8:17   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        │   └─vg1-lvol0      253:2    0  100M  0 lvm   /tmp/new
        └─sdb2                 8:18   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0
        sdc                    8:32   0  2.5G  0 disk
        ├─sdc1                 8:33   0    2G  0 part
        │ └─md1                9:1    0    2G  0 raid1
        │   └─vg1-lvol0      253:2    0  100M  0 lvm   /tmp/new
        └─sdc2                 8:34   0  500M  0 part
          └─md0                9:0    0  996M  0 raid0`
## 17. Сделайте `--fail` на устройство в вашем RAID1 md.
        `vagrant@vagrant:~$ sudo mdadm /dev/md1 --fail /dev/sdb1
        mdadm: set /dev/sdb1 faulty in /dev/md1
        vagrant@vagrant:~$ sudo mdadm -D /dev/md1
        /dev/md1:
                   Version : 1.2
             Creation Time : Thu Dec  2 06:45:52 2021
                Raid Level : raid1
                Array Size : 2105344 (2.01 GiB 2.16 GB)
             Used Dev Size : 2105344 (2.01 GiB 2.16 GB)
              Raid Devices : 2
             Total Devices : 2
               Persistence : Superblock is persistent
        
               Update Time : Thu Dec  2 07:11:46 2021
                     State : clean, degraded
            Active Devices : 1
           Working Devices : 1
            Failed Devices : 1
             Spare Devices : 0
        
        Consistency Policy : resync
        
                      Name : vagrant:1  (local to host vagrant)
                      UUID : e8e9a716:d6e301e6:069ec846:b8f8e766
                    Events : 19
        
            Number   Major   Minor   RaidDevice State
               -       0        0        0      removed
               1       8       33        1      active sync   /dev/sdc1
        
               0       8       17        -      faulty   /dev/sdb1`
## 18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
        `vagrant@vagrant:~$ dmesg |grep md1
        [ 2541.574146] md/raid1:md1: not clean -- starting background reconstruction[ 2541.574148] md/raid1:md1: active with 2 out of 2 mirrors
        [ 2541.574165] md1: detected capacity change from 0 to 2155872256
        [ 2541.577394] md: resync of RAID array md1
        [ 2551.891802] md: md1: resync done.
        [ 4095.181629] md/raid1:md1: Disk failure on sdb1, disabling device.
                       md/raid1:md1: Operation continuing on 1 devices.`
## 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

## 20. Погасите тестовый хост, `vagrant destroy`.

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
        /dev/sdb2       4218880 5242879 1024000  500M 83 Linux`
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

## 7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

## 8. Создайте 2 независимых PV на получившихся md-устройствах.

## 9. Создайте общую volume-group на этих двух PV.

## 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

## 11. Создайте `mkfs.ext4` ФС на получившемся LV.

## 12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

## 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

## 14. Прикрепите вывод `lsblk`.

## 15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

## 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

## 17. Сделайте `--fail` на устройство в вашем RAID1 md.

## 18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

## 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

## 20. Погасите тестовый хост, `vagrant destroy`.

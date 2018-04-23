# archlabs-btrfs
Install ArchLabs &lt;www.archlabslinux.com> 2018.03 with Btrfs filesystem.

## Disk Partition Layout
1. boot (512M) with ext4 filesystem,
2. home (5G) with xfs filesystem,
3. root (5G) with btrfs (@, @btrfs, @pkg, @log, @snapshots),
4. swap (2.5G) with swap space.
> For demonstration purposes only. Do **not** use this scheme for any live or productive system.

## Preparations
1. Create a full backup of your **current** system (!).
1. Download ArchLabs Linux from [here](https://archlabslinux.com/get-archlabs-2/) and save it.
2. Copy ISO image to USB stick with dd: `dd if=archlabs-2018-03.iso of=/dev/sdX bs=5M && sync`
3. Boot ArchLabs (x86_64) with your fresh USB stick.
4. Right-Click on Desktop and select "Install Archlabs".

## System Setup
1. Select Language.

2. Go to Prepare System.
    1. Set Keyboard Layout.
    2. Partition Device: /dev/sda 13G
    3. Chose cfdisk and select gpt for label type.
    4. [NEW] -> Partition size: 512M     # Boot
    5. [NEW] -> Partition size: 5G     # Root with @, @pkg, @log, @snapshots
    6. [NEW] -> Partition size: 5G     # Home
    7. [NEW] -> Partition size: 2.5G # Swap
    8. Click on Write, confirm and exit.
 ```
root@ArchLabs / # fdisk -l
Disk /dev/sda: 13 GiB, 13958643712 bytes, 27262976 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: D696C02E-4788-2346-AA7A-C8E23E1FCDE7

Device        Start      End  Sectors  Size Type
/dev/sda1      2048  1050623  1048576  512M Linux filesystem
/dev/sda2   1050624 11536383 10485760    5G Linux filesystem
/dev/sda3  11536384 22022143 10485760    5G Linux filesystem
/dev/sda4  22022144 27262942  5240799  2.5G Linux filesystem

...
```

3. Click CTRL+F2 and login as root with password archlabs.
4. Format partitions:
    1. `mkfs.ext4 -L boot.ext4 /dev/sda1`    # Boot
    2. `mkfs.btrfs -L root.btrfs /dev/sda2` # Root with @, @pkg, @log, @snapshots
    3. `mkfs.xfs -L home.xfs /dev/sda3` # Home
    4. `mkswap -L swap.swap /dev/sda4` # Swap
 ```
 
root@ArchLabs / # mkfs.ext4 -L boot.ext4 /dev/sda1
mke2fs 1.43.9 (8-Feb-2018)
Creating filesystem with 131072 4k blocks and 32768 inodes
Filesystem UUID: 2de35468-684c-4804-a29c-51aa708bf5e0
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done


root@ArchLabs / # mkfs.btrfs -L root.btrfs /dev/sda2
btrfs-progs v4.15.1
See http://btrfs.wiki.kernel.org for more information.

Label:              root.btrfs
UUID:               0fdbc292-a9b2-4862-a4be-61b019db243a
Node size:          16384
Sector size:        4096
Filesystem size:    5.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP             256.00MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     5.00GiB  /dev/sda2


root@ArchLabs / # mkfs.xfs -L home.xfs /dev/sda3
meta-data=/dev/sda3              isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0, rmapbt=0, reflink=0
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

root@ArchLabs / # mkswap -L swap.swap /dev/sda4
Setting up swapspace version 1, size = 2.5 GiB (2683281408 bytes)
LABEL=swap.swap, UUID=caffb094-e15e-4803-875f-b5a2b7152abc
```

5. Create btrfs subvolumes:
    1. `mount /dev/sda2 /mnt`
    2. `btrfs subvolume create /mnt/@`
    3. `btrfs subvolume create /mnt/@pkg`
    4. `btrfs subvolume create /mnt/@log`   
    5. `btrfs subvolume create /mnt/@snapshots`
    6. `umount /mnt`
```
root@ArchLabs / # mount /dev/sda2 /mnt
root@ArchLabs / # btrfs subvolume create /mnt/@
Create subvolume '/mnt/@'
root@ArchLabs / # btrfs subvolume create /mnt/@pkg
Create subvolume '/mnt/@pkg'
root@ArchLabs / # btrfs subvolume create /mnt/@log
Create subvolume '/mnt/@log'
root@ArchLabs / # btrfs subvolume create /mnt/@snapshots
Create subvolume '/mnt/@snapshots'
root@ArchLabs / # ls /mnt
@  @log  @pkg  @snapshots
root@ArchLabs / # umount /mnt
```

6. Create folders and mount btrfs subvolumes:
    1. `mount -o noatime,compress=lzo,space_cache,ssd,subvol=@ /dev/sda2 /mnt`
    2. `mkdir -p /mnt/{btrfs,boot,home,var/log,var/cache/pacman/pkg,.snapshots}`
    3. `mount -o noatime,compress=lzo,space_cache=v2,ssd,subvolid=5 /dev/sda2 /mnt/btrfs`
    4. `mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots`
    5. `mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg/`
    6. `mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@log /dev/sda2 /mnt/var/log/`
```
root@ArchLabs / # mount -o noatime,compress=lzo,space_cache,ssd,subvol=@ /dev/sda2 /mnt
root@ArchLabs / # mkdir -p /mnt/{btrfs,boot,home,var/log,var/cache/pacman/pkg,.snapshots}
root@ArchLabs / # ls /mnt
boot  btrfs  home  var

root@ArchLabs / # mount -o noatime,compress=lzo,space_cache=v2,ssd,subvolid=5 /dev/sda2 /mnt/btrfs
root@ArchLabs / # mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots
root@ArchLabs / # mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg/
root@ArchLabs / # mount -o noatime,compress=lzo,space_cache=v2,ssd,subvol=@log /dev/sda2 /mnt/var/log/
```

7. Mount boot, home and enable swap partition:
    1. `mount /dev/sda1 /mnt/boot`
    2. `mount /dev/sda3 /mnt/home`
    3. `swapoff -a && swapon /dev/sda4`
```
root@ArchLabs / # mount /dev/sda1 /mnt/boot
root@ArchLabs / # mount /dev/sda3 /mnt/home
root@ArchLabs / # swapoff -a && swapon /dev/sda4
```

8. Verify setup with:
    1. `mount`
    2. `swapon -s`
```
root@ArchLabs / # mount
...
/dev/sda2 on /mnt type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@)
/dev/sda2 on /mnt/btrfs type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=5,subvol=/)
/dev/sda2 on /mnt/.snapshots type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=260,subvol=/@snapshots)
/dev/sda2 on /mnt/var/cache/pacman/pkg type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=258,subvol=/@pkg)
/dev/sda2 on /mnt/var/log type btrfs (rw,noatime,compress=lzo,ssd,space_cache,subvolid=259,subvol=/@log)
/dev/sda1 on /mnt/boot type ext4 (rw,relatime,data=ordered)
/dev/sda3 on /mnt/home type xfs (rw,relatime,attr2,inode64,noquota)

root@ArchLabs / # swapon -s
Filename                                Type            Size    Used    Priority
/dev/sda4                               partition       2620392 0       -2
```

9. Leave terminal by pressing ALT+[KEY-LEFT] two times.
10. Go Back and Install ArchLabs.
11. Wait a few minutes. 
12. Select as Bootloader Device your partition /dev/sda.
13. Set System Hostname, Locale + Timezone and Root Password.
14. Create a New User and apply System Tweaks (if required).
15. Go Back, click on Done and restart into your (new) ArchLabs system.

```
florian@myarchlabs ~ % df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
dev            devtmpfs  2.0G     0  2.0G   0% /dev
run            tmpfs     2.0G  476K  2.0G   1% /run
/dev/sda2      btrfs     5.0G  2.9G  1.8G  62% /
tmpfs          tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs          tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
tmpfs          tmpfs     2.0G  6.3M  2.0G   1% /tmp
/dev/sda2      btrfs     5.0G  2.9G  1.8G  62% /btrfs
/dev/sda2      btrfs     5.0G  2.9G  1.8G  62% /var/cache/pacman/pkg
/dev/sda2      btrfs     5.0G  2.9G  1.8G  62% /var/log
/dev/sda2      btrfs     5.0G  2.9G  1.8G  62% /.snapshots
/dev/sda1      ext4      488M   54M  400M  12% /boot
/dev/sda3      xfs       5.0G   50M  5.0G   1% /home
tmpfs          tmpfs     395M  8.0K  395M   1% /run/user/1000

florian@myarchlabs ~ % ls /btrfs
@  @log  @pkg  @snapshots

florian@myarchlabs ~ % cat /etc/fstab
# /dev/sda2 LABEL=root.btrfs
UUID=0fdbc292-a9b2-4862-a4be-61b019db243a       /               btrfs           rw,noatime,compress=lzo,ssd,space_cache,subvolid=257,subvol=/@,subvol=@      0 0

# /dev/sda2 LABEL=root.btrfs
UUID=0fdbc292-a9b2-4862-a4be-61b019db243a       /btrfs          btrfs           rw,noatime,compress=lzo,ssd,space_cache,subvolid=5,subvol=/  0 0

# /dev/sda2 LABEL=root.btrfs
UUID=0fdbc292-a9b2-4862-a4be-61b019db243a       /.snapshots     btrfs           rw,noatime,compress=lzo,ssd,space_cache,subvolid=260,subvol=/@snapshots,subvol=@snapshots    0 0

# /dev/sda2 LABEL=root.btrfs
UUID=0fdbc292-a9b2-4862-a4be-61b019db243a       /var/cache/pacman/pkg   btrfs           rw,noatime,compress=lzo,ssd,space_cache,subvolid=258,subvol=/@pkg,subvol=@pkg        0 0

# /dev/sda2 LABEL=root.btrfs
UUID=0fdbc292-a9b2-4862-a4be-61b019db243a       /var/log        btrfs           rw,noatime,compress=lzo,ssd,space_cache,subvolid=259,subvol=/@log,subvol=@log        0 0

# /dev/sda1 LABEL=boot.ext4
UUID=2de35468-684c-4804-a29c-51aa708bf5e0       /boot           ext4            rw,relatime,data=ordered        0 2

# /dev/sda3 LABEL=home.xfs
UUID=dae18c6c-4361-4e73-a775-102662f08ff5       /home           xfs             rw,relatime,attr2,inode64,noquota   0 2

# /dev/sda4 LABEL=swap.swap
UUID=caffb094-e15e-4803-875f-b5a2b7152abc       none            swap            defaults,pri=-2 0 0
```

## Acknowledgements
1. [Btrfs](https://btrfs.wiki.kernel.org/index.php/Main_Page) for this great and powerful filesystem,
2. [unicks.eu](http://unicks.eu) aka Nick for all the great videos about GNU/Linux!

## ToDoList
1. [ ] Fix Markdown Syntax.

# archlabs-btrfs
Install ArchLabs &lt;www.archlabslinux.com> 2018.03 with Btrfs filesystem.

## Preparations
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
    5. [NEW] -> Partition size: 10G     # / with @, @home, @cache, @.snapshots
    6. [NEW] -> Partition size: 2.5G # Swap
    7. Click on Write, confirm and exit.

3. Click CTRL+F2 and login as root with password archlabs.
4. Format partitions:
    1. `mkfs.ext4 /dev/sda1`    # Boot
    2. `mkfs.btrfs /dev/sda2`   # / with @, @home, @cache, @.snapshots
    3. `mkswap /dev/sda3`       # Swap

5. Create btrfs subvolumes:
    1. `mount /dev/sda2 /mnt`
    2. `btrfs subvolume create /mnt/@`
    3. `btrfs subvolume create /mnt/@home`
    4. `btrfs subvolume create /mnt/@cache`
    5. `btrfs subvolume create /mnt/@snapshots`
    6. `umount /mnt`

6. Create folders and mount btrfs subvolumes:
    1. `mount -o noatime,compress=lzo,space_cache,ssd,subvol=@ /dev/sda2 /mnt`
    2. `mkdir /mnt/boot`
    3. `mkdir /mnt/home`
    4. `mkdir /mnt/var/cache -p`
    5. `mkdir /mnt/.snapshots`
    6. `mount -o noatime,compress=lzo,space_cache,ssd,subvol=@home /dev/sda2 /mnt/home`
    7. `mount -o noatime,compress=lzo,space_cache,ssd,subvol=@cache /dev/sda2 /mnt/var/cache`
    8. `mount -o noatime,compress=lzo,space_cache,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots`

7. Mount boot and enable swap partition:
    1. `mount /dev/sda1 /mnt/boot`
    2. `swapoff /dev/sda3 && swapon /dev/sda3`

8. Leave terminal with ALT+<KEY-LEFT> two times.
9. Go Back and Install ArchLabs.
10. Wait a few minutes. 
11. Select as Bootloader Device /dev/sda  13G
12. Set System Hostname, Locale + Timezone and Root Password.
13. Create a New User and apply System Tweaks (if required).
14. Go Back, click on Done and restart your new btrfs system with ArchLabs.

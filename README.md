# Arch Linux installation

Arch Linux installation process on my Lenovo Yoga S730.  
This note is subject to change; changes in preference of utilities or changes in the installation process itself.

## Getting the Arch ISO running on the laptop

I'll probably write a Python script to automatically download the ISO and verify its integrity by hashing the file and compare it with the hashed version on the official website.

Use Rufus to burn the ISO onto my USB stick that must have at least a USB type-C connector. 

Plug the USB stick onto the laptop, turn it on and press F2 to enter the BIOS. Check the boot order and ensure the USB stick has the highest boot priority.

Once loaded, enter the Arch Linux installation medium.

## Verify boot mode

To verify that the system is booted in UEFI mode, list the efivars directory:

    ls /sys/firmware/efi/efivars

## Enable Network Time Protocol (NTP)

Turn on NTP to ensure that the clock is always synchronise to the internet.

    timedatectl set-ntp true
  
## Set up network adapter and connect to internet

I will be using `iwctl` utility to set up my network adapter and connect to my home router wirelessly to gain internet access.

Enter `iwctl` to enter the `iwd` prompt, and I will be entering the following commands:

    station wlan0 scan
    station wlan0 get-networks
    station wlan0 connect *SSID*

The first command will use the network adapter to scan for any surrounding networks. It will not show anything until I enter the second command which will show what the network adapter has detected. Finally, the third command will allow me to connect to the network. If the SSID has a passphrase, it will prompt after passing the command.

To verify that the network adapter has connected to the network, enter the following command to verify:

    station wlan0 show

Exit the `iwd` prompt:

    quit

To check if the laptop can access the internet, do a quick ping:

    ping archlinux.org

If there is reply, it means it's successful in connecting to the internet.

## Disk partitioning

I will be using `fdisk` utility to partition my SSD into 3 partitions: EFI system partitions, Linux Swap, and the Linux root `/`.

To start, enter the following command to modify the partition table:

    fdisk /dev/nvme0n1

Enter `g` to create a new GPT table.

Enter `n` to create a new partition. 

For the first partition, it will be the EFI partition. The partition number and the first sector of the partition will be by default. However, for the last sector, I will enter `+500M` to determine the size of the partition, which will be 500MB.

*If asked whether to remove vfat signature, enter `Y`.*

Enter `t` to change the partition type.

It will select partition 1 by default. To switch to EFI partition, enter `1`.

Create another new partition by entering `n`.

For this second partition, it will be Linux Swap which acts like virtual memory. The partition number and first sector will be default once again, and for the last sector, I will opt `+8G` or 8GB of swap partition. There's no consensus on how much swap I really need, it really depends, so it will vary.

Enter `t` to change the partition type and enter `2` for partition 2.

For Linux Swap, the alias will be `19`.

Finally, create a last partition for the root `/` with `n`, and enter the rest by default since it will be taking up the remainder of the SSD. I do not need to change the partition type since the default is Linux Filesystem.

| Partition type | Size | Alias |
| -------------- | ---- | ----- |
| EFI System | 500MB | 1 |
| Linux Swap | 8GB | 19 |
| Linux Filesystem | \**Remainder* | 20 |

\* The laptop has around 477GB of storage, so the remainder in this case will be 468.5GB. It'll vary depending on how much I allocate for the EFI and swap partitions

Lastly, enter `w` to write all the changes to the disk and exit to the terminal.

## Partition formatting

The partitions created can't be used without any file system; the file system is needed for how data is being organised. For the EFI system, it is mandatory to use FAT32; Swap partition with creating swap area; Linux Filesystem with EXT4.

The following commands will do the formatting in sequence above:

    mkfs.fat -F 32 /dev/nvme0n1p1
    mkswap /dev/nvme0n1p2
    mkfs.ext4 /dev/nvme0n1p3

## Mounting file systems

Now I need to mount the partitions to the appropriate location. The EFI partition will be mounted to `/mnt/boot`, but that directory does not exist so I need to create first. As for swap partition, I simply have to enable it. And for the Linux Filesystem, I will mount it to `/mnt`.

The following commands will do the mounting in sequence above:

    mkdir /mnt/boot
    mount /dev/nvme0n1p1 /mnt/boot
    swapon /dev/nvme0n1p2
    mount /dev/nvme0n1p3 /mnt

This will be the final product:

| Partition type | File system | Mount point |
| -------------- | ---- | ----- |
| EFI System | `FAT32` | `/mnt/boot` |
| Linux Swap | `[SWAP]` | `[SWAP]` |
| Linux Filesystem | `EXT4` | `/mnt` |

## Installation

Installing all the foundations of the systems as well as the essential tools (and may be essential) needed. The command to install will be this:

    pacstrap /mnt base linux linux-firmware linux-headers grub efibootmgr intel-ucode man vim nano gedit networkmanager pulseaudio reflector base-devel git

## Configuration

To make sure that the appropriate mountings can be done automatically every time the machine boots, I will generate a fstab file based on the current mounting. The command to generate the fstab will be this:

    genfstab -U /mnt >> /mnt/etc/fstab

Next, I will be transferring root into the newly installed system with the command:

    arch-chroot /mnt

## Timezone

Since `timedatectl` no longer works after chroot, I will have to manually create a link so that the correct timezone can be set on the system. The command will be:

    ln -sfv /usr/share/zoneinfo/Singapore/Singapore /etc/localtime

Once done, adjust the hardware clock based on the system clock with the command:

    hwclock --systohc

## Localisation

Edit the file `/etc/locale.gen` and uncomment `en_SG.UTF-8 UTF-8`. Once done, generate the locale with the command:

    locale-gen

Once generated, append a line to `/etc/locale.conf` such that the system will use the locale I want:

    echo "LANG=en_SG.UTF-8" >> /etc/locale.conf

## Network configuration

Add a hostname to my system:

    echo "Shawn-Arch-Linux" >> /etc/hostname

Modify `/etc/hosts` and add the following lines:

    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   Shawn-Arch-Linux

## User account and passowrd

Create my own user account that allows me to run administrative task.

First, modify the `/etc/sudoers` file with `visudo` such that `wheel ALL=(ALL:ALL) ALL` is uncommented. The wheel group allows users to have administrative privilege.

Next, create my own account and assign myself to the appropriate group:

    useradd -m shawn
    usermod -aG wheel shawn

Finally, assign passwords to both the user account and root:

    passwd shawn
    passwd

## Set up bootloader

halfway

    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

and

    grub-mkconfig -o /boot/grub/grub.cfg

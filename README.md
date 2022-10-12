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

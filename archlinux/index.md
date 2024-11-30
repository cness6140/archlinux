---
layout: page
title: Arch
permalink: /archlinux/
---

# Arch Linux Install

I used the [Archboot](https://release.archboot.com/aarch64/2024.09/iso/) ARM install of Arch Linux for this project, downloading the ISO and installing it to VMWare Fusion.

I checked the IP and network connectivity:
![alt text](<Screenshot 2024-11-03 at 1.42.34 PM.png>)
Figure 1

Used `cfdisk` to partition the disk into EFI, swap, and the root filesystem:

![alt text](<Screenshot 2024-11-03 at 2.05.56 PM.png>)
Figure 2

![alt text](<Screenshot 2024-11-03 at 2.11.24 PM.png>)
Figure 3

I then formatted the new partitions; I created an ext4 filesystem, initialized the `swap` partition, and formatted the EFI partition to FAT32.

![alt text](<Screenshot 2024-11-03 at 2.28.44 PM.png>)
Figure 4

I then mounted the partitions to /mnt (I forgot to take screenshots for this part).

```
mount /dev/nvme0n1p3 /mnt
mount /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
```

I then selected the mirrors:

![alt text](<Screenshot 2024-11-03 at 2.35.44 PM.png>)
Figure 5

![alt text](<Screenshot 2024-11-03 at 2.47.02 PM.png>)
Figure 6

I then ran into a lot of issues installing pacman. At first I typed in the command `pacstrap -K /mnt base linux linux-firmware` incorrectly: I used an underscore instead of a hyphen and kept getting the error `target not found: linux_firmware`. Once I realized that silly error, I ran the command again, but got the error `Out of Memory`. This was confusing, as I had already installed a 10GB partition of swap, but when I ran `free -l`, it said I had 0GB of swap space.

I eventually realized that I must've turned on swap incorrectly previously. I reran `mkswap /dev/nvme0n1p2` before running `swapon /dev/nvme0n1p2`. Once I ran these again, I had memory. 

However, the pacstrap command still didn't work; I then got the following error: 

![alt text](<Screenshot 2024-11-17 at 1.14.22 PM.png>)
Figure 7

Users on Reddit suggested that removing the file `/var/lib/pacman/db.lck` would fix this issue. It did not.

I also ran `ps aux | grep pacman` and killed the few pacman processes that were running, before re-running pacstrap. This also did not work.

I then decided to re-run every command I had done after `cfdisk`, and then try `pacstrap` again. And then, mysteriously, it worked. I think the issue was that I would shut down my VM and restart it when resuming work, and each time that happened, systemd unmounted everything, but I mistakenly assumed that every change I made stayed between shutdowns.

![alt text](<Screenshot 2024-11-17 at 1.29.50 PM.png>)
Figure 8

![alt text](<Screenshot 2024-11-17 at 1.35.39 PM.png>)
Figure 9

Yay! Now arch-chroot seemed to work. Sadly, this was far from the end of my problems. As I continued through the steps, `nano` was required to edit files. Chroot did not have nano, but when I tried to install it, a required file was corrupted.

![alt text](<Screenshot 2024-11-17 at 2.08.28 PM.png>)
Figure 10

![alt text](<Screenshot 2024-11-17 at 2.15.18 PM.png>)
Figure 11

To attempt to solve this issue, I updated the keyring. I removed everything in `/var/cache/pacman/pkg/`, as well as `/etc/pacman.d/gnug.` I then reinitialized the keyring (`pacman-key --init`), repopulated it (`pacman-key --populate archlinux`), and updated the system package database (`pacman -Sy`). However, this did not work. The file was still corrupted, and I could not install `nano`. Attempting to install other packages had the same issue.

![alt text](<Screenshot 2024-11-18 at 4.09.35 AM.png>)
Figure 12

Trying to install the archlinuxarm keyring, I ran out of memory. I ran `free -l` and saw I still have nearly 10G of swap available, but for some reason it's not being utilized. I tried to up the priority with `swapon /dev/nvme0n1p2 -p 10`, and adjust the swappiness parameter using `sysctl vm.swappiness=10 `, but even after these changes I still received the `Out of Memory` error.

Once I receive the `Out of Memory` error, I can't Ctrl-C out of the command, so I have to restart the VM. But, of course, restarting the VM un-mounts everything, and I have to start over every time. I can't use snapshots, because after restoring a snapshot, the keyboard can't input (so I can't type anything).

I then realized the error was likely to do with the ISO being for ARM, but the keyring wasn't being populated for ARM specifically. I tried to remedy this following the instructions on [this](https://archlinuxarm.org/about/package-signing) link. Sadly, these did not solve the error.

At this point, I was out of ideas, so decided to pivot. I found [these](https://www.infinitescript.com/2022/12/install-arch-linux-arm-on-macbooks-with-apple-silicon/) instructions, aimed specifically at installing Arch Linux on an ARM Macbook. I created a new VM, and used these instructions roughly as a guide for installation. Setup was the same as before until setup of the Linux system, using `pacstrap`.

Instead of `pacstrap -K /mnt base linux linux-firmware`, I used `pacstrap -i /mnt base base-devel net-tools`. This did not yet install the Linux kernel or firmware, and installed additional development tools.

![alt text](<Screenshot 2024-11-24 at 4.18.05 PM.png>)
Figure 13

I then generated the fstab, very similarly to as in Figure 9:
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Then I could arch-chroot. Once in arch-chroot, I immediately tested for installation functionality (for presence of the corruption error from earlier) by installing `nano`. This time, installation of `nano` was successful.

![alt text](<Screenshot 2024-11-24 at 4.20.12 PM.png>)
Figure 14

Then I was able to configure various settings. The timezone:
```
ln -s /usr/share/zoneinfo/US/Central /etc/localtime
```
 locale, by removing `#` before `en_US.UTF-8` in `/etc/locale.gen` , and setting the language to English:

![alt text](<Screenshot 2024-11-24 at 4.23.15 PM.png>)
Figure 15

Setting the hostname:
```
echo "ArchLinux" > /etc/hostname
```
And setting the password with `passwd`.

Then, I installed the Linux kernel, and created an `initramfs` (initial RAM filesystem) image:

![alt text](<Screenshot 2024-11-24 at 4.26.28 PM.png>)
Figure 16

![alt text](<Screenshot 2024-11-24 at 4.28.01 PM.png>)
Figure 17

I then installed the bootloader, using grub and efibootmgr:
```
pacman -S grub efibootmgr
```
![alt text](<Screenshot 2024-11-24 at 4.31.03 PM.png>)
Figure 18

These were also successful. The final step before rebooting was to install `dhcpcd`:
```
pacman -S dhcpcd
```
Then I rebooted the system:
```
exit
umount /mnt/boot
umount /mntreboot
```

The VM opened into a CLI login screen upon reboot.

![alt text](<Screenshot 2024-11-24 at 5.28.38 PM.png>)
Figure 19

The first task was to connect to the internet. I listed the network interfaces, then started the network interface `ens160`.

![alt text](<Screenshot 2024-11-24 at 5.30.04 PM.png>)
Figure 20

Then used `dhcpcd` to obtain an IP address.

![alt text](<Screenshot 2024-11-24 at 5.31.08 PM.png>)

I also created users Justin, Codi, and Clara, and added them to the `sudoers` file.

![alt text](<Screenshot 2024-11-24 at 6.44.04 PM.png>)

I did take more screenshots past this point, but my laptop stopped saving them and I didn't realize until now, so I apologize. I'm writing this part of the report a few days after I actually finished the project, so hopefully I remember everything...

I decided to use KDE Plasma as my desktop environment:
``` 
pacman -S plasma kde-applications
pacman -S sddm
systemctl enable sddm.service
systemctl start sddm.service
```

Then, after rebooting the system, it booted into a login page:

![alt text](<Screenshot 2024-11-29 at 5.54.06 PM.png>)

![alt text](<Screenshot 2024-11-26 at 12.31.31 PM.png>)
![alt text](<Screenshot 2024-11-26 at 1.41.09 PM.png>)
![alt text](<Screenshot 2024-11-25 at 12.33.33 PM.png>)
![alt text](<Screenshot 2024-11-25 at 12.32.37 PM.png>)
![alt text](<Screenshot 2024-11-25 at 12.11.57 PM.png>)
![alt text](<Screenshot 2024-11-26 at 12.32.48 PM.png>)
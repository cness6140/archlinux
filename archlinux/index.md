---
layout: page
title: Arch
permalink: /archlinux
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

`mount /dev/nvme0n1/p3 /mnt`

`mount /dev/nvme0n1/p1 /mnt/boot`

`swapon /dev/nvme0n1/p2`

I then selected the mirrors:

![alt text](<Screenshot 2024-11-03 at 2.35.44 PM.png>)
Figure 5

![alt text](<Screenshot 2024-11-03 at 2.47.02 PM.png>)

I then ran into a lot of issues installing pacman. At first I typed in the command `pacstrap -K /mnt base linux linux-firmware` incorrectly: I used an underscore instead of a hyphen and kept getting the error `target not found: linux_firmware`. Once I realized that silly error, I ran the command again, but got the error `Out of Memory`. This was confusing, as I had already installed a 10GB partition of swap, but when I ran `free -l`, it said I had 0GB of swap space.

I eventually realized that I must've turned on swap incorrectly previously. I reran `mkswap /dev/nvme0n1p2` before running `swapon /dev/nvme0n1p2`. Once I ran these again, I had memory. 

However, the pacstrap command still didn't work; I then got the following error: 

![alt text](<Screenshot 2024-11-17 at 1.14.22 PM.png>)

Users on Reddit suggested that removing the file `/var/lib/pacman/db.lck` would fix this issue. It did not.

I also ran `ps aux | grep pacman` and killed the few pacman processes that were running, before re-running pacstrap. This also did not work.

I then decided to re-run every command I had done after `cfdisk`, and then try `pacstrap` again. And then, mysteriously, it worked. Not sure why.



You probably can't tell, since this README is pretty short, but I spent a really long time troubleshooting stuff here... and I'm not sure how to fix it all :(
    
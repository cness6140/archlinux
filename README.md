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

I then ran into a lot of issues installing pacman.
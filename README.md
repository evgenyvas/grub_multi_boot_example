# UEFI GRUB multi boot example
A grub.cfg example of multi iso UEFI boot

This guide is based on a host using a recent version of Ubuntu (or similar). No other prerequisites are required.
Users attempting to create a multiboot USB from CentOS 7 (or similar) will need to install some prerequisite packages on their host.
```
# CentOS 7 ONLY:
yum install dosfstools grub2-efi-modules
```

**Setup your USB**:
--------
8GB USB drive for example
- Clean the beginning of USB drive to prevent possible problems:
```
dd if=/dev/zero of=/dev/sdb bs=2M count=2 (make sure your usb device is sdb!)
sync ; echo 1 | tee /proc/sys/vm/drop_caches
```
- Start with offset of 2MB (you should check your drive to see what the optimal offset is)
Create a new disklabel of GPT type using parted: (use extreme caution; parted could destroy your linux installation if you are careless)
Some users experience issues when using USB devices with 'advanced format' (4K sector size) and an EFI System Partition less than 270MB.
To ensure compatibility with all USB drives types, the ESP should be set to at least 270MB.
```
parted -a optimal /dev/sdb (make sure drive you are working on is sdb)
mklabel gpt
unit MB
mkpart EFIBOOT fat32 0% 270M   (creates the EFI partition which is 270M in size)
set 1 boot on
mkpart bootiso ext4 270M 7.5G  (creates a second partition of 7.5 GB)
mkpart storage fat32 7.5G 100%  (creates a third partition which will take up the rest of the drive)
unit MB  
print
print free
align-check optimal 1
align-check optimal 2
align-check optimal 3
quit
```
- Now you must create filesystems:
```
mkfs.vfat -F32 -n EFIBOOT /dev/sdb1
mkfs.ext4 -L bootiso /dev/sdb2

(If you receive an error stating "WARNING: Not enough clusters for a 32 bit FAT!" please ensure the EFIBOOT partition is atleast 270MB in size).
```
- Mount these partitions in the following order:
```
mkdir -p /target/boot

mount /dev/sdb2 /target/boot
mkdir /target/boot/efi

mount /dev/sdb1 /target/boot/efi
```
- Now install grub2:
```
# UBUNTU Users:
grub-install --removable --target=x86_64-efi --efi-directory=/target/boot/efi --boot-directory=/target/boot --bootloader-id=grub --recheck /dev/sdb

# CentOS 7 Users:
grub2-install --removable --target=x86_64-efi --efi-directory=/target/boot/efi --boot-directory=/target/boot --bootloader-id=grub --recheck /dev/sdb
```
- For the third partition (sdb3) I use the fat system because it can be used with Windows machines as well as Linux and Macintosh boxes:
```
mkfs.vfat -F32 -n storage -v /dev/sdb3
```
- Next you will have to copy “unicode.pf2” from your linux system and place it in the grub directory of the usb drive:
```
cp /usr/share/grub/unicode.pf2 /target/boot/grub
```
- Now you just have to create the directory (where you will put your Linux distros) on your /dev/sdb2 device that is mounted as /target/boot and copy your Linux distros there as follows (your iso files will probably be different, so change commands accordingly):
```
mkdir /target/boot/isos
cp ~/Downloads/archlinux-2015.05.01-dual.iso /target/boot/isos/
cp ~/Downloads/gparted-live-0.22.0-1-amd64.iso /target/boot/isos/
...
```
- And finally you must safely unmount the usb device:
```
umount /dev/sdb1
umount /dev/sdb2
umount /dev/sdb3

sync ; echo 1 | tee /proc/sys/vm/drop_caches

udisks --detach /dev/sdb
```
--------

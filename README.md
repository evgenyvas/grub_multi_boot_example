# UEFI GRUB multi boot example
A grub.cfg example of multi iso UEFI boot
**setup your usb**:
--------
- Clean the beginning of USB drive to prevent possible problems:
```
dd if=/dev/zero of=/dev/sdb bs=2M count=2 (make sure your usb device is sdb!)
sync ; echo 1 | tee /proc/sys/vm/drop_caches
```
- Start with offset of 2MB (you should check your drive to see what the optimal offset is)
Create a new disklabel of GPT type using parted: (use extreme caution; parted could destroy your linux installation if you are careless)
```
parted -a optimal /dev/sdb (make sure drive you are working on is sdb)
mklabel gpt
unit MB
mkpart BOOTEFI fat32 0% 256M   (creates the EFI partition which is 200M in size)
set 1 boot on
mkpart bootiso ext4 256M 7.5G  (creates a second partition of 7.5 GB)
mkpart private ext4 7.6G 100%  (creates a third partition which will take up the rest of the drive)
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
grub-install --removable --target=x86_64-efi --efi-directory=/target/boot/efi --boot-directory=/target/boot --bootloader-id=grub --recheck /dev/sdb
```
- For the third partition (sdb3) I use the fat system because it can be used with Windows machines as well as Linux and Macintosh boxes:
```
mkfs.vfat -F32 -n SHARED -v /dev/sdb3
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

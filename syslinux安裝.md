# Syslinux安裝
## 建立磁碟分區
假設在/dev/sdb
```
umount /dev/sdb*
parted /dev/sdb --script mktable gpt
parted /dev/sdb --script mkpart EFI fat16 1MiB 10MiB
parted /dev/sdb --script mkpart live fat16 10MiB 3GiB
parted /dev/sdb --script mkpart persistence ext4 3GiB 100%
parted /dev/sdb --script set 1 msftdata on
parted /dev/sdb --script set 2 legacy_boot on
parted /dev/sdb --script set 2 msftdata on
```
格式化
```
mkfs.vfat -n EFI /dev/sdb1
mkfs.vfat -n LIVE /dev/sdb2
mkfs.ext4 -F -L persistence /dev/sdb3
```
掛載分區
```
mkdir /tmp/usb-efi /tmp/usb-live /tmp/usb-persistence /tmp/live-iso
mount /dev/sdb1 /tmp/usb-efi
mount /dev/sdb2 /tmp/usb-live
mount /dev/sdb3 /tmp/usb-persistence
mount -oro live.iso /tmp/live-iso
```
安裝grub
```
grub-install /dev/sdb --no-uefi-secure-boot --removable --target=x86_64-efi --boot-directory=/tmp/usb-live/boot/ --efi-directory=/tmp/usb-efi
```
安裝gptmbr.bin
```
dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/mbr/gptmbr.bin of=/dev/sdb
```
安裝syslinux
```
syslinux --install /dev/sdb2
```
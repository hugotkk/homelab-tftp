## TFTP Server for PXE Boot

I manage my homelab using Proxmox.

VM are created with GRUB2 as the bootloader, utilizing UEFI with tftp server on dnsmasq.

However, I had to resort to legacy boot for my XCP-ng installation since I encountered issues with UEFI.

RAM disk and kernel are extracted from the ISO and were not committed to the repo.

To extract the RAM disk and kernel:
```
mount -o loop <image>.iso /mnt
cp /mnt/xxxx /srv/tftp/xxxx
```

Folder structure
```
tree /srv
```

```
└── tftp
    ├── EFI
    │   └── BOOT
    │       └── grub.cfg
    ├── grubx64.efi
    ├── harvester-1.2.1
    │   ├── initrd
    │   └── vmlinuz
    ├── mboot.c32
    ├── menu.c32
    ├── pxelinux.0
    ├── pxelinux.cfg
    │   └── default
    ├── rocky-9.3
    │   ├── initrd
    │   └── vmlinuz
    ├── ubuntu-22.04
    │   ├── initrd
    │   └── vmlinuz
    └── xcp-ng-8.3.0
        ├── install.img
        ├── vmlinuz
        └── xen
```

The grubx64.cfg file was generated using the latest Debian image from Docker. 

This choice was made because most of the grubx64.efi files did not support booting grub2 config based on MAC addresses.

The legacy boot method allows us to boot the menu config based on MAC addresses, enabling us to manage various machines with different menus for installation. 

This feature is available in CentOS 7 and Debian, but the CentOS 7 version lacked multiboot2 support. 

To address this limitation, I rebuild the grubx64.efi with modules that CentOS 7 has and then add multiboot2 support.

The resulting file size is approximately 1.2MB.

prepare the modules needed
```bash
cat /tmp/modules
```

```
acpi
all_video
at_keyboard
backtrace
bitmap_scale
boot
btrfs
bufio
cat
chain
configfile
crypto
datetime
disk
diskfilter
echo
efi_gop
efi_uga
efifwsetup
efinet
ext2
extcmd
fat
font
fshelp
gcry_sha512
gettext
gfxmenu
gfxterm
gzio
halt
hfsplus
http
iso9660
jpeg
keylayouts
linuxefi
loadenv
loopback
lvm
lzopio
mdraid09
mdraid1x
minicmd
mmap
multiboot
multiboot2
net
normal
part_apple
part_gpt
part_msdos
password_pbkdf2
pbkdf2
png
priority_queue
reboot
search
search_fs_file
search_fs_uuid
search_label
serial
sleep
syslinuxcfg
terminal
terminfo
test
tftp
trig
usb
usbserial_common
usbserial_ftdi
usbserial_pl2303
usbserial_usbdebug
video
video_bochs
video_cirrus
video_colors
video_fb
xfs
lsmmap
lsefimmap
linux
relocator
bitmap
```

build the custom grubx64 image
```bash
docker run -it --rm -v /tmp:/tmp debian sh
apt update
apt install -y grub-efi-amd64-signed
grub-mkimage --output=/tmp/grubx64.efi -O x86_64-efi -p /EFI/BOOT $(cat /tmp/modules)
```

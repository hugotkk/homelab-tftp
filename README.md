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

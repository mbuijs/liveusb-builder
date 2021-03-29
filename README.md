# liveusb-builder

liveusb-builder is a script suite to create multiboot USB stick for GNU/Linux distributions. It's lightweight with few dependencies, and it's extensible and hackable.

The source code is hosted on both [git.wehack.space](https://git.wehack.space/liveusb-builder/) and GitHub.

A presentation in Chinese is at https://wehack.space/vimacs/liveusb-builder.odp.

## Features

- Multiboot support with syslinux and GRUB
- Support legacy BIOS, UEFI, and various firmware payloads (e.g. GRUB, petitboot) that reads GRUB or syslinux config files
- Support placing kernel files (kernel and initramfs) and other data files (squashfs, CD image) in separate partitions
- Download an up-to-date CD image and verify it
- A GNU/Linux command line tool

## Install

You need these packages on your GNU/Linux system to use liveusb-builder.

- libarchive: for ISO image extraction
- udevil: for mounting USB disk partitions
- wget: for downloading
- syslinux (recommended): bootloader for legacy BIOS
- GRUB: bootloader for legacy BIOS if there's no syslinux, and bootloader for UEFI

For Arch Linux users, just install [liveusb-builder-git](https://aur.archlinux.org/packages/liveusb-builder-git/) from AUR.

## Will my boot image support liveusb-builder?

liveusb-builder needs a kernel image, an initramfs image, and a proper kernel command line to support a boot image. However, not every CD/DVD image supports booting this way. There are some other ways to boot these images, but they will break our philosophy to support various firmware and bootloaders.

If you want to use a disk image to install an operating system, I suggest using the virtual machine method as follows. Boot an AMD64 machine with enough memory and VMX feature with a Live USB with QEMU GUI (e.g. Grml 2020.06, also see [my ticket](https://github.com/grml/grml-live/issues/71) for Grml) after attaching the hard disk you want to install the OS on, then run the following (assume the hard disk attached is /dev/sda):

```bash
sudo qemu-system-x86_64 -enable-kvm -m 4G -cdrom your-install-cd.iso -drive /dev/sda,format=raw -boot order=d -no-reboot
```

Do the system installation in the QEMU window. After QEMU exits, put the hard disk on the target machine to continue installing.

I've already used this method to successfully install Windows 7, OpenBSD 6.7, and various GNU/Linux systems.

## Usage

### The easier way: one FAT32 partition

First mount your USB drive partition. I recommend using udevil so that you can write files without as root.

Then run buildlive script as follows, suppose your USB is /dev/sdb and /dev/sdb1 is mount to /media/sdb1:

```bash
# install Arch, Mint (x86_64 with MATE Desktop) and Fedora 32 to USB
./buildlive --root=/media/sdb1 arch mint/mate fedora/32
```

### The more customizable way: using a FAT32 boot partition and an ext2 data partition

Partition your disk as follows to create a 500MB FAT32 boot partition, and an ext2 partition using the remaining space, suppose your USB is /dev/sdb.

```
$ sudo fdisk /dev/sdb

Command (m for help): o
Created a new DOS disklabel with disk identifier 0x24c5dd70.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-30463999, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-30463999, default 30463999): +500M

Created a new partition 1 of type 'Linux' and of size 500 MiB.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): b
Changed type of partition 'Linux' to 'W95 FAT32'.

Command (m for help): a
Selected partition 1
The bootable flag on partition 1 is enabled now.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (1026048-30463999, default 1026048):
Last sector, +sectors or +size{K,M,G,T,P} (1026048-30463999, default 30463999):

Created a new partition 2 of type 'Linux' and of size 14 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Format the partitions:

```
sudo mkfs.msdos -F 32 /dev/sdb1
sudo mkfs.ext2 /dev/sdb2
```

Mount them and create the directory, and we need to make the directory ``liveusb-data`` writable by the current user.

```
udevil mount /dev/sdb1 /media/boot
udevil mount /dev/sdb2 /media/root
sudo install -d /media/root/liveusb-data
sudo chown $(whoami) /media/root/liveusb-data/
```

At last, make the Live USB (we install Arch and Fedora 32 in it):

```
$ ./buildlive --boot /media/boot --root /media/root arch fedora/32
```

## Experimental features

I've made some experimental features that are not yet merged to the main source tree.

- [metaiso](https://github.com/mytbk/liveusb-builder/tree/metaiso): support using a custom iso file for a certain distro
- [ia32efi-test](https://github.com/mytbk/liveusb-builder/commits/ia32efi-test): support IA32 EFI

## Status

The resulting USB stick works on QEMU with PC BIOS (SeaBIOS), UEFI (OVMF), libreboot (i440fx, GRUB txtmode) as firmware.

## Related work

You can search keyword ``multiboot`` on GitHub and find some related projects. Listed below is some related work I know or find.

- [Yumi](https://www.pendrivelinux.com/yumi-multiboot-usb-creator/): a Windows GUI multiboot USB builder, I need a similar tool that runs on GNU/Linux, so I created this project
- [aguslr/multibootusb](https://github.com/aguslr/multibootusb): provides grub.cfg files for many CD images, I used some of the kernel command line of distros in this project, but some grub.cfg files use features provided by GRUB and thus not portable for loaders like syslinux
- [Ventoy](https://www.ventoy.net): a new multiboot USB project that supports making a live USB with most of the CD images. It has its own boot loader so doesn't rely on GRUB or syslinux config files.
- [MultiBootLiveUSB](https://github.com/moontide/MultiBootLiveUSB)
- [Multiboot USB drive - ArchWiki](https://wiki.archlinux.org/index.php/Multiboot_USB_drive)
- [cbodden/multiboot](https://github.com/cbodden/multiboot)
- [mbusb/multibootusb](https://github.com/mbusb/multibootusb)


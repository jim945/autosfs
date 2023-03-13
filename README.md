# autosfs
Config GRUB2 for loading systems from sfs containers using loopsfs.cfg

Autosfs is designed to search for and automatically start operating systems from sfs files from GRUB2.
This script looks for .sfs files in the /bootsfs directory containing the configuration file /boot/grub/loopsfs.cfg

https://archlinux.org.ru/forum/topic/19907/

## About linux in sfs container

### Background
Somehow I got hooked on the idea that distributing Linux assemblies in the iso format is redundant at the moment and only adds problems.
ISO files were relevant in the days of optical discs and eventually adapted to work on flash drives.
By copying the image to a flash drive (dd if of), we get an analogue of CD / DVD, losing all information on the media.
Later there were crutches for loading iso from the file system. autoiso is an example.

### Idea
Get rid of the extra layer.
All linux isos have a squashfs compressed root filesystem.
Why not distribute assemblies directly in sfs, getting rid of unnecessary work with the bootloader, repacking to iso and gaining a little in size?

## Usage

1. Next to your grub.cfg, create an autosfs directory and drop autosfs.cfg into it

2. To start, use the line in your grub.cfg, correcting the path to the config with your own, if different.

```
submenu "autosfs" {
configfile /boot/grub/autosfs/autosfs.cfg
}
```

3. On any partition, create the /bootsfs directory

4. We put the squashfs image with the OS into it. Required with sfs extension

5. Done


## loopsfs.cfg

The config is used as a label that the sfs container is bootable.
When transferring control, two variables are passed to it for the correct detection of the container by the sfs system:

$sfs_dev - stores disk, partition number in GRUB format (hd1,gpt5)

$sfs_path - path to the container (/bootsfs/rx-20-06-06-1.sfs)

The $root variable is set to the root of the current container.

### Sample loopsfs.cfg from the test image.

```
menuentry "ArchLinux sfs" {
probe -s root_uuid -u $sfs_dev                   #Find the UUID of the $sfs_dev partition
regexp -s sfs_name '^.*/(.*)$' "$sfs_path"       #Split the $sfs_path variable into the image name
regexp -s sfs_dir '^/(.*)/.*$' "$sfs_path"       #and the path to it.

linux /boot/vmlinuz-linux \                      #Load the kernel
root=live:/dev/disk/by-uuid/$root_uuid \         #from the $sfs_dev partition
rd.live.dir=$sfs_dir \                           #correct directory
rd.live.squashimg=$sfs_name \                    #and with the correct name
rd.live.image

initrd \
/boot/amd-ucode.img \
/boot/intel-ucode.img \
/boot/initramfs-linux.img
}
```

## Test image with ArchLinux and bootloaders for EFI and BIOS with built-in bootsfs

https://mega.nz/folder/l1Z0BKwS#XP3b9wR5lVGdBDiyZ9axeg

An example of a live image with ArchLinux.

Contains xfce4, firefox and many other graphical and console utilities.

You can remove, install, update packages in an image with a regular pacman. But changes are stored only in RAM.

To rebuild the image, run the repackrootsfs command as root.


### Install

Copy all files and directories to a flash drive with fat32.

EFI bootloader is detected automatically.

To install the bootloader BIOS, run the installbiosbldr file on the flash drive as root. (Do not run the installbiosbldr file from the disk partitions on which the main operating system is installed!!!!)

```
sh installbiosbldr
```

The system must have GRUB2 installed. Installation is carried out by its regular means.

Login/password: user/123 root/321



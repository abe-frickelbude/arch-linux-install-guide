# Arch Linux Install Guide

## Basic

**Assumes the machine is running in UEFI mode!**

1. Boot from Archlinux ISO and wait until the root prompt appears
   * Be aware that according to <https://wiki.archlinux.org/index.php/VirtualBox>
    booting inside VBox may initially take a couple of minutes after the initial boot menu
    for the system to display anything - i.e. the blank screen at the beginning is normal!
   * Check that there's an internet connection (e.g. ping google.com should work)

2. Set timezone / clock
   1. `timedatectl set-ntp true`
   2. `timedatectl status` should show a valid time/zone
  
3. Set keyboard map (if different from default)
   TODO

4. Create partitions
   1. `fdisk -l` to show available devices
   2. e.g. `fdisk /dev/sda` to work with /dev/sda
   3. `g` for new GPT partition table
   4. `n` for creating new partitions, at least two are needed: ~400MB for EFI boot and "the rest" for the root i.e. `/`
    The syntax for quickly specifying sizes is like `+15G` would mean `15 GBytes` from whatever "first sector" might be
   5. `t` to switch partition types -> `l` shows a list of available partition types. Type for EFI boot is `1`, for root it is `24`
   6. `w` to finalize and write out the new partitions.

5. Create filesystems
   1. `mkfs.msdos -F32 /dev/<sdX for the EFI boot partition>` to create FAT32 on the EFI boot partition
   2. `mkswap /dev/<sdX for the swap partition>` if using a swap partition followed by `swapon`(possibly a bad idea if installing on an SSD)
   3. ``mkfs.ext4 ./dev/<sdX for the root partition>`` to create EXT4 on the root partition

6. Temporarily mount file systems for installation
7. Bootstrap installation
8. Create `fstab`
9.  TODO setup systemd-boot

## Adding full system encryption with dmcrypt and LUKS

## Using btrfs

## Misc

## Links, Tutorials etc

<https://www.youtube.com/watch?v=2zciJYPwUWQ>

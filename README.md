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
   1. ``fdisk -l`` to show available devices
   2. e.g. ``fdisk /dev/sda`` to work with /dev/sda
   3. ``g`` for new GPT partition table
   4. ``n`` for creating new partitions, at least two are needed: ~400MB for EFI boot and "the rest" for the root i.e. `/`
        * The syntax for quickly specifying sizes is like `+15G` would mean ``15 GBytes`` from whatever "first sector" might be
        * 
5. Create filesystems
6. Temporarily mount filesystems for installation
7. Bootstrap installation
8. Create fstab
9.  TODO setup systemd-boot


## Adding full system encryption with dmcrypt and LUKS

## Using btrfs

## Misc

## Links, Tutorials etc

<https://www.youtube.com/watch?v=2zciJYPwUWQ>

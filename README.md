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
   7. For the further steps below, the example is going to assume that the EFI boot partition resides on `/dev/sda1`, the swap partion - on
     `/dev/sda2` and the root partition - on `/dev/sda3`

5. Create file systems
   1. `mkfs.msdos -F32 /dev/sda1` to create FAT32 on the EFI boot partition
   2. `mkswap /dev/sda2` if using a swap partition followed by `swapon`(possibly a bad idea if installing on an SSD)
   3. `mkfs.ext4 ./dev/sda3` to create EXT4 on the root partition

6. Temporarily mount file systems for installation

    1. Note: mounting partitions into `/mnt` may look a little weird, but is only during the installation process (because pacstrap needs a target to work with), and will **not** affect the final `fstab` mount points in any way!
    2. `mkdir /mnt/boot`
    3. `mount /dev/sda1 /mnt/boot`
    4. `mount /dev/sda3 /mnt`

7. Bootstrap installation
   * `pacstrap /mnt base` will initiate installation of base system packages into `/` and `/boot` currently mounted under `/mnt`
  
8. Create `fstab`
   1. `genfstab -U /mnt >> /mnt/etc/fstab`
   2. `less /mnt/etc/fstab` to make sure `fstab` was generated OK
   3. Take a screenshot of `fstab` contents (or write down the mapping for the root partition, especially the partition UUID)

9. Basic configuration
   1.  `arch-chroot /mnt` change root to /mnt
   2.  `ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime` to set the local timezone
   3.  `hwclock --systohc` to set the hardware clock
   4.  `vi /etc/locale.gen` and then uncomment the en_US.UTF-8
   5.  `locale-gen` to generate the locale
   6.  `vi /etc/locale.conf` and add the line `LANG=en_US.UTF-8`

10. Network configuration
    1.  `vi /etc/hostname` and add e.g. `arch-machine`
    2.  `vi /etc/hosts` and the lines ``127.0.0.1   arch-machine`, `::1 arch-machine`, and `127.0.1.1   arch-machine.localdomain  arch-machine`

11. Security configuration
    1.  `passwd` and set root password (do NOT disable the root account just yet!)
    2.  Install vim by ``pacman -S vim`
    3.  Install sudo by `pacman -S sudo`
    4.  Run `visudo`, look for the line `%wheel ALL=(ALL) NOPASSWD: ALL`, uncomment it and save the changes

12. Configure systemd-boot
    1.  Install `systemd-boot` via `bootctl --path=/boot install`
    2.  `cd /boot/loader`
    3.  Edit `loader.conf`, change `default` to `arch-*`, adjust timeout as desired
    4.  `cd /boot/loader/entries`, create a file called `arch.conf`
    5.  Edit `arch.conf` and add the following lines:
        *   `title   Arch Linux`
        *   `linux   /vmlinuz-linux`
        *   `initrd  /initramfs-linux.img`
        *   `options root=UUID=[UUID saved in step 8.3] rw`

## Adding full system encryption with dmcrypt and LUKS

## Using btrfs

## Misc

## Links, Tutorials etc
* <https://wiki.archlinux.org/index.php/installation_guide>
* <https://gist.github.com/kevinwright/6884737>
* <https://www.youtube.com/watch?v=2zciJYPwUWQ>
* Encrypted Arch Setup: <https://www.youtube.com/watch?v=rT7h62OYQv8>
* Manual mounting of encrypted partitions: <https://evilshit.wordpress.com/2012/10/29/how-to-mount-luks-encrypted-partitions-manually>

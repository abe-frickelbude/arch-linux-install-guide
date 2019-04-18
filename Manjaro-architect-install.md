
# Manjaro Architect-assisted installation with encrypted root FS 

## Prerequisites

* Previous Linux experience is **required** - step descriptions below may omit trivial stuff i.e. it is assumed 
  you can do basic CLI work and know your way around a text-mode GUI
* Manjaro Architect ISO image (use e.g. RUFUS to create a bootable USB stick from it)

## Notes

* The setup assumes a very simple partitioning scheme with one "big" root partition using the EXT4 file system type
* No LVM is used for the setup i.e. the root partition is "real"
* Any usage of `pacman`, `systemctl` and other administrative utilities assumes you are either root or use `sudo`
* When in installation chroot, use `exit` followed by `fg` to return to the installer GUI
* Make sure to follow the installation steps in the suggested order to avoid unnecessary trouble
* In a system containing more than one physical disk, it is important to keep track of the `/dev/sdX` partition
  mappings during the installation to not accidentally overwrite existing data during partitioning/mounting steps!

## Setup steps

* Boot into Manjaro Architect Live ISO
* Log in with `manjaro:manjaro` and start `setup`  
* [1] **Prepare installation**
  1. `Set virtual console` - use this to set the desired keymap (default is en_US.UTF-8)
  2. `Partition disk` -> select device and use automatic partitioning as suggested by the installer. Afterwards, use the `List devices` menu
     item to examine the partition descriptors, you'll need them for the "mount partitions" step below.
  3. `LUKS encryption` -> use "Automatic LUKS encryption", select the "/" partition, leave the default "cryptroot" 
     device mapper mapping and pick a passphrase. A few moments later, a box should appear saying "Done!". Go back
  4. Skip `[5] Logical Volume Management` and `[6] ZFS` and go straight to `[7] Mount partitions`
  5. `[7] Mount partitions` - The installer will ask for specific partitions to pick, observe the box descriptions carefully! 
     1. Hit "ok" in the warning window
     2. Pick the root partition, this should be the `/dev/mapper/cryptroot` entry
     3. Pick the `btrfs` file system, answer `Yes` in the warning box after that, tuff off the `autodefrag` flag, turn on the `ssd` flag if installing on a SSD in the box after that with `OK` and confirm
     4. Optionally select the SWAP partition and use the default suggested size
     5. In the `Select additional partitions` box, pick `Done -` and hit `OK`
     6. In the `Select UEFI partition` box, pick the UEFI partition created in step `[2]` above, confirm format, and in the next box
        be sure to pick the `/boot` mount point instead of `/boot/efi` as this is required for proper operation of `systemd-boot`
  6. Skip `[8] Configure mirror list` and select `[9] refresh pacman keys` - this will take a while and automatically return you 
     back to the menu upon completion
  7. Pick `[11] go back` to return to the main installer menu
   
* [2] **Install desktop system**
  1. Pick `Install desktop system`.
  2. Pick `yay + base-devel` as well as `linux51` (as of this writing)
  3. Pick at least `kernel-headers` on the next screen
  4. Pick the desired desktop environment on the next screen
  5. Answer `no` to additional packages question
  6. Answer `full` to the `full` vs `minimal` question
  7. Wait for the installation to complete - this takes a while and will generally have to download ~ 1 GB data
  8. After that, a screen will automatically come up and ask to install free/proprietary graphics drivers - select according to your system

* [3] **Install bootloader**
  * Select `systemd-boot` from the available bootloaders and confirm with `Yes` on the next screen

* [4] **Configure base**
  1. `Generate FSTAB` - select the second option (device name)
  2. `Set hostname` - pick a hostname
  3. `Set system locale` - select and set an appropriate locale,  e.g. `en_US.UTF-8`
  4.  `Set desktop keyboard layout` - ditto
  5.  `Set timezone and clock` - pick a timezone, and set the clock either to UTC or localtime
  6.  `Set root password` - this is the password for the root user (which is not often used, but pick a password just in case)
  7.  `Add new user(s)` - Use this to create a new non-root user for yourself. You have a choice of username, password and the default shell
  8.  Go back one level

* [5] **Review configuration files** (item `[4 System tweaks]` can be skipped for now) 

---
**This is one of the most critical puzzle pieces not described adequately by the ArchLinux wiki and Manjaro documentation!**
**Do not reboot yet!** 

(If you do accidentally reboot before this point, the steps below are still possible, but will require booting the live Architect
system and manually mounting the EFI boot partition to gain access to the config files)

Apparently, the Manjaro Architect installer contains a bug (or the usage of the installer options is counter-intuitive enough to make it
look like a bug) which prevents it from correctly constructing the loader entry files for `systemd-boot` when using 
dmcrypt / LUKS encryption.

The following is a series of steps to rectify this problem. As an example, the instructions assume the EFI partition is on `/dev/sda1`
while the encrypted LUKS partition is on `/dev/sda2`

---

  1. Pick `systemd-boot` from here, this will open the loader configuration entries in Nano
  2. In Nano, edit the `options` line to match this: `options rw cryptdevice=/dev/sda2:cryptroot root=/dev/mapper/cryptroot` 
  3. Use `Ctrl-O` followed by `Ctrl-X` to write out changes and exit Nano
  4. The installer will automatically open further systemd-boot config entries - repeat (3) until done
  5. Go back to the main installer menu
---

* [6] **Done**
  * At this point, if everything worked correctly, there should be one last screen "Close installer?", answer with "yes"
  * Now you can do `sudo reboot now` to restart the system 

After reboot in a few moments you should see the passphrase prompt provided by the initrd `encrypt` hook and be able
to unlock the root partition with the previously picked pass phrase

## Post-install

---

## Activate network interfaces and install NetworkManager

If network is down after reboot (especially in a CLI system), do the following:

* Use `ip link` to see the status of network interfaces, look for e.g. `eth0` (or enp0s3 or similar if in a VirtualBox VM)
  and see if the status is "DOWN". 
* Do `systemctl enable systemd-networkd.service` and `ystemctl start systemd-networkd.service`
* Now do `dhcpcd` and wait for it to initialize
* Using `networkctl` confirm that the ethX interface is now "routable" (alternatively, the output of `ip link show dev ethX` should
  now contain "UP" in the <...>)

## Install NetworkManager for automatic network connection management:

* Use `pacman -S networkmanager` to install the Gnome network manager
* Use `systemctl enable NetworkManager` to activate the NetworkManager systemd service

## mkinitcpio.conf changes

* Move the `keyboard` hook _before_ of the `encrypt` hook as described here <https://wiki.archlinux.org/index.php/Dm-crypt/Swap_encryption>
  to ensure support for a USB keyboard!
* Don't forget to regenerate initramfs after any configuration changes!

## Additional packages

* `pacman -S terminus-font` for extra console fonts (required by the `consolefont` hook in `initramfs`)

## Optional: modifications to systemd-boot configuration

* `pacman -S vim` unless already installed
* Edit the `/boot/loader/loader.conf`, set the timeout as desired 
* You can also adjust the default entry to `manjaro-*`, this will instruct `systemd-boot` to automatically pick up all entries 
  prefixed with "manjaro" (useful if installing multiple kernels)

## Optional: adjust virtual console font

* Edit the `etc/vconsole.conf` file to set a different console font. See in `/usr/share/kbd/consolefonts` for a list of available fonts.

## Optional: add plymouth for pretty startup / LUKS password screen

Note: once again facing some tomfoolery here... The `plymouth` package contains a installation script that will automatically
use `mkinitcpio` to rebuild the initramfs image(s), but the script is incomplete - it does not add the plymouth hooks to the
`mkinitcpio.conf` file. These have to be added manually and the image rebuild again for the plymouth integration to be successful!

* `pacman -S ttf-dejavu` for DejaVu TTF fonts (required by the `plymouth / plymouth-encrypt` hooks)
* Follow this setup guide <https://wiki.archlinux.org/index.php/plymouth>

## Recommended: use block device UUIDs for partitions

* Reasons are outlined here: <https://wiki.archlinux.org/index.php/Persistent_block_device_naming>
* Use `sudo blkid` to display block device UUIDs
* Note that for all configuration purposes (e.g. for kernel options) the `UUID` value is required, not the `PARTUUID` one!

## Optional: encrypted SWAP partition

* The following discussion on stackexchange provided the correct answer: 
<https://serverfault.com/questions/312123/how-to-create-a-randomly-keyed-encrypted-swap-partition-referring-to-it-by-uu>
* See this <https://bbs.archlinux.org/viewtopic.php?id=183045> 
  and this <https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#Using_sd-encrypt_hook> for correctly setting up 
  the systemd hooks in mkinitcpio.conf
* Note that in order for the crypttab to be properly processed by systemd boot, the kernel options in the boot loader
  configuration **MUST** be prefixed with `rd.`, otherwise everything in `/etc/cryptab` will be ignored!   


## Caveats

Some of these are assumed to be Manjaro-specific until proven otherwise (i.e. if "stock" Arch Linux would also exhibit the
same behavior).

### Kernel upgrades

* Can potentially produce initramfs images with broken VFAT filesystem support, resulting in the inability to mount the
  EFI partition on startup...

### Systemd-Boot

* There's probably a bug in the current _Manjaro_ implementation of the post-install hook that updates the systemd-boot entries.
  Upon installation of new kernels, this leads to _all_ entries in `/boot/loader/entries` being broken due to incorrect syntax
  for the kernel `options` line w.r.t to dmcrypt/LUKS. See the above section `[5] Review configuration files` for manually fixing
  the entries.  Also, one can simply make a backup of all existing entries + loader.conf and simply copy them back to the boot
  partition once a new kernel is installed.
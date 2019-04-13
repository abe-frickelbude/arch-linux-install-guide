
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
     3. Pick the `ext4` file system, answer `Yes` in the warning box after that, accept the default flags in the box after that with `OK` and       confirm
     4. Optionally select the SWAP partition and use the default suggested size
     5. In the `Select additional partitions` box, pick `Done -` and hit `OK`
     6. In the `Select UEFI partition` box, pick the UEFI partition created in step `[2]` above, confirm format, and in the next box
        be sure to pick the `/boot` mount point instead of `/boot/efi` as this is required for proper operation of `systemd-boot`
  6. Skip `[8] Configure mirror list` and select `[9] refresh pacman keys` - this will take a while and automatically return you 
     back to the menu upon completion
  7. Pick `[11] go back` to return to the main installer menu
* [2] **TODO Install system**      


## Additional packages

* `pacman -S terminus-font` for extra console fonts (required by the `consolefont` hook in `initramfs`)

## Modifications to systemd-boot configuration

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

From the menu, select "chroot into installation" and do the following:

* `pacman -S vim` unless already installed
* Edit the `/boot/loader/loader.conf`, set the timeout as desired
* Edit _at least_ the `/boot/loader/entries/manjarolinux[X].conf` where `[X]` corresponds to the `default` entry in `loader.conf`
  and modify the options line as follows:<br/><br/> 
  `options rw cryptdevice=/dev/sda2:cryptroot root=/dev/mapper/cryptroot`
* Save the config file, reboot, and after a few moment you should see the passphrase prompt provided by the initrd `encrypt` hook

---

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

## Optional: adjust virtual console font

* Edit the `etc/vconsole.conf` file to set a different console font. See in `/usr/share/kbd/consolefonts` for a list of available fonts.

## Optional: add plymouth for pretty startup / LUKS password screen

Note: once again facing some tomfoolery here... The `plymouth` package contains a installation script that will automatically
use `mkinitcpio` to rebuild the initramfs image(s), but the script is incomplete - it does not add the plymouth hooks to the
`mkinitcpio.conf` file. These have to be added manually and the image rebuild again for the plymouth integration to be successful!

* `pacman -S ttf-dejavu` for DejaVu TTF fonts (required by the `plymouth / plymouth-encrypt` hooks)
* Follow this setup guide <https://wiki.archlinux.org/index.php/plymouth>
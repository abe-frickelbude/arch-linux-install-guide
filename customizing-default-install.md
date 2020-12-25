
# Customization of a Manjaro installation originally created by Arhitect

## Original hardware and software configuration

This guide applies specifically to a configuration similar to the following specs:

* OS resides on an NVMe drive
* The root partition is ext4 fully encrypted via `LUKS/dmcrypt`
* The UI is `Cinnamon` as provided by manual installation via Architect
* The guide assumes the system has been originally installed using Manjaro Architect (as outlined e.g. in `Manjaro-architect-install`)

```
For transparency: My PC build is a Ryzen 7-based machine with a 2GB Samsung PCIe SSD and a
Radeon RX 570-based graphics card - i.e. YMMV depending on your actual system. 
```

```
The basic purpose of this guide is to generally describe the pitfalls produced by the current Manjaro installer,
how to fix/improve them, and how to customize the install using "systemd-boot" and some modern boot loader features
```

## Default installation pitfalls / drawbacks

* Uses GRUB (pfui...) `[Full disclosure] - I severely dislike Grub...`
* Partition type UUIDs are technically correct but do not conform to the 'Discoverable partitions specification'
* EFI partition not mounted to recommended mount point and also potentially using a too restrictive umask/fmask 
  (which prevents the systemd-boot `bootctl` utility from accessing it correctly)
* The UI is GNOME (shell), obviously - maybe we want something else...  


## "Wishlist"

* Replace GRUB-based boot with `systemd-boot`, using `systemd` initramfs hook instead of `busybox`
* Nice(ish) input prompt for the LUKS password (possibly using `plymouth` if it actually works...)
* Cinnamon instead of GNOME, using Wayland as compositor

### Nice to have

* Use `systemd-boot` in conjunction with "Discoverable Partitions" to remove dependence on specific parameter
values for the kernel command line in boot loader configuration. This will hopefully make the boot loader config
immune to the "overwrite" bug (if it still exists) in Manjaro Linux kernel manager where during a kernel 
switch/upgrade the kernel command line in **all** /boot/loader/entries/*.conf files is overwritten with a
broken default configuration that removes e.g. the `cryptroot` parameters and produces a non-bootable
system upon next restart.


## Configuring systemd-boot to use automatic partition discovery using the discoverable partitions specification

### Required partition type UUIDs

* The `esp` partition needs to have the type `C12A7328-F81F-11D2-BA4B-00A0C93EC93B`
* The root partition has to be on the same drive (i.e. in the same GPT) as the `esp` and nees the type `4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709`
(assuming you are running on an x86-64 architecture)
* If the root partition is encrypted with LUKS, then the type is `CA7D7CCB-63ED-4C53-861C-1742536059CC` instead


### Preparing partition type UUIDs

#### Find out current types

E.g. using `fdisk` or `gdisk`, use the `i` command to print the detailed information about a partition, the
type UUID will be shown as `Type UUID` or `Partition GUID Code` for `fdisk` or `gdisk`, respectively

#### Adjust types

Again, can use either `fdisk` or `gdisk` for this. Let's assume `fdisk`:

* `sudo fdisk <device>` (in my case it is `/dev/nvme0n1`)
* `p` to see the current partition table and note the partition numbers (necessary for other steps)
* `t` to change partition type, select partition number (the boot one will most likely be `1`)
* For the boot (ESP) partition, use `1` if not already set
* For the root partition, use `23` as type (this corresponds to `Linux Root x86_64` in the list of known partition types)



## Documentation links

* <https://systemd.io/DISCOVERABLE_PARTITIONS> or <https://www.freedesktop.org/wiki/Specifications/DiscoverablePartitionsSpec>

TODOs:
* Spell/grammar-check the damn thing!

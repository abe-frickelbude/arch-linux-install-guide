
# Configuring systemd-boot to use automatic partition discovery using the discoverable partitions specification

## Required partition type UUIDs

* The `esp` partition needs to have the type `C12A7328-F81F-11D2-BA4B-00A0C93EC93B`
* The root partition has to be on the same drive (i.e. in the same GPT) as the `esp` and nees the type `4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709`
(assuming you are running on an x86-64 architecture)
* If the root partition is encrypted with LUKS, then the type is `CA7D7CCB-63ED-4C53-861C-1742536059CC` instead


## Preparing partition type UUIDs

### Find out current types

E.g. using `fdisk` or `gdisk`, use the `i` command to print the detailed information about a partition, the
type UUID will be shown as `Type UUID` or `Partition GUID Code` for `fdisk` or `gdisk`, respectively

### Adjust types

Again, can use either `fdisk` or `gdisk` for this. Let's assume `fdisk`:

* `sudo fdisk <device>` (in my case it is `/dev/nvme0n1`)
* `p` to see the current partition table and note the partition numbers (necessary for other steps)
* `t` to change partition type, select partition number (the boot one will most likely be `1`)
* For the boot (ESP) partition, use `1` if not already set
* For the root partition, use **TODO**



## Documentation links

* <https://systemd.io/DISCOVERABLE_PARTITIONS> or <https://www.freedesktop.org/wiki/Specifications/DiscoverablePartitionsSpec>
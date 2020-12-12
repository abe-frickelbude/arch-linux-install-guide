
# Configuring systemd-boot to use automatic partition discovery using the discoverable partitions specification

## Required partition type UUIDs

* The `esp` partition needs to have the type `c12a7328-f81f-11d2-ba4b-00a0c93ec93b`
* The root partition has to be on the same drive (i.e. in the same GPT) as the `esp` and nees the type `4f68bce3-e8cd-4db1-96e7-fbcaf984b709`
(assuming you are running on an x86-64 architecture)
* If the root partition is encrypted with LUKS, then the type is `CA7D7CCB-63ED-4C53-861C-1742536059CC` instead




## Find out current partition type UUIDs



## Documentation links

* <https://systemd.io/DISCOVERABLE_PARTITIONS>
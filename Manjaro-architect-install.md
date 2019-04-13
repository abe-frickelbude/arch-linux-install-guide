
#  Encrypted-root installation using Manjaro Architect

## Setup steps

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

* `sudo pacman -S vim` unless already installed
* Edit the `/boot/loader/loader.conf`, set the timeout as desired
* Edit _at least_ the `/boot/loader/entries/manjarolinux[X].conf` where `[X]` corresponds to the `default` entry in `loader.conf`
  and modify the options line as follows:<br/><br/> 
  `options rw cryptdevice=/dev/sda2:cryptroot root=/dev/mapper/cryptroot`
* Save the config file, reboot, and after a few moment you should see the passphrase prompt provided by the initrd `encrypt` hook

## Post-install

### Activate network interfaces and install NetworkManager

If network is down after reboot (especially in a CLI system), do the following:

* Use `ip link` to see the status of network interfaces, look for e.g. `eth0` (or enp0s3 or similar if in a VirtualBox VM)
  and see if the status is "DOWN". 
* Do `sudo systemctl enable systemd-networkd.service` and `sudo systemctl start systemd-networkd.service`
* Now do `sudo dhcpcd` and wait for it to initialize
* Using `networkctl` confirm that the ethX interface is now "routable" (alternatively, the output of `ip link show dev ethX` should
  now contain "UP" in the <...>)

Installing NetworkManager for automatic network connection management:
* Use `sudo pacman -S networkmanager` to install the Gnome network manager
* Use `sudo systemctl enable NetworkManager` to activate the NetworkManager systemd service
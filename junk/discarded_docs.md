


From the menu, select "chroot into installation" and do the following:

* `pacman -S vim` unless already installed
* Edit the `/boot/loader/loader.conf`, set the timeout as desired
* Edit _at least_ the `/boot/loader/entries/manjarolinux[X].conf` where `[X]` corresponds to the `default` entry in `loader.conf`
  and modify the options line as follows:<br/><br/> 
  `options rw cryptdevice=/dev/sda2:cryptroot root=/dev/mapper/cryptroot`
* Save the config file, reboot, and after a few moment you should see the passphrase prompt provided by the initrd `encrypt` hook

# Working with systemd-boot and managing initramFS images with mkinitcpio 

## Configuration presets and systemd-boot loader entry generation 

The overall procedure that generates or cleans up initramfs images and boot loader entries during 
kernel installation/removal is somewhat convoluted. The following applies at least to Manjaro Linux
but likely also to other ArchLinux-based distros and "pure" Arch. 

The process is controlled by four Pacman hooks residing in `/usr/share/libalpm/hooks`:

```
60-mkinitcpio-remove.hook
90-mkinitcpio-install.hook
sdboot-kernel-remove.hook
sdboot-kernel-upgrade.hook
```

The `mkinitcpio` hooks in turn invoke the corresponding shell scripts in `/usr/share/libalpm/scripts`, namely:

```
mkinitcpio-install
mkinitcpio-remove
```

Whereas the `sdboot` hooks utilize a special systemd-boot management utility (also a script) called
`sdboot-manage` which resides in `/usr/bin`.

When a new kernel is installed (e.g. via `Manjaro Settings Manager`):

1. the aforementioned install hook is triggered and uses the `mkinitcpio-install` script to create 
a preset file in `/etc/mkinitcpio.d` directory. 
2. This preset tells the `mkinitcpio` utility the kernel version, which images to produce and
which configuration file to use (default is simply `/etc/mkinitcpio.conf`), along with any custom
kernel parameters embedded _inside_ an initramfs image (not to be confused with the kernel parameters
passed inside a boot loader entry file!)
3. Then the `mkinitcpio` utility is invoked with this preset to generate a initramfs image corresponding to 
the new kernel.
    * The presets are all based on a template contained inside the `/usr/share/mkinitcpio/hook.preset` file
    * All presets reference the `/etc/mkinitcpio.conf` configuration file.
4. After the initramfs image is built, the `sdboot-kernel-upgrade` hook invokes the `sdboot-manage` utility
    which (re-)generates the configuration entries for the `systemd-boot` 
    loader based on the currently installed kernels.
    * The `sdboot-manage` is controlled by `/etc/sdboot-manage.conf` configuration file which specifies
    overall behavior of the utility, as well as some entry parameters such as any special kernel command 
    line options you might have.

### Implications 

* Due to the structure of the preset file template mentioned above in (3) the system cannot accomodate
using multiple different `mkinitcpio` configuration files in its current form, i.e. all presets will ultimately
share the single `/etc/mkinitcpio.conf` file, so any module/hook/etc configuration will be identical for all
generated initramfs images. The only available recourse is to manually write custom presets that reference
individual configuration files.
* The `sdboot-manage` configuration file is currently the only place to supply any custom kernel parameters, e.g.
`quite` and `splash` as required by `plymouth`. This is understandable as only the `sdboot-manage` has the
control over generating the boot loader configuration entries.
* Switching the initramfs image _itself_ from `udev` to `systemd` requires some modifications to the kernel command line
parameters to boot properly, especially on an "encrypted root" system. Unfortunately, the `sdboot-manage` script 
doesn't "know" anything about systemd-based  init and can only generate the required kernel command line parameters 
for "conventional" `udev`-based init scenarios.
    * Specifically there is currently no way to specify the `rd.luks.name=UUID=cryptroot` form required by 
    the `systemd` initramfs hook instead of the conventional `root=UUID` / `cryptdevice=UUID:cryptroot` form.  

## Installing plymouth

## Nuisances and troubleshooting



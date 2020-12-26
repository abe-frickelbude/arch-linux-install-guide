
# Ongoing brain dump

Dislaimer: this is exactly what it says on the box - at this point I've given up creating any kind of structured
guide due to lack of time and constantly shifting "landscape" of my own knowledge of Arch/Manjaro system 
specifics. Therefore this will serve as a living document of my system setup/configuration efforts until a
configuration crystallizes out that proves to be stable and reproducible. The information contained herein
will then be rewritten as an actual (hopefully step-by-step) guide with explanations etc.

**WARNING: Please disregard the documentation in customizing-default-install.md - this doc basically
started "there" but the previous one is no longer current!**


## GUIs - State of the Union

* `Wayland` support is still basically non-existent outside of a "true" Gnome (i.e. Gnome shell) desktop and 
there is currently no sensible way to enable Wayland support on Arch in e.g. Cinnamon without resorting
to unreasonable amount of effort. Also, the resulting system configuration would be quite fragile as
it would require manual installation of Wayland-related packages and adjustment of various low-level 
configuration files. 
  * The above obviously doesn't apply if the Gnome destop is used (as installed by the corresponding
  graphical installer or via Manjaro Architect). In this case, Wayland is supported out-of-the-box and
  is enabled by default in case of AMD or Intel GPUs.  
  * If using an Nvidia GPU, you're basically out of luck for now (there's enough discussion on 
    Reddit/StackOverflow/etc - check those out for a detailed explanation why this is the case)
* `XOrg` is properly supported across all desktop environments and GPU manufacturers

## AMD graphics card control

At this moment, the _comfortable_ way of controlling an AMD graphics card appears to be the
radeon-profile software, see <https://github.com/marazmista/radeon-profile> for details. There are
AUR packages available for both `radeon-profile` and `radeon-profile-daemon`

* In particular this provides a daemon which allows the control UI to run without elevated permissions
* Can run minimized to tray (nice!)
* It is quite simple to set up a custom fan control curve in the UI
* Slight bummer: fan control may occasionally glitch and revert to "auto" - easy to fix in the UI by
  re-applying the custom profile (i.e. the custom control curve is _not_ lost, the software just 
  reverts to the automatic profile)

## mkinitcpio nuisances and troubleshooting

### Configuration presets and boot loader entry generation


### Pitfalls of systemd-based initramfs in conjunction with systemd-boot

Switching the initramfs image from `udev` to `systemd` requires some modifications to the kernel command line
parameters to boot properly, especially on an "encrypted root" system. If using the `systemd-boot` EFI bootloader,
the boot loader entries are managed by the `sdboot-manage` script which (re)generates the said entries every time
a kernel is installed or removed. Unfortunately this script doesn't "know" anything about systemd-based 
init and only generates the required kernel command line parameters for "conventional" `udev`-based
init scenarios.


## Bitz an' piecez


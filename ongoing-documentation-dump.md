
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


## Bitz an' piecez

### Mounting additional stuff including network shares (SMB)

* In general: don't forget the `uid=<used id>,gid=<group id>` in the mount entries in `/etc/fstab`, default for a single 
  user installation will be `1001` for both, so that you have correct access permissions on mounted directories
* Samba - don't bother with the stupid-ass `smb.conf`, there's too much documentation on the net and yet no useful
  HOWTOs, just create an empty `/etc/samba/smb.conf` to keep the CLI utilities happy if you ever use them, e.g. `smbclient`,
  and simply use the typical CIFS mount in `/etc/fstab` to permanently connect an SMB share, e.g. from a NAS.
* Mount stuff into `/mnt`, then symlink the mounts into your home for easier access
* For EXT4: `umask`,`fmask`,`uid`,`gid` parameters are not supported - read this here: 
  <https://unix.stackexchange.com/questions/33330/how-to-make-an-ext4-file-writable-on-mounting-by-a-user-not-root>
  The correct permissions need to be set directly on the mount point, e.g. via `chown -R <user>:<group>` followed
  by a desired `chmod -R 775` or similar

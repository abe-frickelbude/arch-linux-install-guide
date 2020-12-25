
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
* `XOrg` is properly support across all desktop environments and GPU manufacturers


## mkinitcpio nuisances and troubleshooting

## Bitz an' piecez


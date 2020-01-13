# Nvidia Optimus driver setup and configuration

## Basic bits

* Remove the braindead `nouveau` open-source driver (this is often the default installed driver)
* Install the bumblebee driver using either `mhwd` CLI utility or using the "Hardware detection" GUI in `Manjaro Settings Manager`

### Switching from Intel to Nvidia GPU on-demand

* Install the `virtualgl` package - among other things this contains the `glxspheres` demo which can be quickly used
  to test proper operation of the graphics setup
* The simplest form is `optirun <application name>` - this will temporarily switch to Nvidia GPU rendering for the
  specified application
* Recommended: add the "Nvidia Prime GPU display" applet to your task bar to visually monitor the currently active
  GPU
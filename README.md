This is a fork I made to suit my own needs, which is to say map Wiimote controls in a way that works well with Trackmania (once appropriate keybindings are set in Trackmania). You may want to disable the steam overlay to prevent it from coming up when pressing certain buttons on the Wiimote (I find that the '+' button triggers it for me).

To install on Ubuntu Linux, first clone the repository into a directory of your choice and navigate into the cloned repository:

```bash
git clone https://github.com/jasons05/wiimote-pad-trackmania.git
cd wiimote-pad-trackmania
```

Then make sure you have the necessary dependencies installed:

```bash
sudo apt update
sudo apt install xwiimote libxwiimote-dev
```

And finally compile the project:

```bash
make
```

This last step will create an executable called `wiimote-pad`.

If you are on a Linux distribution other than Ubuntu, the process of installing dependencies will potentially differ but the rest should be the same. If your distribution uses the `apt` package manager then the above installation instructions can likely be followed as-is.

When running this utility, a Wiimote must already be connected. To connect a Wiimote, you must first make sure that `xwiimote` is already installed. If you have followed the installation instructions above, this should already be the case. To connect the Wiimote for the first time, go to terminal and run the `bluetoothctl` command which will put you into an interactive session with the `bluetoothctl` utility. In this session, run the command `scan on` to start scanning for devices. Once you have run this command, press the red button under the battery cover of your Wiimote. This will cause the Wiimote to broadcast itself for several seconds, which will be indicated by the Wiimote LEDs flashing. During this time period, take note of the address of the Wiimote (should look something like `01:23:45:67:89:AB` but with random numbers and letters, and can be identified from other devices because it is labeled as Nintendo). This is best done by copying the address to the clipboard. Once the address is copied, pair it by running `pair <address>` (e.g. `pair 01:23:45:67:89:AB` for a hypothetical Wiimote with that address). Once it is paired, connect with `connect <address>`. If this succeeds, the Wiimote should stop flashing and show one lit LED. Once the Wiimote has connected, you should finally run `trust <address>` to trust the Wiimote. At this point, the Wiimote has been successfully connected and `bluetoothctl` may be exited with Ctrl-D.

If you are too slow and the Wiimote gives up connecting, simply run `remove <address>` in your `bluetoothctl` session and start the process all over again (note that the address doesn't change, so you won't have to worry about that again if you copied it down the first time). The Wiimote might have to be in search mode (blinking lights, triggered with the red button under the battery cover) for `bluetoothctl` to be able to remove the device.

If you are doing everything correctly but when running `connect <address>` the Wiimote fails to stop blinking and eventually just turns off, it is most likely because `xwiimote` is not installed. `xwiimote` is required for the computer to successfully connect in a way the Wiimote recognizes.

Once the Wiimote has been successfully connected according to the instructions outlined above, the Wiimote will remember the computer and automatically connect in the future whenever any button on the Wiimote is pressed. Note that this can easily lead to the Wiimote turning on and connecting by accident which can drain the battery, so be careful.

Once your Wiimote has connected, simply run the `wiimote-pad` executable. This executable will display a number from -100 to 100 which indicates your current steering angle. -100 is maximum steering left and 100 is maximum steering right. If you turn off the Wiimote the executable will automatically exit with error code 0. If you do not have a Wiimote connected when attempting to run the executable, it will give the message `no joysticks found` and exit immediately with error code 19. Note that the executable does not run automatically when the Wiimote connects. You will have to run it manually each time.

If you want to change the mapping between Wiimote buttons and emulated gamepad buttons, look at lines 68 to 88 in `wiimote-pad.c`. If you want to change the steering sensitivity, go to line 238 in `wiimote-pad.c`. By default, the steering has a range of 90 degrees (meaning that maximum steering is attained at 45 degrees away from neutral, which is how the code represents it).

The rest of this README is as found in the original repository.

# Wiimote-Pad

This is a small tool to use a Wiimote as a gamepad.

## Introduction

Linux has had built-in support for the Wii Remote (Wiimote for short)
since v3.1, support which was significantly cleaned up and improved
since v3.11. However, the low-level kernel driver exposes each component
of the Wiimote (accelerator, buttons, IR camera), as well as each
extension, as a distinct device, none of which is (fully) functional as
a controller ‘out of the box’ (the ‘buttons’ device —which the driver
calls the controller ‘proper’— does appear as a joystick device to
Linux, since it has a BTN\_A mapping, but it's a device with no axes and
thus not really usable).

A higher level interface (built on top of the Linux driver) is provided
by [xwiimote][], which provides a library for ‘coalesced’ access to the
Wiimote and its extension. As programs need to be designed specifically
to make use of the library, this still doesn't allow an ‘out of the box’
experience.

The purpose of this tool is to allow any application to use a Wiimote
—held sideways— as if it were a standard gamepad, with a 2-axes joystick
(using the accelerometer), and a D-Pad.

## Usage

Associate your Wiimote with your computer (details on how to do this are
not discussed here, but you may want to look at [xwiimote][]'s page for
additional information), then start the program. As long as the program
is running, a virtual controller (called “Nintendo Remote in gamepad
mode”) will be available. Just press Ctrl+C to terminate the program and
‘disconnect’ the virtual controller.

Syntax:

	wiimote-pad [device]

where _device_ is the path to a Linux-created device associated with the
Wiimote (e.g. `/dev/input/js0` or something like that). If no _device_
is specified, the program will look for the first device that it can
associate with and use that.

## D-pad orientation

Since 2017, by default, the D-pad will be oriented in landscape mode (the
arm closest to the A button will be mapped to _right_ rather than
_down_), in accordance to the Wiimote orientation.

This is a break from the previous behavior of the program. If you prefer
the old behavior, you can pass the command-line option `--dpad
portrait`, which will set the D-pad in portrait mode (arm closest to the
A button will be mapped to _down_). You can enforce the new behavior
with `--dpad landscape`, and even specify different D-pad settings for
different Wiimotes, for example:

	wiimote-pad --dpad portrait /dev/input/js0 --dpad landscape /dev/input/js1

will set the `js0` Wiimote with the D-pad in portrait mode, and the
`js1` one with the D-pad in landscape mode.

### Note

`wiimote-pad` is _specifically_ designed to expose the sideways Wiimote
as a gamepad. All other Wiimote uses (especially the ones involving
the infrared (IR) sensor) are outside of its scope. Please refer to the
`xwiimote` and `xf86-input-wiimote` projects for those.

## `udev` rules

This repository also provides a set of `udev` rules to:

* change the group and the permissions of all Wiimote-related devices
  (both the kernel ones and the virtual one created by `wiimote-pad`);
* create descriptive symlinks for the event devices associated with
  Wiimotes.

By default the group assigned to Wiimote devices is `bluetooth`, you
might need to tune it for your system. The group and permission change
is needed so that applications that use the event interface instead of
the joystick interface can still access the Wiimote.

## Requirements

Dependencies for `wiimote-pad` are `libudev` and `libxwiimote`. The
latter should be version 2 or higher.

## Compile

Just running

	make

should work.

If you compiled and built `libxwiimote` yourself, you might
need to fix the include path in the `Makefile` to point to the correct
locations to look for the headers, or run

	make XWIIMOTE=/path/to/xwiimote/sources

instead. By default, aside from standard locations, the `Makefile` will
look for an `xwiimote` source directory in the parent of the
`wiimote-pad` directory.

[xwiimote]: http://dvdhrm.github.io/xwiimote

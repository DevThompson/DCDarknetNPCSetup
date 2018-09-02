# DCDarknetNPCSetup
A guide on setting up a DCDarknet NPC

# Hardware Requirements
Raspberry Pi
A computer
SD card (class 10 recommended)
Keyboard
Mouse
Display
External wifi dongle. Adafruit sells one [here](https://www.adafruit.com/product/1030). More on this later.

# Software Setup

### Install Raspbian
Installation instructions can be found on the Rasbpian site [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

### Confirm everything is up to date
    sudo apt-get update && apt-get upgrade

### Disable built-in wifi and bluetooth
If you're using a Pi3 or similar board that has wifi built in, you'll want to disable it to force the external wifi chip to become wlan0. Edit /boot/config.txt and add

    # Disable on-board
    dtoverlay=pi3-disble-wifi

    # Disable on-board bluetooth
    dtoverlay=pi3-disable-bt

More about overlays [here](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README)


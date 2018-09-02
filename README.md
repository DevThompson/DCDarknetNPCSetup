# DCDarknetNPCSetup
A guide on setting up a DCDarknet NPC


# Setup Steps

### Install Raspbian
Installation instructions can be found on the Rasbpian site [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

### Confirm everything is up to date
    sudo apt-get update && apt-get upgrade

### Disable built-in wifi and bluetooth
Edit /boot/config.txt and add

    # Disable on-board
    dtoverlay=pi3-disble-wifi

    # Disable on-board bluetooth
    dtoverlay=pi3-disable-bt

For more on boot overlays go to https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README 


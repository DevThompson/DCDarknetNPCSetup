# DCDarknetNPCSetup
A guide on setting up a DCDarknet NPC

# Hardware Requirements
* Raspberry Pi
* A computer
* SD card (class 10 recommended)
* Keyboard
* Mouse
* Display
* External wifi dongle. Adafruit sells one [here](https://www.adafruit.com/product/1030). More on this later.

# Software Setup

### Install Raspbian
Installation instructions can be found on the Rasbpian site [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

### Confirm everything is up to date
    sudo apt-get update && apt-get upgrade
    
### Download the DCDarknet source files and install the related dependencies
    git clone --recurse-submodules https://github.com/thedarknet/nodes.git
    cd nodes
    pip install -r requirements.txt

### Disable built-in wifi and bluetooth
If you're using a Pi3 or similar board that has wifi built in, you'll want to disable it to force the external wifi chip to become wlan0. Edit /boot/config.txt and add

    # Disable on-board
    dtoverlay=pi3-disble-wifi

    # Disable on-board bluetooth
    dtoverlay=pi3-disable-bt

More about overlays [here](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README)

### Configure the Pi to be an access point
Thanks to Steven Lovely and his article [here](https://thepi.io/how-to-use-your-raspberry-pi-as-a-wireless-access-point/). This section mostly mimics his post but has a few tweaks specific to our NPC setup.

Install the necessary tools hostapd and dnsmasq.

    sudo apt-get install hostapd
    sudo apt-get install dnsmasq

We need to make some edits to configuration files for these two packages so we should shut them down.

    sudo systemctl stop hostapd
    sudo systemctl stop dnsmasq

The badge looks for a specific IP so we need to set a static one for the NPC. Edit the dhcpcd.conf file with:

    sudo nano /etc/dhcpcd.conf

Add the follow lines:

    interface wlan0
    static ip_address=192.168.4.1
    denyinterfaces eth0
    denyinterfaces wlan0

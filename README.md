# DCDarknetNPCSetup
A guide on setting up a DCDarknet NPC in order to infect your DCDarknet badge.

&nbsp;

# Hardware Requirements
* [Raspberry Pi](https://www.amazon.com/Raspberry-Pi-RASPBERRYPI3-MODB-1GB-Model-Motherboard/dp/B01CD5VC92/ref=sr_1_3?s=pc&ie=UTF8&qid=1536017481&sr=1-3&keywords=raspberry+pi+3)
* A computer
* [Micro SD card](https://www.amazon.com/Sandisk-Ultra-Micro-UHS-I-Adapter/dp/B073JYVKNX/ref=sr_1_4?s=pc&ie=UTF8&qid=1536017509&sr=1-4&keywords=micro+sd+card)
* Keyboard
* Mouse
* Display
* External wifi dongle that supports MAC spoofing such as the [Alfa AWUS036NHA](https://www.amazon.com/Alfa-AWUS036NHA-Wireless-USB-Adaptor/dp/B004Y6MIXS/ref=sr_1_8?ie=UTF8&qid=1536017308&sr=8-8&keywords=alfa). 

&nbsp;

# Software Setup

### Install Raspbian
Installation instructions can be found on the Rasbpian site [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

&nbsp;

### Confirm everything is up to date
    sudo apt-get update && apt-get upgrade

&nbsp;

### Disable built-in wifi and bluetooth
If you're using a Pi3 or similar board that has wifi built in, you'll want to disable it to force the external wifi chip to become wlan0. Edit /boot/config.txt and add

    # Disable on-board
    dtoverlay=pi3-disble-wifi

    # Disable on-board bluetooth
    dtoverlay=pi3-disable-bt

More about overlays [here](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README)

&nbsp;

# Configure the Pi to be an access point
Thanks to Steven Lovely and his article [here](https://thepi.io/how-to-use-your-raspberry-pi-as-a-wireless-access-point/). This section mostly mimics his post but has a few tweaks specific to our NPC setup.

Install the necessary tools hostapd and dnsmasq.

    sudo apt-get install hostapd
    sudo apt-get install dnsmasq

We need to make some edits to configuration files for these two packages so we should shut them down.

    sudo systemctl stop hostapd
    sudo systemctl stop dnsmasq

&nbsp;

### Set a static IP
The badge looks for a specific IP so we need to set a static one for the NPC. Edit the dhcpcd.conf file with:

    sudo nano /etc/dhcpcd.conf

Add the follow lines:

    interface wlan0
    static ip_address=192.168.4.1
    denyinterfaces eth0
    denyinterfaces wlan0

&nbsp;

### Configure the DHCP server
We need to make some changes to a config file so to be safe, let's back it up:

    sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig

Open the config file:

    sudo nano /etc/dnsmasq.conf
    
And add the following:

    interface=wlan0
        dhcp-range=192.168.4.2,192.168.4.99,255.255.255.0,24h

&nbsp;

### Setup the ap software
Open the config file:

    sudo nano /etc/hostapd/hostapd.conf
    
Add the following info:

    interface=wlan0
    hw_mode=g
    channel=6
    wmm_enabled=0
    macaddr_acl=0
    auth_algs=1
    ignore_broadcast_ssid=0
    wpa=0
    ssid=dark

&nbsp;

### Point the system to the location of the config file

    sudo nano /etc/default/hostapd

Locate the line that begins with #DAEMON_CONF=”” and delete the # at the beginning of the line. The line should now appear like this:

    DAEMON_CONF="/etc/hostapd/hostapd.conf"

&nbsp;

### Setup traffic forwarding
When a device connects to our access point we want it to be able to reach the internet. We direct the device to the ethernet port after it connects to our wireless network. Open the config file:

    sudo nano /etc/sysctl.conf

Find this line:

    #net.ipv4.ip_forward=1
    
Delete the # so that the line looks like this:

    net.ipv4.ip_forward=1

&nbsp;

### Setup iptables
Configure IP masquerading for outbound traffic on eth0:

    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Save the iptable rules:

    sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

Set the rule to load on boot by opening rc.local:

    sudo nano /etc/rc.local
    
Locate the line with "exit 0" and add the following just above it:

    iptables-restore < /etc/iptables.ipv4.nat

&nbsp;

### Forward internet traffic
Devices can connect to our access point but they won't have internet. We need to setup a bridge to allow this. Start by installing the necessary tool brctl:

    sudo apt-get install bridge-utils
    
Add a new bridge:

    sudo brctl addbr br0

If you get a message stating that br0 already exists, you will need to delete the bridge and recreate it:

    sudo ifconfig br0 down
    sudo brctl delbr br0
    sudo brctl addbr br0
    
Connect the bridge to the ethernet:

    sudo brctl addif br0 eth0
    
Edit the interfaces file:

    sudo nano /etc/network/interfaces
    
Add the following the bottom of the page:

    auto br0
    iface br0 inet manual
    bridge_ports eth0 wlan0
    
Reboot:

    sudo shutdown -r now
    
Once the pi comes back up, you should have a wireless network called 'dark' with the password 'darknetnpc'. Connect using your phone or laptop and verify that you are able to reach the internet. If not, walk through the steps again to make sure nothing was missed.

&nbsp;

### Spoof the MAC address
The DCDN badge looks for a specific MAC address start with dc:d0 so we need to lie a little bit.

Shutdown the wifi:

    sudo ifconfig wlan0 down

Use macchanger to set the mac address on the wifi chip.

    sudo macchanger --mac=dc:d0:22:33:44:55 wlan0
    
Note the output of this command. You should see an original MAC and a new MAC. Copy the original MAC address as we will need it on the next step.
    
&nbsp;

### Set the MAC address to spoof on boot
Using macchanger only spoofs the mac address until the next boot. Open up 00-default.link:

    sudo nano /etc/systemd/network/00-default.link
    
Add the following and replace ORIGINALMAC with the original MAC address that you obtained from the previous step:

    [Match]
    MACAddress=ORIGINALMAC
    
    [Link]
    MACAddress=dc:d0:22:33:44:55
    
&nbsp;

# Configure the NPC code

&nbsp;

### Download the DCDarknet source files and install the related dependencies
    git clone --recurse-submodules https://github.com/thedarknet/nodes.git
    cd nodes
    pip install -r requirements.txt

&nbsp;

### Configure the Flask app location
When the badge connects to the NPC, it looks for a Flask app running at 192.168.4.1:8080. Make sure you are in the folder that you downloaded the DCDN repo which is most likely ~/nodes/npc. If you chose to download the repo elsewhere, navigate to that folder:

    cd ~/nodes/npc
    
 Open darknet.py to edit:
 
     sudo nano darknetnpc.py
     
The very last line of the file should look similar to the following:

    app.run(host="0.0.0.0", port=int("8080"), debug=True)
    
Edit this line to include the static IP that we set earlier:

    app.run(host="192.168.4.1", port=int("8080"), debug=True)

&nbsp;

# Infect your badge
At this point, the NPC should be setup and ready to spread infections. Program the wifi settings into the badge and then scan for for NPCs. If you run into any issues, revisit the steps above. Feel free to hit me up on [Twitter](https://twitter.com/r00_______).

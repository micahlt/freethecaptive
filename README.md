# ⛓️ FreeTheCaptive

How to hack captive portals and distribute the connection openly to other devices.  

## Disclaimer

Using this software **could** provoke legal action under your jurisdiction.  I take _no responsibility_ of what you do with this program, nor do I endorse it.  It exists purely for educational and explorative purposes.  

## What is a Captive Portal?

We've all been to restaurants, schools, or businesses that had seemingly "open" WiFi networks, only to open a browser and discover that a password is required.  These password pages are called Captive Portals, and while annoying, they are not in the slightest secure.  You can use a technique known as MAC spoofing to bypass them and access the Internet.  

## The way to fix it

Our goal here is to do three things:

- Spoof the MAC address of an existing client on the captive network
- Connect a Linux device (preferably something like a [Raspberry Pi](https://www.raspberrypi.com/)) to the internet
- Share that connection with other devices via NAT bridging

## Getting the tools

### Software

There's been a large amount of research done on this topic, and the tools you need are already out there.  You'll need the following repositories downloaded to the machine that you'll be using to hack and bridge the connection:

- [systematicat/hack-captive-portals](https://github.com/systematicat/hack-captive-portals) to hack the portal
- [oblique/create_ap](https://github.com/oblique/create_ap) to host an access point

You can do a simple `git clone` to get these repos on your Linux device.  You'll also have to install a few dependencies with `apt`:

```bash
sudo apt install -y sipcalc nmap iptables dnsmasq
```

### Hardware

Most network adapters have the ability to recieve and broadcast a network signal simulaneously.  However, some devices (like the Raspberry Pi 3) have [driver issues](https://github.com/oblique/create_ap/issues/203#issuecomment-263063307) that are not compatible with create_ap, meaning that you'll need to get a secondary network card.  Linux-compatible USB network cards are pretty cheap - [here's one that's cheap and small](https://www.amazon.com/wifi-adapter-usb-pc-network/dp/B008IFXQFU).

## Pre-Setup

If you have trouble with network configuration, read this!  I had a Raspberry Pi 3 Model B lying around that was perfect for this project.  However, there are a few things to set up with this specific Pi:

### Install `iptables`

The `create_ap` script uses `iptables`, which does not come installed by default on Raspbian.  To fix this, install it:

```bash
sudo apt install -y iptables
```

### Set up a secondary WiFi card

Like I mentioned earlier, the Raspberry Pi 3 requires a secondary WiFi card.  **Your device may not!**  I got one that did not require drivers and was easily recognized in `lsusb`.  However, I ran into another problem.  By default, `wpa_supplicant` (the network manager on Raspbian) has a generic configuration for all adapters, meaning that when I plugged in the adapter it automatically connected to the same network as the Pi's integrated WiFi radio.  There is a way to change this, though.  The following steps are taken from [this Stack Exchange question](https://raspberrypi.stackexchange.com/a/104717/146567).  

1. **Find your adapter names**

```bash
ifconfig
```

Your adapter names are probably something like `wlan0` and `wlan1`, though they could be different.
  
2. **Copy the `wpa_supplicant.conf`**

```bash
cp /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant-[your first adapter name].conf
cp /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant-[your second adapter name].conf
```

3. **Edit the second adapter**

```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant-[your second adapter name].conf
```
  
Now that you're in the `nano` editor, either delete the "network" block or just change the SSID to an SSID that is not near you.  Use <kbd>Ctrl</kbd> + <kbd>X</kbd> to exit and type <kbd>Y</kbd> when asked to save.

4. **Enable both configuration files**
  
```bash
sudo chmod 600 /etc/wpa_supplicant/wpa_supplicant-[your first adapter name].conf
sudo chmod 600 /etc/wpa_supplicant/wpa_supplicant-[your second adapter name].conf
```
  
5. **Replace old configuration file**
  
```bash
systemctl disable wpa_supplicant.service
systemctl enable wpa_supplicant@[your first adapter name].service
systemctl enable wpa_supplicant@[your second adapter name].service
```

Notice that we're only _disabling_ the original `wpa_supplicant` service.  That means you can still revert the changes if you want to by disabling the individual adapter services and enabling the original service.

## Actual setup

Assuming you're in your home directory and you've cloned both the required repositiories with their original names, you should enter the following commands:

```bash
cd hack-captive-portals
sudo chmod u+x hack-captive.sh
cd ..
cd create_ap
make install
cd ..
```

## Time to hack

Ensure you're connected to the desired network to hack by checking the current SSID in `ifconfig`.  If you are, then you're ready to hack the captive portal.  Run the hacking script: 

```bash
sudo ./hack-captive.sh
```

If you get a result that looks like this:

```
Pwned! Now you can surf the Internet!
```

...that means that you're connected!  You should be able to access the internet on your device.  If that's all you wanted, then stop here.  However, I often find it practical to distribute the connection.

## Distribute the connection

The `create_ap` script is a wonderful tool for easily bridging your WiFi connection to other devices.  First, use the `ifconfig` tool to identify the name of your adapter(s).  Names should be something like `wlan0`, `eth0`, or `wlps0`, though it may vary across devices.

### For single adapter setup

```bash
create_ap [network adapter name] [network adapter name] [your broadcast network name] [your network password]
```

### For dual adapter setup
  
```bash
create_ap [broadcasting adapter name] [recieving adapter name] [your broadcast network name] [your network password]
```
  
## Conclusion

Captive portals are incredibly insecure, and with this amount of minimal effort they can be successfully infiltrated.  Again, I take _zero responsibility_ for the use of this tutorial.  Should you get into trouble with law enforcement or regulations, that's not my fault.  Final disclaimer.

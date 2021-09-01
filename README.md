# Discover Wi-Fi clients using Raspberry Pi Zero W, airodump-ng and Go



[![Sau Sheong](https://miro.medium.com/fit/c/48/48/1*x5p_Ae1Y0AWqpKhOQV8Uog.png)](https://sausheong.com/?source=post_page-----8099be08cc1e--------------------------------)

[Sau Sheong](https://sausheong.com/?source=post_page-----8099be08cc1e--------------------------------)

[Apr 28, 2020](https://medium.com/swlh/scanning-for-mobile-devices-through-wi-fi-using-pi-zero-w-8099be08cc1e?source=post_page-----8099be08cc1e--------------------------------) · 15 min read



Attending conference calls and online meetings for the past couple of months have been [driving me crazy](https://www.cnbc.com/2020/04/27/why-video-chat-during-covid-19-self-quarantine-causes-social-burnout.html). Being stuck in a room, with a few monitors and computers facing you,  then gesticulating to them while clicking on multiple mice and poking at the various keyboards and screens does that to you. My peak activity  was having 2 conference alls, 3 Whatsapp conversations and 4 Slack chats all at the same time.

In all that craziness I’ve also started mucking around with the [Pi Zero W (Pi0W)](https://www.raspberrypi.org/products/raspberry-pi-zero-w/) I got not long after my Pi4. After playing around with the Pi4 for a  bit, I wanted to take a look at something different so I got myself its  little cousin. While the specs are much lower than a Pi4, I really liked the form factor and price and it hardly heats up.

![img](https://miro.medium.com/max/30/1*Rnvd15fVlJB4Bnd3qlwkwQ.jpeg?q=20)

![img](https://miro.medium.com/max/700/1*Rnvd15fVlJB4Bnd3qlwkwQ.jpeg)

Pi Zero W (credits Chang Sau Sheong)

I had a couple of projects in mind, then one day as I was discussing (and gesticulating to myself alone in the room) with some friends who are  looking for solutions to some interesting recent problems, something  popped up. It was an interesting but not uncommon idea. Human [traffic monitoring](https://www.hindawi.com/journals/wcmc/2018/3136471/) and [geofencing](https://www.cio.com/article/2383123/geofencing-explained.html) using Wi-Fi signals sent out by your mobile phones. No one walks around without a smart phone any more (except my mom). And smart phones send  out Wi-Fi broadcast signals like nobody’s business.

This could really work. After all there are several [startups already on it](https://angel.co/indoor-positioning), with serious money too.

Of course, as an engineer, the immediate question comes to mind — can I do that too? I can try. And I have a Pi.

# Setting up

Before we start let’s look what we need. For the hardware, we just need a  Pi0W. It’s important that it’s the Pi Zero W and not the Pi Zero,  because the original Pi Zero doesn’t have Wi-Fi or Bluetooth while W  version does!

![img](https://miro.medium.com/max/30/1*Za7QqTUmhfDWO_CTivlJ6w.jpeg?q=20)

![img](https://miro.medium.com/max/700/1*Za7QqTUmhfDWO_CTivlJ6w.jpeg)

Pi Zero W connected (credits Chang Sau Sheong)

You’ll also need a microSD card and that’s about it. If you’re new to the Pi0W (like I was) you’ll need to remember a few things:

1. You should have a mini HDMI cable to have a proper output to a monitor. Remember it’s *mini* HDMI — the Pi4 has *micro* HDMI and the Pi3 has full-sized HDMI. When you start you won’t have  network access so if you’re looking to SSH or VNC in, you’ll first need  to set up a Wi-Fi connection.
2. On the same point, remember that the Pi0W doesn’t have any full-sized USB  ports! Instead, it has 2 micro-USB ports, one of which you can only use  to supply power to it. So if you need to have an external device, like a mouse or keyboard you’ll need to have a micro-USB to USB adapter.  Remember, if you are thinking you’ll use Bluetooth for keyboard and  mouse, you’ll still need to pair them up first and you can’t do that  without at least a mouse!

![img](https://miro.medium.com/max/30/1*KPy9UbWe6YVSUKtu1SQxUQ.jpeg?q=20)

![img](https://miro.medium.com/max/700/1*KPy9UbWe6YVSUKtu1SQxUQ.jpeg)

micro-USB to USB cable (credits: Chang Sau Sheong)

These points seem trivial, but if you don’t have the necessary pieces of  cables or adapters lying around the house you’ll be literally stuck,  like I was for a few days (until I managed to get something shipped to  me).

## Setting up the OS

Next is setting up the OS, which is quite trivial. I used to use [Balena Etcher](https://www.balena.io/etcher/) to burn the images directly to a micro-SD card, which was super easy. But now it’s even easier, just use the [Raspberry Pi Imager](https://www.raspberrypi.org/blog/raspberry-pi-imager-imaging-utility/)! You don’t even need to download an image, just select the OS you want  to use, select the SD card to burn it to and it will handle everything  else for you.

![img](https://miro.medium.com/max/30/1*RdzEypvS_sV9Icx0lX8ycA.png?q=20)

![img](https://miro.medium.com/max/700/1*RdzEypvS_sV9Icx0lX8ycA.png)

Raspberry Pi Imager

For this little project, I used Raspbian. After you’re done, boot up your  Pi0W and follow the instructions through. You’ll be asked to set up your location, password, Wi-Fi and it’ll do some system updates, after which you’ll be asked to reboot.

After rebooting, you can go to the `Raspberry Pi Configuration` (Applications Menu >Preferences >Raspberry Pi Configuration) to enable SSH through the `Interfaces` tab. With this you can SSH into the Pi0W and don’t need to connect to a monitor any more. You should also set up your Local, Timezone and Wi-Fi Country in `Localisation` tab. When you’re done you’ll be asked to reboot again.

Once it’s all set up, you can SSH into the Pi0W to continue.

## Enabling monitor mode

To do network monitoring, we need to set up our [wireless network interface controller (WNIC)](https://en.wikipedia.org/wiki/Wireless_network_interface_controller) to [*monitor mode*](https://en.wikipedia.org/wiki/Monitor_mode). Monitor mode is one of the few modes that 802.11 WNICs can operate in,  including master (or access point), managed (or station) and ad hoc  (IBSS).

Master mode is for access points while managed mode (or station mode) is what  the WNICs in our devices are normally in). Monitor mode allows the  device to monitor all traffic received without having to be associated  to an access point and this is what we need.

Let’s take a look at the setup we have on the Pi0W now. Run this command:

```
iwconfig
```

![img](https://miro.medium.com/max/30/1*Y2uYDURsz4phYu4HlYfRVQ.png?q=20)

![img](https://miro.medium.com/max/700/1*Y2uYDURsz4phYu4HlYfRVQ.png)

As you can see, the Pi0W is not in the managed mode. Instead it’s  connected to my home Wi-Fi at 2.4 GHz. Monitor mode is not commonly  supported by all WNICs though, this has to be the correct wireless  chipset and drivers. The Pi0W uses the [Broadcom/Cypress BCM43438 chipset](https://www.cypress.com/file/298076/download), which is the same as the one that provides 802.11n (now also known as [WiFi 4](https://www.theverge.com/2018/10/3/17926212/wifi-6-version-numbers-announced)) and Bluetooth 4.0 connectivity to the Pi 3 Model B. Note that in  802.11n, the 5GHz band is optional, and in this case the Pi0W only  supports the 2.4GHz band.

Let’s see if out setup supports monitor mode. Run this command:

```
iw phy phy0 info
```

![img](https://miro.medium.com/max/30/1*uYVgSnb320x5QjmlZLqICg.png?q=20)

![img](https://miro.medium.com/max/700/1*uYVgSnb320x5QjmlZLqICg.png)

From the list of supported modes, you can tell that monitor mode is not  supported out of the box in Raspbian. However, the good news is that the BCM43438 can support monitor mode but we need to first patch it up with [Nexmon](https://github.com/seemoo-lab/nexmon) to enable it. The easier way though, is to simply install The [Re4son-Kernel](https://re4son-kernel.com/re4son-pi-kernel/) for Raspberry Pi, which patches Raspbian with the [Kali Linux](https://www.kali.org) kernel and adds the necessary Nexmon drivers as well.

```
wget -O re4son-kernel_current.tar.xz https://re4son-kernel.com/download/re4son-kernel-current/
```

This downloads the current release, we have to unpack it and install.

```
tar -xJf re4son-kernel_current.tar.xz
cd re4son-kernel_4*
sudo ./install.sh
```

This will take a bit of time and when we’re done, it will reboot. After rebooting, let’s check:

![img](https://miro.medium.com/max/30/1*lxQOj6R7RFYJ63bMM9qtbQ.png?q=20)

![img](https://miro.medium.com/max/700/1*lxQOj6R7RFYJ63bMM9qtbQ.png)

Success! You can see now that monitor mode is supported! But wait, our `wlan0` interface is still in managed mode so we can’t use that. We don’t want  to use that either, because if we do we’ll lose our connectivity to the  outside world, which would be sad.

So we need create another interface in monitor mode. Run this command:

```
sudo iw phy phy0 interface add mon0 type monitor
sudo ifconfig mon0 up
```

![img](https://miro.medium.com/max/30/1*vCZyazYjosm1U6Vwq7SJ7Q.png?q=20)

![img](https://miro.medium.com/max/700/1*vCZyazYjosm1U6Vwq7SJ7Q.png)

When we check with `iwconfig` again we see there’s a new interface named `mon0` that’s in monitor mode. To keep this interface after a reboot, we need to add it to `/etc/rc.local`.

# Using airodump-ng to scan

Now that we prepared out Pi0W, let’s install the tools to do the  monitoring. There are plenty of them out there, but the one I’m using  for this project is called aircrack-ng.

[Aircrack-ng](https://aircrack-ng.org) is a suite of tools for doing lots of stuff with Wi-Fi networks. It’s a fork of the original Aircrack project (hence ‘ng’ or new generation)  and has a ton of tools for “testing Wi-Fi security”, ranging from  cracking WEP and WPA/PSK keys to capturing and injecting packets into  the network.

The tool we’re interested in particular is [airodump-ng](https://aircrack-ng.org/doku.php?id=airodump-ng), which is a tool for monitoring and capturing 802.11 packets. We’ll be  using this tool to create a data file for our processing later.

Let’s start with installing aircrack-ng.

```
sudo apt install aircrack-ng
```

Once you’re done, do a quick test with airodump-ng on the `mon0` interface:

```
sudo airodump-ng mon0
```

You should see a whole bunch of stuff starting to come up on the screen.

![img](https://miro.medium.com/max/30/1*rLU_gW5_fTE9GfhishdPzw.png?q=20)

![img](https://miro.medium.com/max/700/1*rLU_gW5_fTE9GfhishdPzw.png)

That’s a lot of data to take in! Let’s start from the top. The first line  shows the channel that it is currently scanning and the elapsed time.  Next comes 2 different sections. The first contains a list of nearby  access points airodump-ng found while the second are a list of stations  (or clients).

The BSSID is the MAC address of the access point (probably a router). BSSID stands for basic service set identifier where basic service set (BSS)  is the access point. To be clear, ESSID (extended service set  identifier) is the name of the ESS (extended service set, or wireless  LAN) which can have one or more access points. SSID is another term for  ESSID.

Station is the MAC address of the clients. You might notice that some of the  stations are associated with BSSIDs — this means they are connected to  that access point. For those that are not associated, they are either  not connected to any network, or they are connected to a 5GHz network  (remember, the Pi0W can only ‘see’ the 2.4GHz networks).

But wait, what does it mean it’s not connected to any network, then how does airodump-ng find them? Well this is due to how [802.11 authentication and association](https://documentation.meraki.com/MR/WiFi_Basics_and_Best_Practices/802.11_Association_Process_Explained) works. Clients constantly send out probe requests to discover networks, and these probes contain information about itself and its capabilities. When access points receive these probe requests they will in turn send  probe responses with information about itself. Airodump-ng simply  captures these information and processes them accordingly.

![img](https://miro.medium.com/max/30/1*rrw7jk4U0TcCRv3nE83Frw.png?q=20)

![img](https://miro.medium.com/max/591/1*rrw7jk4U0TcCRv3nE83Frw.png)

802.11 authentication and association flow

Airodump-ng can be configured to write its data to file. In fact it can write a LOT of data to file, but to simplify things, we will ask it to write to its own CSV formatted file only. In addition, we will also ask it to scan  both the 2.4GHz (1 to 13) as well as the 5GHz (36 to 165) channels.

```
sudo airodump-ng --channel 1-13,36-165 --write dump --output-format csv mon0
```

This captures the output of scan to a file named `dump-01.csv`.

![img](https://miro.medium.com/max/30/1*zjrnpui6B9-cdPezVs0ZTg.png?q=20)

![img](https://miro.medium.com/max/700/1*zjrnpui6B9-cdPezVs0ZTg.png)

airodump-ng CSV flow

Now that we have the data dump, all is left is to process and show it.

# Processing and displaying the data

Instead of showing it up as a web data, I thought maybe it’s good to make a web service out of it. In that case if we deploy multiple Pi0Ws as  scanners, we can aggregate them through another application to show a  complete picture. Of course, we could also push the data to the cloud to be aggregated too but I thought a pull-based system is less resource  intensive than a push-based one.

For the project I used Go, my trusty Swiss Army knife programming language, to create a simple web service. I’ll start with the structs that are  used to represent access points and clients.

## The structs

the `AccessPoint` struct is rather straightforward, it simply takes the data from the list of BSSIDs found and populate them accordingly. The `Client` is almost the same, except that for the `Organization` field, I used the client’s MAC to figure out which manufacturer produced the device. More on this later.

<iframe src="https://medium.com/media/8f69bc1b988a72c29d3e4977dbbeeff4" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="569" frameborder="0"></iframe>

## Parsing the airodump-ng CSV

The bulk of the work is on parsing the CSV file that we got from  airodump-ng. It’s a rather unusual format because there are two parts to this file and technically it’s not even a proper CSV file. Nonetheless  it’s not that difficult to parse, I simply split the file into two  accordingly to get the two proper CSV content into strings, then parse  them separately.

<iframe src="https://medium.com/media/967da8c7967717bf4e077ba37d18ca88" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="317" frameborder="0"></iframe>

## Parsing the access points CSV

Let’s start with the access points since there are simpler. Go has a [nice CSV package](https://golang.org/pkg/encoding/csv/) in its standard library so parsing it is pretty straightforward. We  just need to set the CSV Reader to expect a variable number of fields  per record by setting the field to `-1`. Also, there’re lots of empty spaces with in every column, so we trim  those spaces out as well. The rest of the code checks if there’s  something wrong with the CSV file and skips the record if so. The  function returns an `AccessPoint` array.

<iframe src="https://medium.com/media/f5092f5eef7828bc7181ea3da047be09" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="1065" frameborder="0"></iframe>

## All about MAC addresses

Before we look at the clients CSV, let’s step back a bit to understand what a [MAC address](https://en.wikipedia.org/wiki/MAC_address) is. A MAC (media access control) address is a unique identifier  assigned to a network interface controller. Addresses can be universally administered (UAA) or locally administered (LAA). UAA is uniquely  assigned to a device by its manufacturer, and its first 3 octets (known  as the [organizationally unique identifier or OUI](https://en.wikipedia.org/wiki/Organizationally_unique_identifier)) identify the organization that issued that address. [OUIs are issued by the IEEE](https://standards.ieee.org/products-services/regauth/oui/index.html) and organizations that wants to issue them must buy them from IEEE. UAAs are usually burnt in physical addresses.

A LAA is assigned to the device by the network administrator or user or  even be randomly generated. When a LAA exists, normally it overrides the UAA. For example,if you have a home Wi-Fi you might find that your home router allow you to use a different MAC address to identify itself.  That’s the LAA, while the true MAC address of your home router is the  UAA.

You can tell if a MAC address is UAA or LAA by its second least significant bit of the first octet of the address. This bit is also known as the [U/L bit](https://packetsdropped.wordpress.com/2011/01/13/mac-address-universally-or-locally-administered-bit-and-individualgroup-bit/). If the bit is 1 the address is LAA, if the bit is 0 is UAA.

```
|            OUI                 |
| Octet 0  | Octet 1  | Octet 2  |
|  A     C |  D     E |  4     8 |
|1010  1100|1101  1110|0100  1000|
         |                 
         |                 
         |                 
     second least significant bit of first octet of OUI =  U/L bit
```

Here’s the code to check if it’s LAA or UAA.

<iframe src="https://medium.com/media/3f378c1f30d1d95f80e0e924416eb532" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="283" frameborder="0"></iframe>

Because the MAC address is a string, we take the first octet parse it into a hex number using `ParseInt`. Then using a combination of bitwise right shift and bitwise AND  operations, we extract the U/L bit to check if it’s LAA or UAA.

If it’s UAA, we will need to go find the organization from the OUI. There  are in fact plenty of APIs that provide the OUI given the MAC address.  However, parsing the OUI from the the database from IEEE is pretty  simple, so I didn’t bother calling any of them. The database itself can  be found at http://standards-oui.ieee.org/oui.txt.

![img](https://miro.medium.com/max/30/1*wR-Cc2siPP-o-swyhsCajA.png?q=20)

![img](https://miro.medium.com/max/700/1*wR-Cc2siPP-o-swyhsCajA.png)

You can see from this file that pretty straightforward to parse. I simply look at lines that has `(hex)` in it, ignoring the rest. Then I split up the line and then use the  first part (which is the OUI, looking something like this — `00-22-72`) as the key to a map, and the organization’s name as the value. This  will be our OUI database, loaded up every time the program starts.

<iframe src="https://medium.com/media/ca0f7af5ea7429dd7afa228d486c91b4" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="437" frameborder="0"></iframe>

In some organisations, when they issue LAA, instead of using a random  identifier, they can choose to use a company ID (CID). A CID, like the  OUI is 3 octet identifier that is registered with IEEE. It’s not meant  to be used to generate UAA (the U/L bit is 0 anyway) but it can be  useful to identify an LAA belonging to a certain company. For example,  Google sometimes use the`DA-A1–19` CID when they create their LAA so if we see this as the first 3 octets in a MAC address, we know the client is a Google device.

Because the CID database file from http://standards-oui.ieee.org/cid/cid.txt has somewhat the same format, we use the same technique to parse and populate our CID database.

<iframe src="https://medium.com/media/929de68507049341afde39fa597801ff" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="449" frameborder="0"></iframe>

## Parsing the clients CSV

With this understanding, let’s take a look at parsing clients CSV now. Most  of the other code is the same as before but we also check the MAC  address to figure out a few things:

1. Is it a universally or locally administered MAC address?
2. If it’s LAA, we check if the first 3 octets are found in the CID database. If it is, we use the company name as the organization. If it’s not then we can’t do anything, we just label is as `LOCAL`
3. Otherwise it’s UAA, we’ll check the first 3 octets against the OUI database to find the organization’s name.

<iframe src="https://medium.com/media/92d0db41dec4b40e61e5ff37ba5b4e72" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="1241" frameborder="0"></iframe>

## Building the web service

The rest of the code is quite straightfoward. I created a `serve` function to register the handlers, run a goroutine to periodically get data and then start the server.

<iframe src="https://medium.com/media/e49b254215a4f9bc77e1a1888452a43c" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="361" frameborder="0"></iframe>

The goroutine runs an infinite loop, sleeping for 10 seconds, then wakes up to get the data.

<iframe src="https://medium.com/media/935433295b0358971bf1296f16906bed" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="173" frameborder="0"></iframe>

The two main handlers serve to take the current list of clients and access  points and serve them out as JSON. For the clients, because we want to  hide clients that have not been seen for a while, I have a filter that  only shows clients for the past number of minutes only.

<iframe src="https://medium.com/media/a29f16d51f23bec2fcd954dfb1f122bd" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="657" frameborder="0"></iframe>

I only showed snippets of the code above, if you want to see the all the code (it’s all in one file), you can find it at https://github.com/sausheong/netnet.

# Running the web service

To run the web service, we have to install Go in the Pi0W. Run this command:

```
wget https://dl.google.com/go/go1.14.2.linux-armv6l.tar.gz
sudo tar -C /usr/local -xzf go1.14.2.linux-armv6l.tar.gz
```

This will download and unpack the Go binaries into `/usr/local`. Then add `/usr/local/go/bin` to the `PATH` environment variable. You should also create a workspace (usually in the `go` directory of your home directory) and then set the `GOPATH` variable to point to it. You can do this by adding these lines to `$HOME/.profile`:

```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
```

Once Go is set up, you can clone the code directory.

```
git clone https://github.com/sausheong/netnet.git
```

Then go to that directory and build it.

```
go build
```

And you’re done! All that’s left for you to do is to run the web service.

```
./netnet -f=/home/pi/dump-01.csv
```

The web services starts at port 12121, but you can change it to something  else. I’ve also included a simple home page to list the services.

![img](https://miro.medium.com/max/30/1*02YxVsgvPrPpapk-aj4G9A.png?q=20)

![img](https://miro.medium.com/max/700/1*02YxVsgvPrPpapk-aj4G9A.png)

This is the result from trying to get the data from web service.

![img](https://miro.medium.com/max/30/1*l5v5jCQ5FQAkbpYysS4yXA.png?q=20)

![img](https://miro.medium.com/max/700/1*l5v5jCQ5FQAkbpYysS4yXA.png)

# Setting up services

But wait. So far we’ve only setup and created console-based applications to run. Once we log out, or if our SSH session terminates, the systems are shut down and they’re gone. We can’t run scanners this way, we need  them to start up whenever we power on the Pi0W. After all, this is an  IOT device.

To do this we have to do a few things and make some tweaks to what we  already have. First, we need to create a service to start up airodump.  We’ll be using [systemd](https://www.freedesktop.org/wiki/Software/systemd/) to do this. Create a file in `/lib/systemd/system` named `airodump.service`.

```
[Unit]
Description=airodump[Service]
Type=idle
ExecStart=/home/pi/airodump.sh
```

If you’re used to creating systemd services, you might notice that this is an ‘idle’ service. When set to this type, the systemd service will be  delayed until all active jobs are dispatched. We do this because we want to start airodump only after the network is set up, because creating a  monitor mode interface causes some conflict and can disable your normal  network access.

This is the `airodump.sh` file.

<iframe src="https://medium.com/media/4328ea2dd30786d240f094f389ecd978" allowfullscreen="" title="scanner" class="t u v ii aj" scrolling="auto" width="680" height="273" frameborder="0"></iframe>

Notice that we are starting creating the monitor mode interface in this script and not in `/etc/rc.local` any more. Instead in the `rc.local` file, we’ll start up the airodump systemd service.

```
sudo systemctl start airodump
sleep 5s
sudo systemctl start netnet
```

We also wait for 5 seconds, and then start another systemd service that will execute our `netnet` webservice. Create a file in `/lib/systemd/system` named `netnet.service` that does this.

```
[Unit]
Description=netnet[Service]
Type=idle
ExecStart=/home/pi/go/src/github.com/sausheong/netnet/netnet -f=/home/pi/dump-01.csv
```

You might notice that I didn’t install the services with the `enable` command in `systemctl`. This should be the normal way of using systemd but I can’t seem to get  it running. If you can do it, leave me a note in the comments!

After this you can reboot the Pi0W and you should be able to access the web service without starting them manually.

# What’s next?

After all this, we still don’t have any proper application!

Well, this is just the start. If you have a few of these scanners in an open  area you can start detecting the the access points and clients. If you  detect Wi-Fi clients starting to pop up in one scanner and then go away  after a while, then reappear in another scanner you can deduce that it’s probably a mobile device with a human attached to it. And with the  strength of the signals you can probably deduce how far or near they are accordingly.

![img](https://miro.medium.com/max/30/1*3cyZctfFVuP53_f5D1NPjA.jpeg?q=20)

![img](https://miro.medium.com/max/700/1*3cyZctfFVuP53_f5D1NPjA.jpeg)

My Pi Zero W and standard case (credits: Chang Sau Sheong)

With this I can monitor human traffic flow and do some geo-fencing! But um maybe I need to be able to leave the house first.

Oh well.

[The Startup](https://medium.com/swlh?source=post_sidebar--------------------------post_sidebar-----------)

Get smarter at building your thing. Join The Startup’s +735K followers.
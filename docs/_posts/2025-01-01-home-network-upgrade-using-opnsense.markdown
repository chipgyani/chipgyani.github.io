---
layout: post
title:  "Home Network upgrade: OPNsense router/firewall"
date:   2025-01-01 19:45:52 -0500
category: blog
tags: router firewall OPNsense
---

### Introduction ###

Like many others, we had the whole family working and schooling from home during the peak of the COVID-19 pandemic in 2020/2021. The performance of my home network was somewhat inconsistent during multiple video calls happening concurrently. This prompted a complete rethink of the network, helped along by the fact that we were also planning some home remodeling at the time. I am not a network engineer, but have played around enough over the years to have a good grasp of basic concepts. I did a bit of research to understand different options and implemented what I thought would improve our network and would be maintainable over time.

We live in a 100-year old townhouse unit spread over 3 levels: a basement, the main floor (has living+dining+kitchen and 2 bedrooms), and a finished attic level (one bedroom and a spare multi-functional room). The basement is mostly unfinished and it's where we have the utilities and where the main cable and fiber connections come in from the outside. The walls are made of plaster and, as we found out later, the plaster is held together with a wire-mesh. 

#### Previous Network  ####

In the before times, I had my office area setup in one portion of the basement that was finished. This basement room has both coax as well as Cat5e ethernet wired into one wall, and runs to the point where the ONT and coax box is installed in the unfinished side (this existed when we moved in). I have switched between a cable internet provider and a fiber provider, and have used both those connections in the past.

I had a simple consumer router flashed with [OpenWRT](https://openwrt.org/) and had it setup in the basement office. I used wired Ethernet for my computer in the basement office, and the WiFi was okay in the main floor to use from our devices. We didn't spend a whole lot of time in the attic level, so the poor coverage there didn't bother us too much. Later, as kids got a bit older, I added a [MoCA2 adapter](https://mocalliance.org/MoCA2/index.php) in the living room on the main floor using the coax drop for the TV. This adapter had built-in WiFi access point (AP), so this provided better coverage in the main floor as well as the attic level. I measured the wired bandwidth between the living room extender and my router to be about 300Mbps and an average latency just under 1ms -- this was good enough for what we needed. Till the pandemic changed everything ...


#### Network Issues ####

Kids were in elementary school and classes moved to Zoom once it became clear that this COVID pandemic wasn't a short term affair. At work, I was involved in the bring-up of a brand new chip and we were forced to do this remotely instead of being co-located in a lab. This meant being in Teams calls most of the day. Working from the basement was ok when I was doing it infrequently, but I didn't like being there every day (especially in the winter months). So I setup a desk in the multi-functional room in the attic. The kids' bedrooms on the main floor were farthest from the WiFi AP, and the signal was very poor. The signal in the attic also wasn't great: I would go down to the basement office and use the wired connection for certain critical meetings. My kids would experience frequent lags and some dropouts, and they would promptly come to me to complain.

The problem was the wire-mesh in the walls which was significantly attenuating the WiFi frequencies -- essentially becoming a [Faraday cage](https://en.wikipedia.org/wiki/Faraday_cage). The attenuation was worse in the 5GHz band than in the 2.4GHz range. I forced my kids' devices to only connect using 2.4GHz which improved the situation somewhat - at least they didn't experience major dropouts. I thought about getting a mesh router to deal with this, but I didn't because of two reasons:
1. The devices with good reviews were sold out at most reputable places or had a long lead time for delivery
2. Mesh routers use 5GHz band for the back-haul between nodes, and so it may not improve the situation at all

So we muddled on for a while.


#### Wired ethernet ####

We were supposed to have contractors come in to some home remodeling (in March 2020), but that obviously got delayed. The contractor and his crew got vaccinated by mid-2021 and they started work. With the bathroom walls down, I got the idea of running ethernet cables to each floor from the basement. I talked to the contractor and he agreed to run the cables to the kids bedrooms and the multi-function room for a small extra charge: I ran two drops to each room for redundancy. We also added extra insulation and baseboard heating to a three-season enclosed porch and converted one portion of it to be my new office space. I asked cables to be run to this space as well. I chose to run [Cat6A cables](https://en.wikipedia.org/wiki/Category_6_cable), allowing me to go up to 10Gbps, i.e. [10GBASE-T](https://en.wikipedia.org/wiki/10_Gigabit_Ethernet#10GBASE-T) -- 1Gbps is enough for my needs today, but I wanted to future proof it for some time.

I also thought it would be good to ceiling mount the WiFi access points like they do in my office building. So I asked them to run two additional drops: one towards the back of the house on the main floor (central to kids' rooms and the kitchen) and another towards the front in the attic (multi-function room which is right above the main floor living room). I knew I could find APs that are powered via [PoE](https://en.wikipedia.org/wiki/Power_over_Ethernet "Power over Ethernet"), so I didn't need to have a power outlet near the AP mount points.

All the cables came down to one area in the unfinished part of the basement: I would have prefered to get it to the finished part, but that would have required additional work and we were already above budget in our renovation. I would have to setup my "network operations" in this spot.


### Router ###

With the physical layer sorted out, the next item was router selection. Our Internet plan is 500Mbps down, and our existing router didn't really go much past 400Mbps. So, it was time to upgrade the router. The choice was simple:
1. Get another consumer/prosumer router that is capable of 1Gbps WAN and run OpenWRT on it
2. Use a custom machine with a router-focused OS

After some consideration, I decided to go with option #2. These were my reasons behind the choice:
1. There were various news articles talking about router vendors (Linksys, TP-Link, et al) locking down the systems such that 3rd party firmware could not be installed. It would have made it harder to find an OpenWRT capable router that fit my needs. Articles: [Wired](https://www.wired.com/2016/03/way-go-fcc-now-manufacturers-locking-routers/), [NetworkWorld](https://www.networkworld.com/article/948708/manufacturers-start-to-lock-down-wi-fi-router-firmware-thanks-fcc.html)
2. Consumer routers had limited built-in flash and RAM, and this limited what additional services I could run on the router. Also it would be easy to "brick" the router if you were not careful with with which OpenWRT image you tried to flash.
3. There is additional flexibility in choice of router OS if we use a more DIY approach
4. I liked the idea of learning something new in this process


#### Router OS ####

The next choice was deciding the router OS to use. 

The OS had to meet some criteria:
1. Stable
2. Secure, with regular updates
3. Flexible and ease of use (semi-technical spouse should be able to navigate it if required)
4. [FLOSS](https://en.wikipedia.org/wiki/Free_and_open-source_software "Free/Libre and Open Source Software") with a reasonably active developer and user community: I am a strong proponent of open source software and for a security focused device, I think it is important that the code is open for "many eyes".


I also had a few feature requirements for router OS:
* Full featured stateful firewall: this is pretty much a given for any device exposed to the Internet
* DHCP server: again table stakes for a home router
* VLAN support: I wanted to segment my network between our family's own devices, vs guests and other devices (IoT stuff, "smart" TVs, etc.)
* [Wireguard](https://www.wireguard.com/) VPN support: I use Wireguard when I am on an untrusted public WiFi or if I need to connect to the NAS at home. I had Wireguard running on a Raspberry Pi as my router itself wasn't able to run it. I wanted Wireguard to be running on the router itself to avoid running it on a separate device
* (Optional) DNS filtering/blacklisting: I ran PiHole with some basic DNS filtering -- mostly to block known spam and scam sites. It ran on the same RPi as Wireguard, and again it would be good to consolidate it within the router device itself
* (Optional) Traffic shaping / QoS: This would be for giving lower latency access to game consoles and gaming PCs, but this shouldn't be an issue with a fast-enough Internet connection

The OSes that fit most of the above criteria were:
* [pfSense](https://www.pfsense.org/)
* [OPNsense](https://opnsense.org/)
* [OpenWRT](https://openwrt.org/)
* [IPFire](https://www.ipfire.org/)

The first two are derived from FreeBSD while the remaining are built on top of Linux.

The title of this post gives away my final choice: OPNsense. I almost exclusively run Linux on all my machines at home and I thought it would be good to have a FreeBSD based firewall to avoid a complete monoculture. Between PFsense and OPNsense, the latter seemed have a more open community and, based on what I read, OPNsense had a faster pace of development. OPNsense is backed by a company called [Deciso A.B.](https://www.deciso.com/#company) that is based in the Netherlands. Deciso offers business versions of OPNsense for commerical support as well as various appliances that run OPNsense.

OPNsense is actually a fork of pfSense, wich itself was a fork of [m0n0wall](https://en.wikipedia.org/wiki/M0n0wal), which was one of the early firewall oriented distributions. The creators of m0n0wall themselves asked users to check out OPNsense when they ended development of m0n0wall.


#### OPNsense ####

OPNsense is officially supported on x86/AMD64 based machines, but there are some unofficial builds for Aarch64 (64-bit ARM) available from 3rd party repos. It can be installed via USB or othe flash storage using console display or entirely headless via a serial port. Hardware support follows the FreeBSD version that it is derived from, and finding drivers for certain devices can be tricky. The recommendation is to stuck with Intel NICs as those are the best supported: support for Realtek NICs is getting better, but it all depends on the specific version of the controller. OPNsense can be installed on baremetal hardware or as a virtual machine under a hypervisor such as KVM or VMWare. After installation, the configuration can be done via console, a web-based interface or via SSH (Note: SSH is disabled by default even on the LAN side).

OPNsense includes the following features:
* Stateful Firewall
* DNS and DHCP servers, dynamic DNS
* DNS filtering
* Two-Factor Authentication
* 802.1Q VLAN support
* Link Aggregation & Failover
* Traffic Shaping
* Built-in reporting and monitoring tools
* Intrusion Detection & Prevention
* Virus scanner
* VPN Services (IPsec, OpenVPN, WireGuard)
* Support for plugins

The recommended HW for OPNsense (as of version 24.7):
* \> 1.5GHz multi-core CPU
* 4GB RAM
* Serial console or video (HDMI/VGA) for installation
* \> 120GB storage for OS & logs
* \>= 2 NIC ports
   * Single NIC workable with a VLAN capable managed switch, so called “[Router on a stick](https://en.wikipedia.org/wiki/Router_on_a_stick)”
   

### Router Hardware ###

Given these requirements, I next set about finding a machine to run OPNsense. I had heard good things about the [APU2 board](https://www.pcengines.ch/apu2.htm) by PCEngines. However, this board was unavailable at the time. I looked on Reddit's [r/OPNsense](https://www.reddit.com/r/opnsense/) and a lot of the folks there were recommending various router-focused machines on Amazon or AliExpress:
* "mini-PC" machines from [Protectli](https://protectli.com/), [Qotom](https://www.qotom.net/), etc.
* There are also machines you could buy from [Deciso](https://www.deciso.com/hardware/)
* Typically have 4 GbE ports
* Typical power draw of 15 to 35W
* Price ranges from $200 to $400

Several users also reported using a slim PC or thin client and then adding a 4-port GbE NIC to an available PCIe slot. I looked into this and prices of these used machines + NIC would be in the $200 range as well with a similar ~30W power draw. Others had also looked at used rackmount appliances and this also seemed like a good choice. Since the machine was going to be in the basement, fan noise wouldn't be an issue.

I kept looking online for a while, and eventually stumbled on this machine on eBay:

![Kemp LM3400](https://i.imgur.com/mZfU2Hp.jpg)

It is a [Kemp LM-3400](https://kemptechnologies.com/server-load-balancing/loadmaster-3400) machine: sold as a server load balancing appliance. The hardware features of the machine are:
* 4-core Intel Xeon E3-1225 (Sandybridge generation)
* 8GB of DDR3 RAM
* The front has 8 GbE ports (Intel 82583V NICs), 2 USB2, RJ-45 based serial port
* The rear has a VGA port, 2 more USB2 ports, and a power switch
* 200W ATX power supply
* Several SATA ports, a CompactFlash card slot, and one PCIe slot

There was no disk and therefore did not come with the original Kemp OS or applications installed.

The inside looks like this with a 2.5" disk added:

![Kemp internals](https://i.imgur.com/L8F5bSt.jpeg)

I got this machine for a price of $53.11 including tax and shipping, which I thought was a pretty good deal.

I connected the machine to a monitor, plugged in a keyboard and booted OPNsense from a bootable thumb drive that I had created. The installation proceeded very smoothly and I was presented with a console menu to setup everything. All 8 GbE ports got recognized and were numbered em0 through em7. I setup the basic configuration for WAN and LAN and hen proceeded to setup the rest via the web GUI.


### Network setup ###

In addition to the router I also purchased a 16-port, PoE+ capable managed switch ([Netgear GS316EP](https://www.netgear.com/business/wired/switches/plus/gs316ep/)) that would form my wired LAN. The switch is capable of delivering PoE from any of its 15 GbE ports. For WiFi, I went with Ubiquiti's [UniFi U6-Lite](https://store.ui.com/us/en/products/u6-lite) access points that I mounted to the ceiling and connected them via the ethernet cables that were already dropped where I wanted them.

I mounted the switch to the wall in the basement, right where all the ethernet cabling landed. I had carefully labeled each cable, so I knew which room each cable came from. I connected all the ethernet drops to the switch and configured the ports connected to the APs to enable PoE power delivery. I connected one port from the switch to the port on the router that I had designated as the LAN port.

The router has 8 GbE ports and the switch has 15. I was using 11 ports of the switch and just two 2 of the router (one for LAN and the other for WAN). So I decided to connect another ethernet cable between the switch and the router and "bond" the two ports together using link aggregation feature in both the switch and OPNsense. So em0 and em1 ports on the router both get designated as LAN, and I set em7 as the WAN. The remaining ports were unused. 

With link aggregation, I get up to 2 Gbps of bandwidth between the switch and the router. No single device can use all that bandwidth, but it allows multiple devices to communicate without hitting the 1 Gbps ceiling of a single port. This might be overkill for my modest 500Mbps Internet connection, but the bandwidth between the router and switch wasn't only for WAN bound traffic ... more on that later.

I also enabled Wireguard in OPNsense and configured the "wg0" interface to accept connections. I setup clients on our phones as well as the personal laptops that we use. 


#### VLANs ####

Another of my goals was to improve the security of my home network. Segmenting the network using [Virtual LAN](https://en.wikipedia.org/wiki/VLAN) (or VLAN) allows the logical separation of a certain set of computers and other devices into a separate network, while sharing the same physical network layer as all other machines. Previously, I had a "guest" WiFi SSID -- OpenWRT allowed this to be isolated from the rest of the network. We would hand out the guest WiFi credentials to friends and family visitors. But, over time, we had more "IOT" like devices and I wanted to treat them differently. 

VLANs allows more granular separation of the network. I setup a few different VLANs:
1. Management: The network devices themselves are on this VLAN, including the switch and the APs
2. Internal: The home server (NAS + misc VMs), my desktop, and various laptops
3. Devices: Our phones, smart TVs, streaming devices, school issued Chromebooks
4. IoT: Smart plugs, home automation hubs, and other such devices. A VM in the home server runs [HomeAssistant](https://www.home-assistant.io/) and that is also assigned to this VLAN

In OPNsense, I mapped all these VLANs to the aggregated "LAN" interface. Then I had to repeat the same VLAN setup in the switch and assign specific ports to be tagged with specific VLAN IDs or have them untagged for ports with multiple VLANs (like the WiFi APs). I had to then go and repeat this in the Unifi controller and map specific SSIDs to VLANs. Managing all of this is a pain, and I appreciate why several people go "all in" on one vendor to ease setup and management (e.g. using Unifi APs with Unifi switches and Unifi routers). There is definitely some tradeoff of simplicity vs. flexibility. But, if you have everything planned out like I did, it is a question of making sure the settings are done correctly and tested. After that, there is really no need to fiddle with VLAN settings -- I've been using this for a couple of years and I haven't yet had a reason to change it.


#### Firewall Rules ####

I setup the firewall rules such that devices on a lower numbered VLAN can access a higher numbered VLAN but not vice versa. The only exception is that our phones can backup data to the [Nextcloud](https://nextcloud.com/) instance that is running on our home server. I configured the DHCP server in OPNsense to hand out "static" IP addresses to our phones, and only that address range is allowed to talk to Nextcloud's IP address. 

There are several devices on the IoT VLAN that are barred from talking to the WAN -- this prevents any "phoning home". I mostly only buy devices that can be flashed with open firmware -- specifically, [ESPHome](https://esphome.io/index.html). But I am still paranoid about data leaking to external servers, so these devices are limited to talking to HomeAssistant. The only problem I faced with this rule is that over-the-air firmware updates don't always work, but I only do that if I encounter a bug (which is rare). 

I contemplated putting the smart TVs and streaming devices in the IoT VLAN, but this would cause headaches when trying to cast from phones to the TV -- I would have to fiddle with mDNS and I didn't want to venture down that rabbit hole. I have to let these devices to get to the Internet, and there is really no easy way to stop them from phoning home. I have accepted this loss of privacy in exchange for convenient access to all the content.

We give most friends the IoT SSID and let them get to the Internet, but they can't do much else. We did give some close friends and family access to the "Devices" SSID so that they could cast stuff to our TV. Other than this, the firewall rules are fairly straightforward. OPNsense is very good about being secure by default and so nothing is exposed unless I specifically enable it. 

Additionally, I configured some firewall rules for the Wireguard wg0 interface. I enabled traffic coming in via Wireguard to go out via the WAN -- this allows us to connect via our home connection while we are away (untrusted networks or if we need to get around geo-blocking while outside the country). I also allowed access to the NAS and Nextcloud, so that we could sync pictures from our phones while away and also access our files. This requires us to leave everything running even when we go away on vacation.


#### Inter-VLAN routing ####

I had mentioned earlier that the link between the switch and the OPNsense router isn't just for WAN bound traffic. My firewall rules above allow some traffic to go between the different VLANs. Routing between VLANs is by IP address, so I need a Layer 3 device for inter-VLAN routing. In my case, my switch is a Layer 2 device and so cannot route traffic between VLANs. Therefore, this falls to the OPNsense router itself -- all traffic between VLANs have to go to the router and then back in to the switch. Hence, having extra bandwidth between the switch and router is useful and the link aggregated 2Gbps capacity ensures that WAN traffic and inter-VLAN traffic can both get good performance. It is not to say that I would have noticed any significant issues if I only had a 1 Gbps link -- but I had the extra port and so I put it to use.


### Performance and Power ###

I am a performance architect in my day job, so this post won't be complete without looking at the performance of my network. I ran a few different tests: testing Internet bandwidth using [Ookla Speedtest](https://www.speedtest.net/), and testing LAN bandwidth using [iperf3](https://iperf.fr/).

#### Speedtest #### 

On Speedtest, I was able to get full upload and download bandwidth advertised by my cable ISP (500 Mbps download, 35 Mbps upload):

From a wired connection to the switch:
```
Download: 523 Mbps
Upload:    33 Mbps
```

From WiFi, with the laptop in the same area as the AP:
```
Download: 497 Mbps
Upload:    33 Mbps
```

I ran the test a few different times, and the variance wasn't all that much.


#### LAN iperf3 ####

I setup iperf3 to run simultaneously from two different machines on different VLANs to test bandwidth in both directions, both through the switch and through the router. 

Machine A to Machine B:
```
Connecting to host 172.16.2.25, port 52201
[ 5] local 172.16.3.18 port 47572 connected to 172.16.2.25 port 52201
[ ID] Interval Transfer Bitrate Retr Cwnd
[ 5] 0.00-1.00 sec 112 MBytes 940 Mbits/sec 0 744 KBytes
[ 5] 1.00-2.00 sec 110 MBytes 923 Mbits/sec 0 782 KBytes
[ 5] 2.00-3.00 sec 111 MBytes 933 Mbits/sec 0 822 KBytes
[ 5] 3.00-4.00 sec 111 MBytes 933 Mbits/sec 0 822 KBytes
[ 5] 4.00-5.00 sec 110 MBytes 923 Mbits/sec 2 605 KBytes
[ 5] 5.00-6.00 sec 111 MBytes 933 Mbits/sec 0 723 KBytes
[ 5] 6.00-7.00 sec 111 MBytes 933 Mbits/sec 0 758 KBytes
[ 5] 7.00-8.00 sec 111 MBytes 933 Mbits/sec 0 758 KBytes
[ 5] 8.00-9.00 sec 110 MBytes 923 Mbits/sec 0 819 KBytes
[ 5] 9.00-10.00 sec 112 MBytes 944 Mbits/sec 0 834 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval Transfer Bitrate Retr
[ 5] 0.00-10.00 sec 1.08 GBytes 932 Mbits/sec 2 sender
[ 5] 0.00-10.01 sec 1.08 GBytes 929 Mbits/sec receiver
```

Concurrently running Machine B to Machine A:
```
Connecting to host 172.16.3.18, port 5201
[ 5] local 172.16.2.25 port 54792 connected to 172.16.3.18 port 5201
[ ID] Interval Transfer Bitrate Retr Cwnd
[ 5] 0.00-1.00 sec 113 MBytes 949 Mbits/sec 0 430 KBytes
[ 5] 1.00-2.00 sec 112 MBytes 940 Mbits/sec 0 648 KBytes
[ 5] 2.00-3.00 sec 111 MBytes 933 Mbits/sec 0 717 KBytes
[ 5] 3.00-4.00 sec 111 MBytes 933 Mbits/sec 0 717 KBytes
[ 5] 4.00-5.00 sec 111 MBytes 932 Mbits/sec 0 749 KBytes
[ 5] 5.00-6.00 sec 110 MBytes 924 Mbits/sec 0 785 KBytes
[ 5] 6.00-7.00 sec 111 MBytes 933 Mbits/sec 0 830 KBytes
[ 5] 7.00-8.00 sec 111 MBytes 933 Mbits/sec 0 830 KBytes
[ 5] 8.00-9.00 sec 111 MBytes 933 Mbits/sec 0 830 KBytes
[ 5] 9.00-10.00 sec 110 MBytes 923 Mbits/sec 0 830 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval Transfer Bitrate Retr
[ 5] 0.00-10.00 sec 1.09 GBytes 933 Mbits/sec 0 sender
[ 5] 0.00-10.00 sec 1.08 GBytes 931 Mbits/sec receiver
```

As you can see, we are close to maxing out the line rate in both directions simultaneously. I didn't setup the test to fully saturate the aggregated link between the switch and the router.


#### Power ####

I made some power measurements using a Kill-a-Watt meter.

I plugged in the router, switch, and my cable modem into a power strip and connected the power strip via the Kill-a-Watt. As the APs are powered by the switch itself, so the power measured will include the WiFi AP PoE power as well. I let this run for a 48 hour period and we used the network as normal. Over this period, the setup drew an average power of 89W. I repeated this for another 48 hours, but this time only the OPNsense router was plugged into the Kill-a-Watt. The router itself averaged about 65W -- this was a bit higher than I had expected, but not surprising it is an older machine meant for datacenters.

I went about seeing if I could reduce the power as much as possible. I looked at the FreeBSD documentation for [CPU power management](https://docs.freebsd.org/en/books/handbook/config/#cpu-power-management) and found a few tunables using `sysctl dev.cpu.0`.

```
dev.cpu.0.cx_lowest: C1
dev.cpu.0.cx_supported: C1/1/1 C2/2/80 C3/3/104
dev.cpu.0.freq_levels: 3101/95000 3100/95000 3000/90163 2900/86347 2800/82600 2700/78924 2600/74419 2500/70905 2300/64048 2200/59864 2100/56612 2000/53437 1900/50315 1800/47257 1700/43458 1600/40536
```

The CPUs weren't able to go down to C3 state to save power. CPUs are essentially shut off in C3 and the latency to come out of C3 can be quite high. So, C3 shouldn't be enabled if you want a deterministic performance under varying loads. Specifically, in networking it could be bad if you have a surge of traffic coming in when the CPUs are off. I changed `cx_lowest` to `C3` to see if there would be timeouts etc. because of poor CPU responsiveness. However, I didn't notice anything and left it at that. I ran power measurements with everything plugged in, and now the power draw of the whole setup came down to 77W from the 89W before. I didn't change anything else, so I estimate the router power is now 53W.

I also wanted to fix the CPU frequency to the lowest supported 1600MHz speed. In practice, I think the default CPU governor does a pretty good job and I didn't see any real power reduction with this change. I also tried some of the other settings, but those didn't improve.

53W is still quite high for router running 24/7, but given I only paid $53 for it, the [TCO](https://en.wikipedia.org/wiki/Total_cost_of_ownership) calculation should put me ahead for some time compared to a lower-power but more expensive machine. Also, it makes me feel a bit better extending the life of older machines before they become e-waste.


### Conclusion ###

It's been about 3 years since we've had this "new" network at home. It was a bit expensive to install, given the labor costs of ethernet wiring, cost of the managed switch and relatively pricey WiFi access points (the Kemp machine off eBay was the cheapest of all of them). But we've really had ZERO issues since then. I have updated the OPNsense version a few times and they have all gone without a hitch. 

Unlike consumer routers, OPNsense doesn't seem to like random power glitches and takes a long time to reboot. We tend to have sporadic events where we lose power for just about a second, which is just enough to cause the Kemp to throw a fit. I solved this with an inexpensive [UPS](https://www.apc.com/us/en/product/BE600M1/apc-backups-600va-120v-1-usb-charging-port-7-nema-outlets-2-surge/?range=61883-backups&parent-subcategory-id=88975&selectedNodeId=27590290410) and I can now continue meetings from my laptop even if we experience one of these glitches. OPNsense has a daemon called `apcupsd` that talks to the UPS via USB and can gracefully shut down if the power is out for a long time and the battery charge falls below a certain threshold. I can also check apcupsd status page to find out exactly when the power went off and on if I come home to my oven blinking 12:00.

I am not endorsing any of the products mentioned in this page, but this combination has worked extremely well for me. The performance has been solid and I haven't found any reason to upgrade anything. At some point, when 10G gear gets more affordable, I might swap out the switch for a 10G capable one and add 10G NICs to my NAS and desktop. Till then I expect my current setup to keep chugging happily. 







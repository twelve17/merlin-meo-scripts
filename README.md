merlin-meo-scripts
==================

An adaptation of zipleen's [tomato-ddwrt-meo-iptv-scripts](https://github.com/zipleen/tomato-ddwrt-meo-iptv-scripts) to work with routers running [Merlin firmware](http://www.lostrealm.ca/tower/node/79).

These scripts were tested with Merlin firmware version `374.41` running on a RT-AC66U.

## Installation

To sum up the process outlined in [this very informative blog post](http://blog.zipleen.com/2010/10/how-to-make-meo-fiber-iptv-service-work.html), we need to do the following to get the various [MEO](http://meo.pt) services working: 

* Make the router aware of MEO's VLANs
* Configure Internet access using PPPoE via VLAN10
* Run a custom DHCP client to get the network configuration via VLAN12 (IPTV + VoIP)
* Configure static routes, firewall/NAT rules, and DNS overrides for specific MEO-related IP ranges

Most of the work is going to be done on the command line, *not* on the web admin UI, so you should feel comfortable SSH'ing into your router.  The scripts leverage the Merlin [user scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) as well as [custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) mechanisms to accomplish the same goals as the zipleen versions.  Either way, use at your own risk, don't blame us if you brick your router, etc.  :)

Also, you may want to back up your /jffs directory before upgrading firmware, as its contents are not guaranteed to survive firmware upgrades!

Configure VLANs
---------------

These steps only need to be done once.  Their configuration will be saved to the router and survive reboots, etc.

### Internet VLAN

The router needs to be configured to get its Internet access via VLAN10, rather than the default (untagged) network.  As of this writing, this can be done in the web admin UI: 

* Advanced Settings &#8594; LAN &#8594; IPTV 
  * Port:
    * Select ISP Profile: Manual
      * Internet: 10

I'm not sure if this part matters yet, but under "Special Applications", disable "Use DHCP Routes".

### IPTV/VoIP VLAN

The web UI is somewhat limited with VLAN configuration, so IPTV/VoIP configuration needs to be done on the command line.

**Note**: this part may be specific to the ASUS RT-AC66U/RT-N66U.  Refer to the [Switched Ports](http://www.dd-wrt.com/wiki/index.php/Switched_Ports) article for the values that may apply to your router.

On the CLI, run the following:

```
nvram set vlan12ports="0t 8"
nvram set vlan12hwname=et0
nvram commit
# reboot
```
*Be sure to use quotes on the first one since there is a space in the value.*
**You will need to reboot the router for these settings to take effect.** Type `reboot` on the command line to do so.

A little explanation:

Some routers allow you to replicate the network traffic from one port to another.  The RT-AC66U, for example, has five network ports: 1 WAN and 4 LAN ports.  The nvram values for these ports are 0 for the WAN port, and 1-4 for the four LAN ports.

Port #8 is a special port (called the CPU internal port).  If you want the router to interact with any of the network traffic on the ports, it needs to be "plugged into" the CPU internal port.  If omitted, the router will pass along the traffic from one external port to another and otherwise not pay attention to it.

So, `vlan12ports="0t 8"` means "Take the VLAN12 network traffic and make it available to the router itself.", which then allows us to forward the signal to the LAN later.   The "t" in the "0t" means that the incoming signal to the WAN port is "trunked", which just means that the incoming signal will have tagged VLAN traffic (e.g. multiple networks) in it.  

#### Alternative for VoIP

Let's say you wish to keep using your Thomson router as the VoIP client.  You could forward *just* VLAN12 to to one of the router's LAN ports:

```
nvram set vlan12ports="0t 4t 8"
nvram set port4vlans=12
nvram set vlan12hwname=et0
nvram commit
# reboot
```

Now, the `vlan12ports` command is essentially telling the router to duplicate the trunked network on the WAN port 0 to the LAN port 4 (in addition to itself via port 8).  You can then plug in the Thomson router to LAN port 4, and it should be able to acces the VoIP service.  Note that since only VLAN12 is being forwarded, the router will not be able to connect to the Internet, but it does not need it to use the VoIP service (and perhaps IPTV, but that has not been tested here!).

Internet
--------

On the web UI:

* Advanced Settings &#8594; Internet Connection:
  * Basic Config
    * WAN Connection Type: "PPPoE"
    * Enable WAN, Enable NAT: "yes"
  * WAN IP Setting
    * Get the WAN IP automatically, Connect to DNS Server automatically: "yes"
  * Account Setting
    * User Name, Password: your Sapo/MEO login and password.
  * Special Requirement from ISP
    * Enable VPN + DHCP Connection: "no" (This would otherwise run the `udhcpc` DHCP client using incorrect settings.  We are going to do this on our own later.)

At this point, if you have the VLANs configured per the previous section, you should have Internet access.  **There's a chance you may have to reboot the MEO Fiber gateway** if you are switching from the Thomson router to your own.


Everything Else: Custom Scripts + Config
-----------------------------------------

The remainder of the configuration (custom DHCP client, routes, firewall config, IGMP proxy, DNS, etc.) is handled by the custom scripts in this project.  In order to use them, you should [enable JFFS support](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) on the router.  Once this is set up, you can simply clone this project and copy the "scripts" and "configs" folders to the `/jffs` folder on the router.

This assumes you don't have anything in your /jffs directory!  

```
$ git clone git@github.com:twelve17/merlin-meo-scripts.git
$ scp -r merlin-meo-scripts user@my_router:/tmp/
$ ssh user@my_router
...
$ cd /tmp/merlin-meo-scripts
$ mv configs scripts /jffs/
```

The layout of the scripts is as follows:

```
- /jffs
  - scripts
    # triggered by Merlin
    wan-start
    ...
    - custom
      # this project's own scripts
      meo-post-dhcp-vlan-config
      ...
  - configs
    # used by Merlin
    ...
    - custom
      # this project's own configs
      meo-igmp-config
      ...
```

The scripts are written in such a way that they all call a shared function that lives in `scripts/custom/_net_functions`, which in turn reads `configs/custom/_net_config`.  I wanted to keep a lot of the configuration that might change between installations in one place.  So, in the `configs/custom` directory, copy or rename `_net_config.template` to `_net_config`.  This is where you specify the configuration for your particular router.  If you have a RT-x66U, chances are you may only have to worry about `LAN_NET` unless your LAN network is `192.168.1.x`.

Make sure the scripts are executable, both in the main 'scripts' directory and also in 'scripts/custom'.

You should be ready to go!  You can try running the `wan-start` script manually to see if things run as expected.  Each of the scripts does a fair amount of logging to help troubleshoot.  You can grep for `admin:` to get an idea of what they're up to:

```
$ grep "admin:" /tmp/syslog.log | less
```

```
May  9 15:02:28 admin: /jffs/scripts/wan-start: kicking off vlan config
May  9 15:02:28 admin: udhcpc: running command: udhcpc -i vlan12 -p /var/run/udhcpc0.pid -V 2WHPL -s /jffs/scripts/custom/meo-post-dhcp-vlan-config
May  9 15:02:28 admin: /jffs/scripts/firewall-start: adding vlan12(udp) -> 224.0.0.0/4:1025 INPUT rule
May  9 15:02:28 admin: /jffs/scripts/firewall-start: adding vlan12:10.173.0.0/16 -> br0 FORWARD rule
May  9 15:02:28 admin: /jffs/scripts/firewall-start: adding vlan12:213.13.16.0/20 -> br0 FORWARD rule
May  9 15:02:28 admin: /jffs/scripts/firewall-start: adding vlan12:194.65.46.0/23 -> br0 FORWARD rule
```

If everything went well, the web UI's "Network Map" page should report a "Connected" Internet status.
On the command line, you should also see an VLAN12 interface with an IP:

```
$ ip address show vlan12
6: vlan12@eth0: <BROADCAST,MULTICAST,UP,10000> mtu 1500 qdisc noqueue 
    link/ether bc:ee:7b:7b:e7:40 brd ff:ff:ff:ff:ff:ff
    inet 10.243.x.y/18 brd 10.243.191.255 scope global vlan12
```

You could also confirm the connection to the VoIP proxy (from one of the computers on your LAN):

```
$ telnet proxy.ims.iptv.telecom.pt 5070
Trying 213.13.24.225...
Connected to proxy.ims.iptv.telecom.pt.
Escape character is '^]'.
^]
telnet> Connection closed.
```

Custom Script/Config Overview
-----------------------------

The various aspects of the configuration are triggered by the Merlin firmware, which looks for scripts at specific locations during specific events and executes them if they are present.

Let's go through a typical lifecycle of events to explain how these scripts work together:

1. Router is booting, and is ready to start services.  It first calls `init-start`:
  * init-start loads the `ebtables` kernel module
2. WAN interface comes up.  Merlin calls `wan-start`.
  * wan-start runs `udhcpc` (DHCP client) on VLAN12
  * udhcpc gets a VLAN12 IP address, gateway, DNS, then calls `scripts/custom/meo-post-dhcp-vlan-config` 
  * meo-post-dhcp-vlan-config:
    * Configures static routes for MEO services (`route add -net ...`)
    * Saves the VLAN12 IP address to `/tmp/vlan_ip`
3. The firewall filtering rules configured from the web UI have been applied.  Merlin calls `firewall-start`.
  * firewall-start configures iptables rules to allow MEO service traffic into the VLAN12 network 
4. NAT rules (i.e. port forwards, etc. configured in the web UI) are been applied.  Merlin calls `nat-start` so we can add our own.
  * nat-start configures iptables rules to nat-translate MEO service traffic from local LAN network to make it seem like it came from the VLAN12 network (using the IP address in `/tmp/vlan_ip` created by `meo-post-dhcp-vlan-config`)
  * nat-start configures ebtables rules to block multicast traffic from spamming the wireless (WLAN) network

Aside from the above scripts, this project has some config files that are read by the Merlin custom config process.  It basically looks for certain files in the `/jffs/configs`.  If any of the files end in ".add", the config is appended to the configuration file that Merlin creates on its own (from web UI settings).

1. dnsmasq.conf.add
  * Tells the LAN DHCP *server* to forward requests to "*.iptv.telecom.pt" to a DNS server running on the VLAN12 network instead of the default DNS server on VLAN10.

TODO
====

## IGMP in Web UI vs. Script

### IGMP Snooping

If we use the script, the rules are:

```
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     udp  --  vlan12 any     anywhere             base-address.mcast.net/4 udp dpts:1025:65535 
```

If we use the web UI "Advanced Settings &#8594; LAN &#8594; IPTV &#8594; Enable efficient multicast forwarding (IGMP Snooping)" option, the rules are a little more broad:

```
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     igmp --  any    any     anywhere             base-address.mcast.net/4 
    0     0 ACCEPT     udp  --  any    any     anywhere             base-address.mcast.net/4 udp dpt:!upnp 
```

# igmpproxy

If we use the web UI "Advanced Settings &#8594; LAN &#8594; IPTV &#8594; Enable multicast routing (IGMP Proxy)" option, the resulting config runs on vlan10 and on br0 instead of vlan12:

```
# automagically generated from web settings
quickleave

phyint vlan10 upstream  ratelimit 0  threshold 1
	altnet 0.0.0.0/0

phyint br0 downstream  ratelimit 0  threshold 1
```

I think we want to stick with our own, script-based, config and igmpproxy launcher for this.

Acknowledgements
================

Special thanks to Luis Fernandes, whose [blog posts](http://blog.zipleen.com/2010/10/how-to-make-meo-fiber-iptv-service-work.html) and [scripts](https://github.com/zipleen/tomato-ddwrt-meo-iptv-scripts) made this possible.

Other Fun Stuff
===============

## Isolating VLAN12

There are a few ways to allow a computer to "see" the VLAN12 traffic.  One is to [configure VLANs on the computer itself](http://www.ogris.de/howtos/macosx-tagged-vlans.html).  Another is to configure a router to make the VLAN12 appear as a "normal" network on one of its ports.


```
nvram set vlan12ports="0t 2t 3"
nvram set port2vlans=12
nvram set port3vlans=12
nvram set vlan12hwname=et0
nvram commit
# reboot
```

What the above command `vlan12ports="0t 2t 3"` says is, "Take the VLAN12 network which is trunked on port 0 (WAN), and 
duplicate as-is on port 2, and also duplicate it *not trunked* it on LAN port 3 (lack of "t"), which means that the duplicated traffic on port 3 will not be tagged, looking like "normal" traffic to a device plugged into that port.

Plug in the Thomson router on port 2, then a computer on port 3.  

On the computer, configure the interface with a static IP address (say 10.x.y.z) and netmask. 

You should be able to 'ping' the Thomson's VLAN12 IP address.

## tunlr-dyndns Branch

The `tunlr-dyndns` branch of this project contains a bit more complicated configuration.  The extra scripts/configs do the following:
- Update a Tunlr-Clone server with an updated IP or provider domains.  See [tunlr-utils](https://github.com/twelve17/tunlr-utils).
- Run a dynamic DNS client compatible with [DNS Made Easy](dnsmadeasy.net) (guess it wasn't *that* easy, huh).

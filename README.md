merlin-meo-scripts
==================

An adaptation of ziplee's  [tomato-ddwrt-meo-iptv-scripts](https://github.com/zipleen/tomato-ddwrt-meo-iptv-scripts) to work with routers running [Merlin firmware](http://www.lostrealm.ca/tower/node/79).

These scripts were tested with Merlin firmware version `374.41` running on a RT-AC66U.

## Installation

To sum up the process outlined in [this very informative blog post](http://blog.zipleen.com/2010/10/how-to-make-meo-fiber-iptv-service-work.html), we need to do the following to get the various [MEO](http://meo.pt) services working: 

* Make the router aware of MEO's vlans.
* Configure Internet access using PPPoE
* Run a custom DHCP client to get the network configuration via vlan12 (IPTV + VoIP)
* Configure static routes, firewall/NAT rules, and DNS overrides For specific MEO-related IP ranges

Most of the work is going to be done on the command line, *not* on the web admin UI, so you should feel comfortable SSH'ing into your router.  The scripts leverage the Merlin [user scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) as well as [custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) mechanisms to accomplish the same goals as the zipleen versions.  Either way, use at your own risk, don't blame us if you brick your router, etc.  :)

Configure VLANs
---------------

### Internet

The router needs to be configured to get its Internet access via VLAN10, rather than the default (untagged) network.  As of this writing, this can be done in the web admin UI.  

In the `Advanced Settings &#8594; LAN &#8594; IPTV &#8594; Port`.  Under `Select ISP Profile`, select "Manual".  You'll see a few text input fields.  For `Internet`, enter "10".

I'm not sure if this part matters yet, but under `Special Applications`, disable "Use DHCP Routes".

### IPTV/VoIP (CLI)

The web UI is somewhat limited with VLAN configuration, so IPTV/VoIP configuration needs to be done on the command line.

**Note**: this part may be specific to the ASUS RT-AC66U/RT-N66U.  Refer to the [Switched Ports](http://www.dd-wrt.com/wiki/index.php/Switched_Ports) article for the values that may apply to your router.

On the CLI, run the following:

```
nvram set vlan12ports="0t 8"
nvram set vlan12hwname=et0
```
*Be sure to use quotes on the first one since there is a space in the value.*

A little explanation:

Some routers allow you to replicate the signal from one port to another.  The RT-AC66U, for example, has five network ports: 1 WAN and 4 LAN ports.  The nvram values for these ports are 0 for the WAN port, and 1-4 for the four LAN ports.

Port #8 is a special port (called the CPU internal port).  If you want the router to interact with any of the signals on the ports, it needs to be "plugged into" the CPU internal port.  If omitted, the router will pass along the signal from one external port to another and not pay attention to it.  

So, `vlan12ports="0t 8"` means "Take the vlan12 network signal and make it available to the router itself.", which then allows us to forward the signal to the LAN later.   The "t" in the "0t" means that the incoming signal to the WAN port is "trunked", which just means that the incoming signal will have tagged vlan traffic (e.g. multiple networks) in it.  

#### Alternative for VoIP

Let's say you wish to keep using your Thomson router as the VoIP client.  You could forward *just* vlan12 to to one of the router's LAN ports:

```
nvram set vlan12ports="0t 4t 8"
nvram set port4vlans=12
nvram set vlan12hwname=et0
```

Now, the `vlan12ports` command is essentially telling the router to duplicate the trunked network on the WAN port 0 to the LAN port 4 (in addition to itself via port 8).  You can then plug in the Thomson router to LAN port 4, and it should be able to acces the VoIP service.  Note that since only VLAN12 is being forwarded, the router will not be able to connect to the Internet, but it does not need it to use the VoIP service (and perhaps IPTV, but that has not been tested here!).






https://github.com/twelve17/merlin-meo-scripts/blob/tunlr-dyndns/scripts/custom/meo-config-voip-vlan-ports



#### SIP Client via LAN/WLAN


#### Technicolor Router As SIP Client


tunlr-dyndns
-------------

The `tunlr-dyndns` branch contains extra scripts/configs to:
- Update a Tunlr-Clone server with an updated IP or provider domains.  See [tunlr-utils](https://github.com/twelve17/tunlr-utils).
- Run a dynamic dns client compatible with dnsmadeasy.net (guess it wasn't *that* easy, huh).


## Installation

Place the 'configs' and 'scripts' folders in the /jffs directory on your router.

## TODO

A ton of documentation.



#### Alternative #2: Debugging 

Let's say you wish to plug in the WAN port of the Thomson router to a computer.  Because the Thomson router is configured to expect the various VLANs, either you'd have to configure VLAN virtual interfaces on the computer, or, you could instead tell your router 

merlin-meo-scripts
==================

An adaptation of ziplee's  [tomato-ddwrt-meo-iptv-scripts](https://github.com/zipleen/tomato-ddwrt-meo-iptv-scripts) to work with routers running [Merlin firmware](http://www.lostrealm.ca/tower/node/79).

These scripts were tested with Merlin firmware version `374.41` running on a RT-AC66U.

## Installation

To sum up the process outlined in [this very informative blog post|http://blog.zipleen.com/2010/10/how-to-make-meo-fiber-iptv-service-work.html], we need to do the following to get the various [MEO|http://meo.pt] services working: 

* Make the router aware of MEO's vlans.
* Configure Internet access using PPPoE
* Run a custom DHCP client to get the network configuration via vlan12 (IPTV + VoIP)
* Configure static routes, firewall/NAT rules, and DNS overrides For specific MEO-related IP ranges

Most of the work is going to be done on the command line, *not* on the Web UI, so you should feel comfortable SSH'ing into your router.  The scripts leverage the Merlin [user scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) as well as [custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) mechanisms to accomplish the same goals as the zipleen versions.


tunlr-dyndns
-------------

The `tunlr-dyndns` branch contains extra scripts/configs to:
- Update a Tunlr-Clone server with an updated IP or provider domains.  See [tunlr-utils](https://github.com/twelve17/tunlr-utils).
- Run a dynamic dns client compatible with dnsmadeasy.net (guess it wasn't *that* easy, huh).


## Installation

Place the 'configs' and 'scripts' folders in the /jffs directory on your router.

## TODO

A ton of documentation.

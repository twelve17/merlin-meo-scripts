merlin-meo-scripts
==================

Scripts to configure an ASUS router running the [Merlin firmware](http://www.lostrealm.ca/tower/node/79) to work with MEO (Portuguese ISP).

They are an adaptation from the [tomato-ddwrt-meo-iptv-scripts](https://github.com/zipleen/tomato-ddwrt-meo-iptv-scripts) versions by zipleen, which are made for Tomato/DD-WRT firmwares.

This adaptation leverages the Merlin [user scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) as well as [custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) mechanisms to accomplish the same goas as the zipleen versions.

These scripts were tested with the Merlin firmware version `374.41` running on a RT-AC66U.

tunlr-dyndns
-------------

The `tunlr-dyndns` branch contains extra scripts/configs to:
- Update a Tunlr-Clone server with an updated IP or provider domains.  See [tunlr-utils](https://github.com/twelve17/tunlr-utils).
- Run a dynamic dns client compatible with dnsmadeasy.net (guess it wasn't *that* easy, huh).


## Installation

Place the 'configs' and 'scripts' folders in the /jffs directory on your router.

## TODO

A ton of documentation.

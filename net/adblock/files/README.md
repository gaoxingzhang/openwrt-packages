# adblock script for openwrt

## Description
A lot of people already use adblocker plugins within their desktop browsers,  
but what if you are using your (smart) phone, tablet, watch or any other wlan gadget...getting rid of annoying ads, trackers and other abuse sites (like facebook ;-) is simple: block them with your router.  

When the dns server on your router receives dns requests, you will sort out queries that ask for the resource records of ad servers and return the local ip address of your router and the internal web server delivers a transparent pixel instead.

## Main Features
* support of the following domain blocklist sources (free for private usage, for commercial use please check their individual licenses):
    * [adaway](https://adaway.org)
    * => infrequent updates, approx. 400 entries (enabled by default)
    * [disconnect](https://disconnect.me)
    * => numerous updates on the same day, approx. 6.500 entries (enabled by default)
    * [dshield](http://dshield.org)
    * => daily updates, approx. 4.500 entries
    * [feodotracker](https://feodotracker.abuse.ch)
    * => daily updates, approx. 0-10 entries
    * [malwaredomains](http://malwaredomains.com)
    * => daily updates, approx. 16.000 entries
    * [malwaredomainlist](http://www.malwaredomainlist.com)
    * => daily updates, approx. 1.500 entries
    * [openphish](https://openphish.com)
    * => numerous updates on the same day, approx. 1.800 entries
    * [palevotracker](https://palevotracker.abuse.ch)
    * => daily updates, approx. 15 entries
    * [ruadlist+easylist](https://code.google.com/p/ruadlist)
    * => weekly updates, approx. 2.000 entries (experimental support, may include false positives!)
    * [shallalist](http://www.shallalist.de) (categories "adv" "costtraps" "spyware" "tracker" "warez" enabled by default)
    * => daily updates, approx. 32.000 entries (a short description of all shallalist categories can be found [online](http://www.shallalist.de/categories.html))
    * [spam404](http://www.spam404.com)
    * => infrequent updates, approx. 5.000 entries
    * [whocares](http://someonewhocares.org)
    * => weekly updates, approx. 12.000 entries
    * [winhelp](http://winhelp2002.mvps.org)
    * => infrequent updates, approx. 15.000 entries
    * [yoyo](http://pgl.yoyo.org/adservers)
    * => weekly updates, approx. 2.500 entries (enabled by default)
    * [zeustracker](https://zeustracker.abuse.ch)
    * => daily updates, approx. 440 entries
* zero-conf like automatic installation & setup, usually no manual changes needed (i.e. ip address, network devices etc.)
* full IPv4 and IPv6 support
* each blocklist source will be updated and processed separately
* timestamp check to download and process only updated adblock list sources
* overall duplicate removal in separate adblock lists (will be automatically disabled on low memory systems)
* adblock source list parsing by fast & flexible regex rulesets
* additional white- and blacklist support for manual overrides
* quality checks during & after update of adblock lists to ensure a reliable dnsmasq service
* wan update check, to wait for an active wan uplink before update
* basic adblock statistics via iptables packet counters for each chain
* status & error logging to stdout and syslog
* use of dynamic uhttpd instance as adblock pixel server
* openwrt init system support (start/stop/restart/reload)
* optional features (disabled by default):
    * adblock list backup/restore
    * debug logging to separate file

## Prerequisites
* [openwrt](https://openwrt.org), tested with latest stable release (Chaos Calmer 15.05) and with current trunk (Designated Driver > r47025)
* usual openwrt setup with 'iptables' & 'uhttpd', additional required software packages:
    * wget
    * optional: 'kmod-ipt-nat6' for IPv6 support
* the above dependencies and requirements will be checked during package installation & script runtime

## Usage
* install the adblock package (*opkg install adblock*)
* start the adblock service with */etc/init.d/adblock start* and check *logread -e "adblock"* for adblock related information
* optional: enable/disable your required adblock list sources in */etc/config/adblock* - 'adaway', 'disconnect' and 'yoyo' are enabled by default
* optional: maintain the adblock service in luci under 'System => Startup'

## Tweaks
* there is no need to enable all blacklist sites at once, for normal use one to three adblock list sources should be sufficient
* if you really need to handle all blacklists at once add an usb stick or any other storage device to supersize your temp directory with a swap partition => see [openwrt wiki](https://wiki.openwrt.org/doc/uci/fstab) for further details
* add static, personal domain white- or blacklist entries, one domain per line (wildcards & regex are not allowed!), by default both lists are located in */etc/adblock*
* enable the backup/restore feature, to restore automatically the latest, stable backup of your adblock lists in case of any processing error
* enable the logging feature for continuous logfile writing to monitor the adblock runs over a longer period
* for a scheduled call of the adblock service via */etc/init.d/adblock start* add an appropriate crontab entry

## Further adblock config options
* usually the adblock autodetection works quite well and no manual config overrides are needed, all options apply to 'global' adblock config section:
    * adb\_enabled => main switch to enable/disable adblock service (default: '1' (enabled))
    * adb\_cfgver => config version string (do not change!) - adblock checks this entry and automatically applies the current config, if none or an older revision was found.
    * adb\_wanif => name of the logical wan interface (default: 'wan')
    * adb\_lanif => name of the logical lan interface (default: 'lan')
    * adb\_port => port of the adblock uhttpd instance (default: '65535')
    * adb\_nullipv4 => IPv4 blackhole ip address (default: '192.0.2.1')
    * adb\_nullipv6 => IPv6 blackhole ip address (default '::ffff:c000:0201')
    * adb\_maxtime => download timeout limit in seconds (default: '60')
    * adb\_maxloop => startup timeout limit in seconds to wait for an active wan interface (default: '20')

## Background
This adblock package is a dns/dnsmasq based adblock solution for openwrt.  
Queries to ad/abuse domains are never forwarded and always replied with a local IP address which may be IPv4 or IPv6.  
For that purpose adblock uses an ip address from the private 'TEST-NET-1' subnet (192.0.2.1 / ::ffff:c000:0201) by default.  
Furthermore all ad/abuse queries will be filtered by ip(6)tables and redirected to internal adblock pixel server (in PREROUTING chain) or rejected (in FORWARD or OUTPUT chain).  
All iptables and uhttpd related adblock additions are non-destructive, no hard-coded changes in 'firewall.user', 'uhttpd' config or any other openwrt related config files.  
There is *no* adblock background daemon running, the (scheduled) start of the adblock service keeps only the adblock lists up-to-date.

## Support
Please join the adblock discussion in this [openwrt forum thread](https://forum.openwrt.org/viewtopic.php?id=59803) or contact me by mail <openwrt@brenken.org>

## Removal
* stop all adblock related services with */etc/init.d/adblock stop*
* optional: remove the adblock package (*opkg remove adblock*)

Have fun!  
Dirk  

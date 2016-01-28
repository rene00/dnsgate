
# dnsgate

**dnsgate** merges 3rd party [1] DNS blocking lists into `/etc/dnsmasq.conf` or `/etc/hosts` format.

While not required, [dnsmasq](https://wiki.gentoo.org/wiki/Dnsmasq) improves on conventional [/etc/hosts domain blocking](http://winhelp2002.mvps.org/hosts.htm) by enabling * blocking of domains.

For example *.google.com:

```
echo 'address=/.google.com/127.0.0.1' >> /etc/dnsmasq.conf
```
Returnins 127.0.0.1 for all google.com domains. Rather than return 127.0.0.1 and make the application chack if port 80/443/whatever is open, dnsmasq has another advantage; it can return NXDOMAIN:

```
echo 'server=/.google.com/' >> /etc/dnsmasq.conf
```
This instead returns NXDOMAIN on *.google.com. Returning NXDOMAIN instead of localhost is default in dnsmasq output mode.

Said another way, conventional `/etc/hosts` blocking can not use wildcards * and therefore someone must keep track of each subdomain / domain combination that should be blocked. This is not necessarily a problem. Even if you don't use dnsmasq, other people [1] keep track of the subdomains for you and dnsgate automatically pulls from them. If you want to block a specific domain completely, you must use dnsmasq.

With `--mode dnsmasq` (which is default) `--block-at-psl` strips domains to their "Public Second Level Domain" which is the top public domain with any subdomain stripped, removing the need to manually specify/track specific subdomains. `--block-at-psl` may block domain's you want to use, so use it with `--whitelist`.

**Features:**
* **Persistent Configuration** see configure.
* **Wildcard Blocking** --block-at-psl will block TLD's instead of individual subdomains (dnsmasq mode only).
* **System-wide.** All programs that use the local DNS resolver benefit.
* **Blacklist Caching.** Optionally cache and re-use remote blacklists (see --no-cache and --cache-expire).
* **Non-interactive.** Can be run as a periodic cron job.
* **Fully Configurable.** See ./dnsgate --help (TODO, add /etc/dnsgate/config file support)
* **Custom Lists.** See /etc/dnsgate/blacklist and /etc/dnsgate/whitelist.
* **Return NXDOMAIN.** Rather than redirect the request to 127.0.0.1, NXDOMAIN is returned. (dnsmasq mode only).
* **Return Custom IP.** --dest-ip allows redirection to specified IP (disables returning NXDOMAIN in dnsmasq mode)
* **Installation Support.** see install-help
* **Verbose Output.** see --verbose
* **IDN Support.** What to block snowman? ./dnsgate blacklist ☃.net
* **Enable/Disable Support.** See enable and disable (dnsmasq mode only)

**TODO:**
* **Full Test Coverage.**

**Dependencies:**
 - [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) (optional)
 - python3 (tested on 3.4)
 - [click](https://github.com/mitsuhiko/click)
 - [tldextract](https://github.com/john-kurkowski/tldextract)

```
Usage: dnsgate [OPTIONS] COMMAND [ARGS]...

  dnsgate combines, deduplicates, and optionally modifies local and remote DNS blacklists. Use "dnsgate
  (command) --help" for more information.

Options:
  --no-restart-dnsmasq  do not restart the dnsmasq service
  --backup              backup output file before overwriting
  --verbose             print debug information to stderr
  --help                Show this message and exit.

Commands:
  blacklist     Add domain(s) to /etc/dnsgate/blacklist
  configure     Write /etc/dnsgate/config
  disable       Disable /etc/dnsgate/generated_blacklist
  enable        Enable /etc/dnsgate/generated_blacklist
  generate      Create /etc/dnsgate/generated_blacklist
  install_help  Help configure dnsmasq or /etc/hosts
  whitelist     Add domain(s) to /etc/dnsgate/whitelist
```
 
**create dnsgate configuration file for --mode dnsmasq:**
 
```  
$ ./dnsgate configure --mode dnsmasq
``` 
**dnsmasq examples that do the same thing:**
 
```  
$ ./dnsgate generate
 * Stopping dnsmasq ... [ ok ]
 * Starting dnsmasq ... [ ok ]
  
$ ./dnsgate --verbose generate
Using output file: /etc/dnsgate/generated_blacklist
102 validated whitelist domains.
Reading remote blacklist(s):
['http://winhelp2002.mvps.org/hosts.txt', 'http://someonewhocares.org/hosts/hosts']
23647 domains from remote blacklist(s).
23647 validated remote blacklisted domains.
23646 blacklisted domains after subtracting the 102 whitelisted domains
Re-adding 64 domains in the local blacklist /etc/dnsgate/blacklist to override the whitelist.
23705 blacklisted domains after re-adding the custom blacklist.
8445 blacklisted domains after removing redundant rules.
Sorting domains by their subdomain and grouping by TLD.
Final blacklisted domain count: 8445
Writing output file: /etc/dnsgate/generated_blacklist in dnsmasq format
 * Stopping dnsmasq ... [ ok ]
 * Starting dnsmasq ... [ ok ]
``` 
**dnsmasq install help:**
 
```  
$ ./dnsgate install_help
    $ cp -vi /etc/dnsmasq.conf /etc/dnsmasq.conf.bak.1453965802.894224
    $ grep conf-dir=/etc/dnsmasq.d /etc/dnsmasq.conf|| { echo conf-dir=/etc/dnsmasq.d >> dnsmasq_config_file ; }
    $ /etc/init.d/dnsmasq restart
``` 
**create dnsgate configuration file for --mode hosts:**
 
```  
$ ./dnsgate configure --mode hosts
``` 
**hosts examples that do the same thing:**
 
```  
$ ./dnsgate generate
  
$ ./dnsgate --verbose generate
Using output file: /etc/dnsgate/generated_blacklist
102 validated whitelist domains.
Reading remote blacklist(s):
['http://winhelp2002.mvps.org/hosts.txt', 'http://someonewhocares.org/hosts/hosts']
23647 domains from remote blacklist(s).
23647 validated remote blacklisted domains.
23646 blacklisted domains after subtracting the 102 whitelisted domains
Re-adding 64 domains in the local blacklist /etc/dnsgate/blacklist to override the whitelist.
23705 blacklisted domains after re-adding the custom blacklist.
8445 blacklisted domains after removing redundant rules.
Sorting domains by their subdomain and grouping by TLD.
Final blacklisted domain count: 8445
Writing output file: /etc/dnsgate/generated_blacklist in hosts format
``` 
**/etc/hosts install help:**
 
```  
$ ./dnsgate install_help
    $ mv -vi /etc/hosts /etc/hosts.default
    $ cat /etc/hosts.default /etc/dnsgate/generated_blacklist > /etc/hosts
``` 



**More Information:**
 - https://news.ycombinator.com/item?id=10572700
 - https://news.ycombinator.com/item?id=10369516

**Related Software:**
 - https://github.com/gaenserich/hostsblock (bash)
 - http://pgl.yoyo.org/as/#unbound
 - https://github.com/longsleep/adblockrouter
 - https://github.com/StevenBlack/hosts (TODO, use this)
 - https://github.com/Mechazawa/FuckFuckAdblock

**Simple Software:**

If you find this useful you may appreciate:

 - Disabling JS and making a keybinding to enable it ([surf](http://surf.suckless.org/)+[tabbed](http://tools.suckless.org/tabbed/) makes this easy)
 - [musl](http://wiki.musl-libc.org/wiki/Functional_differences_from_glibc#Name_Resolver_.2F_DNS)

[1]: http://winhelp2002.mvps.org/hosts.txt http://someonewhocares.org/hosts/hosts



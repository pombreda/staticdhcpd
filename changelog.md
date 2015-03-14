# Content relocated #
From this point forth, all changelog history is now found in `debian/changelog`. This page is no longer maintained, but is preserved for historical integrity.

Changelogs:
  * [staticDHCPd](http://code.google.com/p/staticdhcpd/source/browse/branches/2.0.0/staticDHCPd/debian/changelog)
  * [libpydhcpserver](http://code.google.com/p/staticdhcpd/source/browse/branches/2.0.0/libpydhcpserver/debian/changelog)


---


# 1.6.4 #
  * Backported a feature from 2.0.0 that lets users define their own custom database engines without needing to touch the core code.

---

# 1.5.8, 1.6.3 #
  * Fixed a long-standing issue in which PXE clients that sent broadcast packets containing `ciaddr` were responded to via broadcast, rather than unicast to that `ciaddr` value.

---

# 1.6.2 : staticDHCPd #
  * Refactored the database component, breaking coupling to SQL
  * Implemented support for using an INI file as an alternative to an SQL-based database, borrowing from contributions by John Stowers

---

# 1.6.1 : staticDHCPd #
  * Massive re-architecture of the directory structures
    * staticDHCPd and libpydhcpserver now separated
    * conf/ directory added for source-based testing and deployment
    * /etc/staticDHCPd/ now recognised as a configuration path
  * Better deployment and operation logic
    * Setup scripts for both components of the system
    * Basic unified installer
      * Upgrade-friendly
    * Platform auto-detection and initscript-setup-help
    * No more need for hacks to access custom modules imported relative to `conf.py`
  * Fixed a tab-based indentation issue in 1.6.0 (thanks to John Stowers for pointing it out)
  * `main.py` is now `staticDHCPd`

---

# 1.6.0 (svn-only) #
  * Support for dynamic provisioning.
  * Support for true daemon operation. (Not strictly necessary, but I just realised I somehow left it out, which is odd because it's in everything else I've written)

---

# 1.5.7 #
  * PXE will now work even when ALLOW\_LOCAL\_DHCP is False. This failed before because PXE is strictly unicast, so it doesn't use a relay, even when the other half of the negotiation does.

---

# 1.5.6 #
  * Fixes a significant issue that prevented special DHCP options from being parsed, thanks to Aditya Kulkarni.
  * Fixes an issue that, despite what 1.5.3 said, prevented PXE from working properly, since the socket was never checked for pending data.
  * Fixes the lack of e-mail timeouts and allows ports to be specified.

---

# 1.5.5 #
  * Primarily a bugfix release, and one long overdue.
    * Addresses a number of issues affecting REBIND behaviour and errant NAKs ([issue 7](https://code.google.com/p/staticdhcpd/issues/detail?id=7)).
    * Adds Python 2.5 backwards-compatibility thanks to a patch from Mark Schloesser.
    * Fixes an error-logging issue that caused still more errors to be thrown, thanks again to Mark ([issue 8](https://code.google.com/p/staticdhcpd/issues/detail?id=8)).
    * Fixes issues related to in-code typos that prevent custom attributes from being set correctly in DHCP packets ([issue 10](https://code.google.com/p/staticdhcpd/issues/detail?id=10)).
  * Added some diagnosis logic and a strToPaddedList() function based on patches from Andrew Xuan.

---

# 1.5.4 #
  * Made it easy to upgrade by introducing a buffering setting-default system
  * Refactored some code out of places it didn't need to be
  * Severed a reverse-dependency between libpydhcpserver and staticDHCPd

---

# 1.5.3 (svn-only) #
  * Vendor options are automatically stripped from received packets and exposed to users through a `vendor` parameter
  * Support for a PXE port has been added; see the FAQ for more details
    * Addresses an issue reported by Andrew Xuan
  * Added support for easy logging from user-defined code

---

# 1.5.2 (svn-only) #
  * Addressed an issue with Cisco relays not accepting packets from sockets other than 67
  * Added support for Oracle

---

# 1.5.1 #
  * Fixed a bug with Postgres's configuration, preventing it from being used unless MySQL was configured, too
  * Fixed a documentation error in the README: no netmask was defined

---

# 1.5.0 #
  * Database-flow overhaul
    * Added support for Postgres
    * Added connection pooling
    * Removed concurrency support from SQLite, since it was redundant
  * Converted all source to space-based indentation (you'll want to whitespace-ignorant diff your old <tt>conf.py</tt> files)
  * Cleaned up webserver logging a little

---

# 1.5.0 (preview) #
  * Refactored pydhcplib into libpydhcpserver
    * All code thoroughly commented.
  * Added support for [RFC3442](http://www.rfc-archive.org/getrfc.php?rfc=3442) (option 121)
    * The data to be sent must be crafted by hand.
    * If there is any demand, a special class will be created.
  * Added support for [RFC3495](http://www.rfc-archive.org/getrfc.php?rfc=3495) (option 122)
    * The data to be sent must be crafted by hand.
  * Added support for [RFC3825](http://www.rfc-archive.org/getrfc.php?rfc=3825) (option 123)
    * The data to be sent must be crafted by hand.
    * A special class will be handled to ease this process in a later version.
  * Added support for [RFC3925](http://www.rfc-archive.org/getrfc.php?rfc=3925) (options 124 and 125)
    * The data to be sent must be crafted by hand.
  * Added support for [RFC4039](http://www.rfc-archive.org/getrfc.php?rfc=4039) (option 80)
  * Added support for [RFC4280](http://www.rfc-archive.org/getrfc.php?rfc=4280) (options 88 and 89)
  * Added support for [RFC4578](http://www.rfc-archive.org/getrfc.php?rfc=4578) (options 93, 94, and 97)
  * Added support for [RFC4776](http://www.rfc-archive.org/getrfc.php?rfc=4776) (option 99)
    * The data must be crafted by hand, due to the complexity of the option.
    * If there is sufficient demand, a special class may be created.
  * Added support for [RFC4833](http://www.rfc-archive.org/getrfc.php?rfc=4833) (options 100 and 101)
  * Added support for [RFC5071](http://www.rfc-archive.org/getrfc.php?rfc=5071) (options 208-211)
  * Added support for [RFC5192](http://www.rfc-archive.org/getrfc.php?rfc=5192) (option 136)
  * Added support for [RFC5223](http://www.rfc-archive.org/getrfc.php?rfc=5223) (option 137)
  * Added support for [RFC5417](http://www.rfc-archive.org/getrfc.php?rfc=5417) (option 138)

---

# 1.4.1 #
  * 1.4.0's new RFC options can now be set using lists of bytes

---

# 1.4.0 #
  * Added support for [RFC2610](http://www.rfc-archive.org/getrfc.php?rfc=2610) (options 78 and 79)
  * Added support for [RFC2937](http://www.rfc-archive.org/getrfc.php?rfc=2937) (option 117)
  * Added support for [RFC3046](http://www.rfc-archive.org/getrfc.php?rfc=3046) (option 82)
  * Added support for [RFC3361](http://www.rfc-archive.org/getrfc.php?rfc=3361) (option 120)
  * Added support for [RFC3396](http://www.rfc-archive.org/getrfc.php?rfc=3396) (long options)
  * Added support for [RFC4174](http://www.rfc-archive.org/getrfc.php?rfc=4174) (option 83)
  * Added BETA support for [RFC4388](http://www.rfc-archive.org/getrfc.php?rfc=4388) (LEASEQUERY)
    * Caveat: only MAC-based lookups are supported; anything else will be served a <tt>DHCPLEASEUNKNOWN</tt> out of necessity
  * Added support for specification of options by number
  * Rebuilt support for [RFC3397](http://www.rfc-archive.org/getrfc.php?rfc=3397) (option 119), with the caveat that the compression algorithm it describes was omitted

---

# 1.3.5 #
  * Updated support and resource URLs

---

# 1.3.4 #
  * Fixed a logic error that resulted in non-RFC-compliant NAKing
    * No clients seem to have noticed or cared, though

---

# 1.3.3 #
  * Fixed a typo that prevented NAKing of unknown MACs in <tt>AUTHORITATIVE</tt> mode

---

# 1.3.2 #
  * Improved concurrent efficiency of the <tt>DHCPDECLINE</tt> and <tt>DHCPRELEASE</tt> events
  * Added better reporting of misconfigured/malicious clients that would break dynamic DHCP servers
  * Added better handling for misconfigured/malicious clients
  * More consistent message formatting

---

# 1.3.1 #
  * Added reporting for <tt>DHCPRELEASE</tt> events, to make it easier to determine whether clients are behaving properly
  * Added reporting for <tt>DHCPDECLINE</tt> events, complete with e-mail notification, to help operators find conflicts in their networks
  * Improved log formatting
  * Improved memory-efficiency of caching mode

---

# 1.3.0 #
  * Added <tt>AUTHORITATIVE</tt> mode, allowing staticDHCPd to be configured to NAK DISCOVERs for unknown MACs, rather than silently ignoring them
  * Added an option to have logfiles written to disk with the current timestamp as a suffix, simplifying the process of creating status snapshots

---

# 1.2.0 #
  * Consistently named and documented convenience functions

---

# 1.1.1 #
  * Added <tt>README</tt> with a quick-start proof-of-concept-for-busy-people guide

---

# 1.1.0 #
  * <tt>USE_CACHE</tt> option added to <tt>conf.py</tt>, allowing for memory-speed performance under heavy load

---

# 1.0.0 #
  * First stable release
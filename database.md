# **DEPRECATED** #
This page will be removed when 2.0.0 is released. Its content has [moved](http://static.uguu.ca/projects/staticDHCPd/doc/commentary/database.html).

This article explains how to set values in the database to provide DHCP-handling for staticDHCPd's clients.


---

# Background #
Knowledge of how to maintain databases using the engine you've chosen is a necessary pre-requisite.


---

# <tt>subnets</tt> table #
At least one entry in this table is required before any values can be added to the maps table.

  * <tt>subnet</tt>
    * While it's generally a good idea to use a value like <tt>192.168.0.0/24</tt> so you know, at a glance, what subnet its clients should be on, it is perfectly legal to set a value like <tt>my subnet</tt>.
    * This field is just free-form text.
  * <tt>serial</tt>
    * This field may be used to separate a subnet into partitions to do things like set different default gateways to reduce load on servers.
  * <tt>lease_time</tt>
    * Contains the number of seconds for which clients will believe their "leases" to be valid; by default, <tt>T1</tt> is half of this, so stable clients may update their information in as little as half this time.
  * <tt>gateway</tt>
    * May be an IPv4 or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>subnet_mask</tt>
    * May be an IPv4 or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>broadcast_address</tt>
    * May be an IPv4 or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>ntp_servers</tt>
    * May be up to three IPv4s, separated by commas, or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>domain_name_servers</tt>
    * May be up to three IPv4s, separated by commas, or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>domain_name</tt>
    * May be any arbitrary, FQDN-valid string or may be NULL to avoid setting the corresponding DHCP option.


---

# <tt>maps</tt> table #
Every entry in this table must be bound to a subnet/serial pair in the subnets table, from which options will be inherited.

  * <tt>mac</tt>
    * Must be a lower-case, colon-separated MAC address.
  * <tt>ip</tt>
    * Must be a dot-separated IPv4 address without any unnecessary 0s.
  * <tt>hostname</tt>
    * May be a string or may be NULL to avoid setting the corresponding DHCP option.
  * <tt>subnet</tt>
    * Must correspond to an entry in the subnets table.
  * <tt>serial</tt>
    * Must correspond to an entry in the subnets table.
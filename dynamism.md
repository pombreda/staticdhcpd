# **DEPRECATED** #
This page will be removed when 2.0.0 is released. Its content has [moved](http://static.uguu.ca/projects/staticDHCPd/doc/commentary/setups.html).

# Overview #
staticDHCPd is meant to help administrators easily configure static environments, with easy-to-integrate provisioning facilities. However, special cases have arisen that require dynamic behaviour on limited scales.

The motivating case for adding support for dynamic provisioning to staticDHCPd 1.6.0 is a LAN party context in which guests need to register their systems before they can be given a static mapping by site administration. Using the new facility, unknown clients can be given configuration that puts them into an isolated subnet on a short lease so they can access a registration system, and the DHCP server itself can send notification of the new arrival to a webservice to streamline the operators' work.

The rest of this article outlines how to use the sample configurations provided with staticDHCPd. Any site seeking to use dynamic services will almost certainly need to do some customisation, though, so consider at least basic Python knowledge to be a pre-requisite.

Be aware also that, unlike dynamic-provisioning-focused servers, like the ISC's, not all provisioning semantics are respected and that, unlike staticDHCPd's static behaviour, this facet of the system is **not** RFC-compliant. It probably won't do anything environment-breaking, but be prepared for some weird things; feedback, if you encounter any, is very welcome.


---

# Modules #
## In-memory ##
Provided as `contrib/dynamism.py`, this is the reference module for dynamic allocation, offering management for leases in a single-DHCP-server environment.

To use this module, copy it to the same directory as `conf.py`, the follow the documentation at the start of the file.

That should be it for the basic use-case.  If you want to do anything cool, like send a JSON message to a webservice when an unknown MAC appears or block clients after they renew five times, subclass DynamicPool or just hack it in-place. It's simple code and it's your environment, so just apply what you find in tutorials on the Internet and have fun.

### How stable are dynamic leases? ###
With 1.6.0, not very. If you were to restart the server, everything would be lost. You'd get some DECLINEs, then stuff would start working again.

With 2.0.0, they should be pretty consistent: when IPs are added to the pool, if [scapy](http://www.secdev.org/projects/scapy/) is available, and if the optional scan option is left enabled, the server will ARP for each address (in parallel, so it's not slow), setting up leases as hits are found, making your network a living database. Additionally, if a client requests a specific IP after the server is already online (clients often do this when rebooting), that address will be plucked if available.

### Is DST a factor with leases? ###
No, DST shouldn't be an issue. Internally, leases are managed as offsets against UTC, so timezones are only applied when formatting the timestamps for presentation to operators.
# **DEPRECATED** #
This page will be reduced significantly when 2.0.0 is released. Most of its content has [moved](http://static.uguu.ca/projects/staticDHCPd/doc/commentary/faq.html).

# Release errata #
## [RFC4388](http://www.rfc-archive.org/getrfc.php?rfc=4388) ##
The featureset described by this RFC, LEASEQUERY, is untested, yet has been included in versions 1.4.0+, but was removed in 1.6.3, because its implementation was wrong. It will likely return soon, but better to leave out bad code than try to hack it into a semi-working state.


---

# Known issues #
There are no known issues at this time.


---

# Scope #
## Can I do dynamic provisioning? ##
Starting with 1.6.0, support for limited, non-spec-compliant dynamic provisioning is provided. `conf.py` now has an optional `handleUnknownMAC()` function, which can be used to return binding information, in the event that the database contains nothing. How you use this is up to you, since it's pretty flexible, but it will likely have unexpected results compared to real dynamic DHCP servers; problem reports are very much welcome, if only so they can be documented.

To see examples of how to use this facility, see the [dynamic configuration](dynamism.md) page.

## Can I connect to an unsupported database, like a webservice? ##
Absolutely.

Starting with 1.6.4, you can define your own database engine, as long as it implements staticDHCPd's database interface, and reference it in `conf.py`, with no need to carry patches against the core codebase.


---

# Configuration #
## I need to boot clients over my network. How can I set up PXE? ##
Starting with 1.5.2, two methods are provided for setting up PXE and the main hurdle to getting it running (the fact that staticDHCPd was blindly passing vendor options back to the client out of ignorance) is gone.

In general, it should be sufficient to test for option 60 (vendor\_class\_identifier) in `conf.py:loadDHCPPacket()` to see if it matches the device-type you want to net-boot and set options 60, 66 (tftp\_server\_name), and 67 (bootfile\_name) accordingly, as demonstrated in the following example:

```
if vendor[1] == 'Aastra 57i': #Note: these phones may not actually announce themselves this way; this is a guess
    packet.setOption('vendor_class_identifier', strToList('PXEClient'))
    packet.setOption('tftp_server_name', strToList('bootserver.example.org')) #IPs are valid, too
    packet.setOption('bootfile_name', strToList(mac.replace(':', '').upper() + '.cfg')) #Have the device ask for its own MAC, stripped of colons and uppercased
```

Of course, you can use other criteria (or none at all, if you're sure all served clients will need the same TFTP settings) to evaluate whether the options should be set.


In the event that the client tries to hit a DHCP proxy port (4011, by convention), you'll need to edit `conf.py` and assign the port number to `PXE_PORT`. This will cause staticDHCPd to bind another port on the same interface(s) as the main DHCP port. Because devices that disregard PXE convention are likely to be a little temperamental, staticDHCPd will provide full DHCP service on that port, including IP assignment.

The `pxe` parameter in `conf.py:loadDHCPPacket()` will be set to `True` when a packet is received on this port, meaning you can test it to engage special handling logic. You may need to make use of functions like `packet.isDHCPDiscoverPacket()`, `packet.isDHCPRequestPacket()`, and `packet.isDHCPInformPacket()` to decide when to strip options when working on this port (chances are, in most cases, the client will have been assigned an IP over port 67 already, testable with `packet.getOption('ciaddr')`, and though it's highly unlikely, the device may complain if the response contains an IP offer; `packet.deleteOption('yiaddr')` takes care of this).


Unfortunately, staticDHCPd's maintainer does not have a functional PXE setup, so corrections and feedback are quite welcome. If you have questions or problems, please open an issue against the project and you should get help quickly.

## I want to customise the order of elements in the dashboard ##
And you should want to do this. Customisation is good.

Every dashboard element has an ordering-bias value; those with smaller values appear first. If you have logging at a debug level, you'll see this information printed when each one gets registered; every built-in element is configurable via `conf.py`, and you can set your own bias values in any modules you write.

## I want to use my own CSS/JS/favicon for the web interface ##
Sure, that's not a problem at all.

You can inject your own lines into `<head/>` by using a tiny bit of code in init():
```
def myHeaders(path, queryargs, mimetype, data, headers):
    return "<!-- anything you want to see appear in the <head/> section, as a valid XHTML fragment -->"
    
callbacks.webAddHeader(myHeaders)
```
You can use lambdas, too, if that's your thing. However you do it, this will add another line to the `<head/>` section, so you can add a link to your own stylesheet or just embed code directly. As with the other web-callbacks, the standard set of parameters are passed to your callback, so you can do different things depending on what was requested or what was received via POST; you just have to return a string or `None`, which suppresses output.

Anything you add, like a CSS class or JavaScript function, should be prefixed with a leading underscore, where possible, to avoid potential future conflicts with staticDHCPd's core code.

If you want to replace the CSS or favicon completely, you'll find their definitions in  `web.resources` and handlers in `web.methods`. Just implement your own equivalent method, then, in `init()` in `conf.py`, do something similar to the following:
```
callbacks.webRemoveMethod('/css')
callbacks.webAddMethod('/css', _YOUR_METHOD_HERE_, cacheable=True)
```
Replacing the favicon is pretty much identical. Replacing the JavaScript is discouraged, but also roughly the same; extend that, rather than replace it.


---

# Platform-specific questions #
## On Ubuntu, I get these "non-fatal select() error" messages in my logs at startup. Why? ##
Actually, we're not quite sure why, either. It seems as though Ubuntu's default configuration hits the process, when it starts, with a signal that generates an interrupt, which wakes the select() operations prematurely and causes them to throw an error because no handlers were invoked. No handlers were invoked because the nature of the interrupt is unknown, so to ensure normal operation, the error is semi-silently discarded and select() is invoked again, which is what would normally happen after each wakeup event. No requests can possibly be lost as a result of this error, so it's completely benign.

That said, if you see this message appear _after_ the initial startup, then you should start investigating the cause.

Further information:
> This is actually more of a Python issue than an Ubuntu issue (it would have been fixed if it were reasonably easy): Python's select() receives SIGINT, as it should, but there's no clear way to actually handle the signal gracefully -- although handling it properly would require knowledge of why it's actually being sent.


---

# Troubleshooting #
## How do I quickly identify issues and peek into the state of the system? ##
Starting with 1.5.2, a simple `writeLog("Hello!")` function has been exposed to the context of `conf.py:loadDHCPPacket()`, allowing you to write anything you want to the web interface.

With 2.0.0 and newer, while `writeLog()` is still supported, `logger` is a Python-logging object, supporting the methods `debug()`, `info()`, `warn()`, `error()`, and `critical()`, which you should use instead, like `logger.info("I'm operational now!"). `writeLog()` is now an alias for its `warn()` method.


---

# Unsupported features #
  * [RFC 3011](http://www.rfc-archive.org/getrfc.php?rfc=3011) (Subnet selection)
    * This feature is not required in a purely static environment.
  * [RFC 3004](http://www.rfc-archive.org/getrfc.php?rfc=3004) (User class)
    * staticDHCPd requires that each client be known ahead of time, precuding any need for thenotion of dynamic assignment from pools based on clases.
  * [RFC 3118](http://www.rfc-archive.org/getrfc.php?rfc=3118) (DHCP Authentication)
    * This feature is not supported because of the large number of clients that ignore the option.
    * It is also unnecessary in any environment in which staticDHCPd should be used: if administrators do not have absolute control of their network, staticDHCPd is useless.
  * [RFC 3203](http://www.rfc-archive.org/getrfc.php?rfc=3203) (FORCERENEW)
    * This feature explicitly relies on RFC 3118
    * It also poses problems related to authority and shouldn't be necessary in an all-static environment.
    * It will be implemented if anyone makes a solid case for its inclusion, though.
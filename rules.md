# Deprecated #
This page will be removed when 2.0.0 is released.

Its content has been [moved](http://static.uguu.ca/projects/staticDHCPd/doc/examples/index.html).

This article explains how to write custom DHCP-option-handling routines for staticDHCPd. This is not a feature most users will need to explore, so if you can't imagine why you would need to do it, you don't need to read on.


---

# Background #
No prior programming experience is required to use this feature of staticDHCPd, but having some would be a definite asset.


---

# Foreword #
Python is whitespace-sensitive. All code to be added to the `loadDHCPPacket()` function must be indented such that the outermost scope-line aligns with the comments and default `return True`. Failure to do so will result in a syntax error that will prevent staticDHCPd from starting. If this doesn't make sense, find a "Hello, world!" Python script and do some hacking.

As for DHCP options, don't worry about whether the client is configured to handle them or not; when the response is sent, all options the client did not ask to receive will be omitted, to ensure that nothing gets confused. Exception: in the (non-standard) event that a client does not provide a list in option 55, all set options will be returned.


---

# Formats and convenience functions #
With a few exceptions for special RFC requirements, all DHCP options configured by staticDHCPd are encoded using one of the following formats:
  * `bool`: A single bit, assignable using `packet.setOption(x, [int(True)])`
  * `byte`: A single byte, assignable using `packet.setOption(x, [127])`
  * `byte+`: A series of one or more bytes, assignable using `packet.setOption(x, [127, 255, 100, 2])`
  * `char`: A single character, assignable using `packet.setOption(x, [ord('c')])`
  * `char+`: A series of one or more characters, assignable using `packet.setOption(x, strToList('hello'))`
  * `ipv4`: A single IPv4 address, assignable using `packet.setOption(x, ipToList('127.0.0.1'))`
  * `ipv4+`: A series of one or more IPv4 addresses, assignable using `packet.setOption(x, ipsToList('127.0.0.1,192.168.1.1'))`
  * `ipv4*`: A series of zero or more IPv4 addresses, assignable using `packet.setOption(x, [])`
  * `16-bits`: A single 16-bit value, assignable using `packet.setOption(x, intToList(65535))`
  * `16-bits+`: A series of 16-bit values, assignable using `packet.setOption(x, intsToList((65535, 2, 10)))`
  * `32-bits`: A single 32-bit value, assignable using `packet.setOption(x, longToList(1000000))`
  * `32-bits+`: A series of 32-bit values, assignable using `packet.setOption(x, longsToList((65535, 2, 1000000)))`
  * `string`: A series of zero or more characters, assignable using `packet.setOption(x, strToList('hello'))`

Note: All integers may be specified using hex notation, `0xF0`, or in binary, `int('00010111', 2)`.

Note: If you need [RFC1035](http://www.rfc-archive.org/getrfc.php?rfc=1035) DNS formatting for a custom option, you can use `rfc1035_plus('www.google.ca,www.example.org').getValue()` to compute the byte-list. The RFC-specific handlers do this behind the scenes, though.


---

# Examples #
## Tell every client to use `127.0.0.1` and `192.168.0.1` as DNS servers ##
This example is not representative of the best way to do things; DNS servers may, and should, be set in the [database](database.md).

```
packet.setOption('domain_name_servers', ipsToList('127.0.0.1,192.168.0.1'))
```

This snippet sets the option associated with the symbolic, human-readable name `domain_name_servers` to `[127,0,0,1,192,168,0,1]`, eight bytes that constitute the two IP addresses. For efficiency's sake, you would probably want to write this as follows, although it is less readable:

```
packet.setOption('domain_name_servers', [127,0,0,1,192,168,0,1])
```

For a full list of all human-readable option-name values, see `pydhcplib/dhcp_constants.py:DhcpFields` and `pydhcplib/dhcp_constants.py:DhcpOptions`. Alternatively, if you know the number of the option you want to set, you can just provide it as an unquoted value.

## Tell all clients with an IP address ending in a multiple of 3 to use `192.168.1.254` as a default gateway ##
```
if client_ip[3] % 3 == 0:
    packet.setOption('router', ipToList('192.168.1.254'))
```

Here, the modulus-by-3 of the last octet (zero-based array) of `client_ip`, the IP address assigned to the client, is checked to see if it is zero. If so, the `router` option is set to `[192,168,1,254]`. Note that the function used this time is `ipToList()`. This is because only one IP needs to be translated; it's just more efficient this way. Of course, it would be more efficient still to use the same alternative format as before:

```
if client_ip[3] % 3 == 0:
    packet.setOption('router', [192,168,1,254]))
```

## Set T1 to `60` seconds ##
```
packet.setOption('renewal_time_value', longToList(60))
```

In this case, `longToList()` was used to translate the value `60` into `[0,0,0,60]`, `60` as a four-byte value. It's probably best to always use this function to avoid an endian mishap.

## Set the client's domain name to `example.com` if the request was relayed, but refuse to respond if it was relayed from `10.0.0.1` ##
```
if relay_ip:
    if relay_ip == (10,0,0,1):
        return False
    packet.setOption('domain_name', strToList('example.com'))
```

Here, `relay_ip`, `giaddr`, is checked to see if it was set (always do this before working with `relay_ip`, to avoid an error), indicating that this request was relayed. The IP of the relay server is then checked and, if it matches, `domain_name` is set to `example.com` using the `strToStrList()` convenience function.

## Prevent clients in all `192.168.0.0/24` subnets from having a default gateway ##
```
if subnet == '192.168.0.0/24':
    packet.deleteOption('router')
```

`subnet` was checked to see if it matched `192.168.0.0/24`; `serial` was ignored. These values come directly from the database -- staticDHCPd doesn't infer subnets based on IP and mask values.

## Check to see if option `3`, `router`, was requested, and fail if it was not set ##
```
if packet.isRequestedOption('router') and not packet.getOption('router'):
    return False
```

This one's a bit complicated behind the scnees:
  * The requested options, available through `packet.getRequestedOptions()` is a value that is `None` if the client did not provide option `55`.
    * In the event that it has not been set, it is assumed that the client asked for every option.
  * If option `3` is set, then the value currently associated with `router` is checked: if not set, it will be `None`; if set, it will be a list of four ints, 0 <= x <= 255, though the values associated with other options will vary.

If either of these conditions fail, the client's request is ignored or NAKed, depending on its nature.

## Refuse relays without option `82`, `relay_agent`, with an agent-ID of `[1, 2, 3]` ##
```
if relay_ip:
    relay_agent = packet.getOption('relay_agent')
    if relay_agent and not rfc3046_decode(relay_agent)[1] == [1, 2, 3]:
        return False
```

This allows any non-relayed requests to pass through. Any relayed requests missing option 82 will be allowed (more on this below); any instances of option 82 with an invalid agent-ID (sub-option 1) will be ignored. Any instances of option 82 missing sub-option 1 will generate an error (described in the next example).

Even relay agents configured to set option 82 will omit it if the resulting DHCP packet would be too large. For this reason, it's important to limit the relay IPs allowed in the config settings.

## Do something stupid to generate an error for testing purposes ##
```
if not packet.setOption('router', [192])):
    raise Exception("192 is not a valid IP")
```

The reason why this fails should be self-explanatory. What's important is noting that raising any sort of exception in this function prevents the DHCP response from being sent, but it will help to debug problems by printing or e-mailing a thorough description of the problem that occurred.

Note that if `packet.setOption()` returns `False`, which is what was tested, it's because the option was not set due to a format error. It is safe to ignore these problems, but it will lead to a lot of confusion, so it's always a good idea to use an `if`-statement like this.


---

# Additional information #
There are a few class definitions imported from pydhcplib.type\_rfc that can be used to set the complex options with which they are associated.

## [RFC2610](http://www.rfc-archive.org/getrfc.php?rfc=2610) ##
Set option 78 with the following pattern:
```
packet.setOption('directory_agent', rfc2610_78('192.168.1.1,192.168.1.2'))
```
There are no limits on the number of comma-delimited values you may specify.

Set option 79 with `packet.setOption('service_scope', rfc2610_79(u'slp-scope-string'))`, where `slp-scope-string` is the scope you want to set.

## [RFC3361](http://www.rfc-archive.org/getrfc.php?rfc=3361) ##
Set option 120 with either of the following patterns:
```
packet.setOption('sip_servers', rfc3361_120('example.org,uguu.ca'))
```
```
packet.setOption('sip_servers', rfc3361_120('192.168.1.1'))
```
There are no limits on the number of comma-delimited values you may specify. The only restriction is that either names xor IPs may be used, never both.

## [RFC3397](http://www.rfc-archive.org/getrfc.php?rfc=3397) ##
Set option 119 with the following pattern:
```
packet.setOption('domain_search', rfc3397_119('example.org,uguu.ca'))
```
There are no limits on the number of comma-delimited values you may specify.

## [RFC3925](http://www.rfc-archive.org/getrfc.php?rfc=3925) ##
Set option 124 with the following pattern:
```
packet.setOption('vendor_class', rfc3925_124([(0x00000001, strToList('hello'))]))
```

Set option 125 with the following pattern:
```
packet.setOption('vendor_specific', rfc3925_125([(0x00000001, [(45, strToList('hello'))])]))
```

## [RFC4174](http://www.rfc-archive.org/getrfc.php?rfc=4174) ##
Set option 83 with the following pattern:
```
isns_functions = int('0000000000000111', 2)
dd_access = int('0000000000111111', 2)
admin_flags = int('0000000000001111', 2)
isns_security = int('00000000000000000000000001111111', 2)

packet.setOption('internet_storage_name_service', rfc4174_83(
 isns_functions, dd_access, admin_flags, isns_security,
 '192.168.1.1,192.168.1.2,192.168.1.3'
))
```
There are no limits on the number of comma-delimited values you may specify, but you may require at least two, depending on the rest of your configuration.

## [RFC4280](http://www.rfc-archive.org/getrfc.php?rfc=4280) ##
Set option 88 with the following pattern:
```
packet.setOption('bcmcs_domain_list', rfc4280_88('example.org,uguu.ca'))
```
There are no limits on the number of comma-delimited values you may specify.

Set option 89 as you would set any other `ipv4+` value.

## [RFC5223](http://www.rfc-archive.org/getrfc.php?rfc=5223) ##
Set option 137 with the following pattern:
```
packet.setOption('v4_lost', rfc5223_137('example.org,uguu.ca'))
```
There are no limits on the number of comma-delimited values you may specify.

## [RFC5678](http://www.rfc-archive.org/getrfc.php?rfc=5678) ##
Set option 139 with the following pattern:
```
packet.setOption('ipv4_mos', rfc5678_139(
 (1, '127.0.0.1,192.168.1.1'),
 (2, '10.0.0.1'),
))
```
There are no limits on the number of comma-delimited values you may specify.

Set option 140 with the following pattern:
```
packet.setOption('fqdn_mos', rfc5678_140(
 (1, 'example.org,uguu.ca'),
 (2, 'example.ca,google.com'),
))
```
There are no limits on the number of comma-delimited values you may specify.
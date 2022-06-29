# Purpose
## Preamble
Firewalls, by their nature, are very complex devices. FortiGates are arguably the most complex firewalls of all, because they can perform so many functions beyond just the traditional NGFW capabilities, include SD-WAN, server load-balancing, Wireless LAN Controller, Switch Controller, DNS server, etc. This complexity posses major challenges for firewall administrators, because mistakes in configuration can introduce serious security risks. The more complex the firewall configuration becomes, the greater the risk of mistakes becomes. An essential strategy to help mitigate the complexity of the firewall configuration is to use consistent, semantically-meaningful naming conventions. Such naming conventions will greatly improve readability, thus reducing the chances for human-error, especially when multiple administrators work on the same firewall configurations. Even when a single administrator manages a firewall, someday that administrator will move on and administration duties will pass on to a new administrator. RTNG (Remember the Next-Guy/Gal) is an important motto to live by.

## Semantic Meaningfulness and Object-Oriented Design
FortiOS is object-oriented, by design. Whereas some other firewall systems allow the administrator to choose between literals or objects, FortiOS requires the use of objects in almost all cases. This design choice by Fortinet forces FortiGate administrators to use objects for almost all of the configuration, which is a good thing. Objects are much more semantically meaningful than literalls, as objects can be named according to their use-case, which enhances human readability. There is a reason the world adopted the use of DNS. We humans communicate much more effectively using names rather than numbers. Named objects serve a similar purpose.

When an administrator is used to working in literals, they will sometimes try to circumvent the object-oriented nature of FortiOS by naming their objects by their literal value. Imagine an IP address object whose value is 192.168.1.0/24. If you name this object `192.168.1.0_24`, then you've added no semantic value to the object, which does nothing to aid in human-readability. Instead, if you name that object `users_subnet`, now administrators know that this subnet is where users reside. Some administrators will claim that they want to "see" the actual IP address value of their objects, but every object in the FortiOS GUI can be hovered over to display its content, which will show the IP address. Furthermore, the various search functions within the FortiOS GUI allow you to search by both object names and object values, so it is easy to find objects when you know their values rather than their names.

Objects are also much more flexible and dynamic. For example: If an administrator could choose to use literal values for an IP address that is used in a static route, a firewall policy, and an IPsec Security Association (aka "Phase-2 Selector"), any changes to that IP address would have to be done in three places. But if an object is used instead of a literal, the administrator would only need to change the IP address value of that object, and the change will be reflected everywhere else that object is used in the configuration.

There are a few places where FortiOS allows administrators to choose between literals vs objects. These include static routes and IPsec Security Association (aka "Phase-2 Selector"). Use of literals here should be avoided, for the above stated reasons.

# Guiding Principles
--------------------

## String Length Limitations
FortiOS has some limitations regarding string lengths of object names, so try to use shorter names or abbreviations, where possible, with the comment fields being used for additional elaborations where needed. This could pose challanges in some scenarios. For example, in many of the conventions in this document, I recommend to use the FQDN in the object name, but some FQDNs might be too long. In these cases, you should use the just the hostname in the object name, and then the FQDN can be moved to the comment field.

## Case Sensitivity
ForitOS runs on Linux, and Linux is case sensitive, so do not mix upper and lower case. The preference should be for all object names to be lower case, except for objects that are by default all upper case, such as service objects, so new/custom service objects should also be all upper case. Having consistent case sensitivty will save you headaches, especially when working at the CLI.

## Avoiding White Space
White space can create difficulties for parsers for many automation, documentation, and logging systems, so spaces should never be used in object names, and instead dashes should be used to separate words. This way, you'll never have to bother with the various complexities around escaping white spaces.

## Avoiding Special Character
Special characters can also create difficulties for parsers for many systems so the only special characters that should be used in object names are dashes and underscores, and never characters like ``! @ # $ % ^ & * ( ) ~ + =`` etc., even when FortiOS itself allows it.

## Mutability
FortiOS uses UUIDs for most object types, which allows their object names to easily be changed. A few object type, however, cannot be changed once they are created. These can create challanges in scenarios where semantic naming changes. Imagine a scenario where a company called ACME gets aquired by another company, WidgCo, and the WidgCo CEO wants all ACME assets to be renamed WidgCo. For most object types in FortiOS, this will be easy to accomplish. But for a few object types that can't be renamed, they would have to be deleted and re-created, which would cause downtime.

Therefore, unchangable names (e.g. interfaces) should be named in such a way that they never need to be changed, so they should be named according to their objective essence (which cannot change) rather than their use case (which can change), and the alias field (which is changeable) can then be used to describe its current use case. For names which can easily be changed (e.g. address objects) they should be named according to their use case or their FQDN.

## Object Type Distinctions
There can be many different types of objects that share the same use-case or same FQDN, and so would share the same name, based on the above principles, but sharing the same name may be impossible in FortiOS in some cases, and even when possible, will still be confusing for humans. Therefore, each name should be suffixed by its object type, separated by an underscore.

> E.g.
	- Imagine a webserver whose FQDN is `web01.example.local`, and who has a private IP address of `192.168.1.11`, and a public IP address of `4.4.4.4`, with a 1-to-1 VIP object performing NAT.
	- In this case, the private IP address object would be named `web01.example.local_priv` (for "private IP"), whereas the VIP object would be named `web01.example.local_vip`.
	- Additionally, you might need a MAC address object for the same server, which would be named `web01.example.local_mac`.
	- If you are doing Deep SSL Inspection to proect this web server, its SSL Inspection Profile would be named `web01.example.local_ssl`
	- If you need a custome WAF Profile to further proect this web server, its WAF Inspection Profile would be named `web01.example.local_waf`

- Numbering: Multiple instances of the same object type for the same use case should be suffixed with a number that increments from 1. If you expect to never need more than 9 of an object, this can be a 1-digit number, but if you may need up to 99 of an object, it should be a 2-digit number, etc. 
> E.g. If `web01.example.local` needed two different private IP addresses objects (such as if it were dual-homed), these would be named `web01.example.local_priv01` and `web01.example.local_priv02`

- Object type groups: Group names should also be suffixed with their type, but will be of the plural form, indicating a collection of more than one of that object.
> E.g. A group of `_priv` objects would be named `_privs`. A group of `_vip` objects would be named `_vips`. A group of `_mac` objects would be named `_macs`. Etc.

In the below specifications, I've tried to stick to the above principles as much as possible, and everything else is just arbitrary conventions that have served me well in a wide range of scenarios. Your mileage may vary.

-------------------------

# Conventions 
----------------------------

## Interfaces
-------------------

### Loopback Interface:
- Name: loN, where "lo" means "loopback" (like in Cisco terminology) and N is the number of the loopback interface, which increments from 1
- Alias: <usecase>_loopback, where use case is what this loopback will be used for.
> E.g. if you just had a single loop back for all management function, then the alias for `lo1` would be `mgmt_loopback`, but if you were going to have different loopbacks for different management functions, you might have something like `lo1:ntp_loopback`, `lo2:dns_loopback`, `lo3:bgp_loopback`, etc.

### LACP 802.3ad Aggregate Interface 
- Name: poN, where "po" means "port channel" (like in Cisco terminology) and N is the number of the port channel, which increments from 1
- Alias: <usecase>_trunk, where use case is what this trunk will be used for, and `_trunk` is Fortinet's teminology for a LACP Aggregate
> E.g. `po1` might have an alias of `core_trunk`

### Hardware Switch Interface:
- Name: swN, where "sw" means "switch" and N is the number of the port channel, which increments from 1
- Alias: `<usecase>_switch`, where use case is what this switch will be used for
> E.g. `sw1` might have an alias of `lan_switch`

> Tip: Consider disabling all unused physical ports on the FortiGate (for both security and stability reasons). Put all of them into a special switch object named sw0 with alias `disabled_switch`, and then just disable that whole switch object. This also visually cleans up the interface table in the GUI, especially for FortiGate models with lots of physical interfaces.

### Software switch:
- Never use these, because they can't do hardware acceleration.
- The only time to use a software switch is if you have a FortiGate that can't support hardware switches, in which case, you should try to just use a LACP Aggregate to a physical switch to do your switching. If you must use a software switch, just follow the same naming convention for a hardware switch.

### Physical interface:
- Physical interfaces cannot be renamed, but also they should never be used in your configuration, because they are completely inflexible and will create reference hell for you if you ever need to add/change the underlying physical interfaces. 
- Instead, always create either a LACP Aggregate or a Hardware Switch Interface to add your physical interfaces to, even if you only need a single interface today.

### NPU VDOM Link:
- Name: these are hard-coded as npuN-vlinkM, where N is the number of NPUs, incrementing from 0, and M is the vlink number (either 0 or 1).
- Alias: These cannot have aliases
> N.B. we will not be using the NPU VDOM Links themselves, but instead will be using 802.1q VLAN sub-interfaces on the VDOM links, so that we can have up to 4096 hardware-accelerated VDOM links per NPUs

### Software VDOM Links:
- Do not use these, because they cannot be hardware accelerated

### VLAN Interfaces:
- Name: ``<base-interface>``.VVVV, where ``<base-interface>`` is the base interface of which this VLAN interface is sub-interface, and VVVV is the 4-digit VLAN ID according to 802.1q
- Alias: ``<use-case>_vlan``
	> E.g.
	> - `po1.0002` might have an alias for `inf_vlan` (where inf means infrastructure, such as servers and management IPs of network infrastructure)
	> - `po1.0003` might have an alias of `user_vlan` (where corporate users will reside)
	> - `po1.0004` might have an alias for `voip_vlan` (where Voice-over-IP handset will reside)
	> - `po1.0005` might have an alias of `guest_vlan` (where guest Wi-Fi users will reside)
	> - `po1.0006` might have an alias of `iot_vlan` (where Internet of Things devices will reside, such as printers, cameras, building automation sensors, and other low-security headless devices)
	> - `po1.0007` might have an alias of `dmz_vlan` (where internet-accessible servers will be isolated from non-internet accessible servers)
	> - Alternatively, if you're using a hardware switch instead of LACP Aggregate, `sw1.0002` might have an alias for `inf_vlan`, etc, as above
	> - `npu0-vlink0.0002` might have an alias of `<vdom01>_vlink` while `npu0-vlink1.0002` might have an alias of `<vdom02>_vlink` (thus linking vdom01 with vdom02)

### SSID Interfaces:
- Name: `ssidN`, where `N` is the number of SSID interface, which increments from 1
- Alias: ``<use-case>_ssid``, where use case is what this SSID will be used for
> E.g. `ssid1` might have an alias of `corp_ssid`, and `ssid2` might have an alias of `guest_ssid`

### IPsec VPN Phase-1 Interfaces
- Name: ``<use-case>_tun1``, where "tun" means "tunnel" and number is the number of the tunnel that increments from 1
- Alias: As of FortiOS 6.4, IPsec VPN Phase-1 Interfaces cannot have aliases, which is why the use-case is in the name instead, so the use-case name must be relatively short/abbreviated

### Zones:
- Name: `<use-case>_zone`
- Alias: Zones cannot have aliases as of FortiOS 6.4, but you can use the comments field for additional elaboration if needed
	> E.g. 
	> - `inside_zone` might be the name of for the zone that will collect all of the "inside" interfaces protected behind this FortiGate
	> - `wan_zone` might be the name for the (SD-WAN) zone that will collect all of the "wan" interfaces connected to this FortiGate
	> - `tun_zone` might be the name for the (SD-WAN) zone that will collect all of the "tunnel" IPsec interfaces connected to this FortiGate
- N.B. All interfaces that will have IPv4/6 firewall policies applied must be added to a zone, such that all policies will be zone-based and not interface-based, which will greatly enhance policy consolidation, and make future interface changes/additions much easier and quicker

----------------------------

## Address Objects

### Device/MAC Objects:
- Name: `<fqdn>_mac`
- Comments can be used for additional elaboration

### Subnet Objects:
- For a /32 in the RFC 1918 space
	- Name: `<fqdn>_priv`, or `<fqnd>_privNN` if more than one `_priv` will be needed for that same FQDN
- For a /32 outside of the RFC 1918 space
	- Name: `<fqdn>_pub` or `<fqnd>_NN` if more than one `_priv` will be needed for that same FQDN
- For prefix shorter than /32
	- Name: `<use-case>_subnet`, or `<use-case>_subnetNN` if more than one `_subnet` will be needed for that same use-case
- Comments can be used for additional elaboration

### FQDN Objects:
- Name: `<fqdn>_fqdn`
- Comments can be used for additional elaboration

### GeoIP Objects:
- Name: `<country-abbreviation>_geo`

### IP Range Objects:
- Name: `<use-case>_range`
- Comments can be used for additional elaboration
- N.B. Do not use a `_range` object when a `_subnet` object could be used instead

### Custom ISDB Objects:
- Name: `<use-case>_isdb`

### VIP Objects of type static-nat:
- For a 1-to-1 Destination NAT:
	- Name: `<fqdn>_vip`
	> E.g. `web01.example.com_vip` would forward all ports to the server whose FQDN is `web01.example.com`
- For a N-to-N Destination NAT range:
	- Name: `<use-case>_vip`
- For a 1-to-M Destination PAT:
	- Name: `<fqdn>-<service>_vip` 
	> E.g. `web01.example.com-ssh_vip` for port 22 and `web01.example.com-http_vip` for port 80

### Address Group Objects
- Name: `<fqdn | use-case>_<object-type>s`, where the "s" indicates plurality, hence a group of multiple objects
	> E.g. 
	> - `company-asn_subnets` might contain all of the subnets for the company's ASN such as company-asn_subnet01 and company-asn_subnet02
	> - `ban_geos` might include all of the Geo IP objects that you want to block, such china_geo and iran_geo
	> - `sslvpn_geos` might include all of the Geo IP object for which you want to allow SSL VPN connections
	> - `web01.example.com_vips` might contain both web01.example.com-ssh_vip and web01.example.com-http_vip
	> - `hairpin_vips` might contain all of the VIP objects for which you want to allow hairpin NAT
	> - `collab_isdbs` might contain the ISDB objects for Zoom, Teams, WebEx, Slack, etc
- N.B. Groups should always be preferred over individual objects for creating policies and routes, (even if a group only contains one object member initially) as this makes it easier to facilitate changes/additions in the future 

--------------------

## Special Address Objects

### VIP Objects of type server-load-balance (aka Virtual Servers in the GUI):
- Name: `<cluster-name>:<port>_vlb`, where "vlb" means virtual load balancer
> E.g. 
> - You might have `web-clus01:80_vlb` to load-balance HTTP traffic to `web01.example.com` and `web02.example.com`
> - You might have `web-clus01:443_vlb` to load-balance HTTPS traffic to `web01.example.com` and `web02.example.com`

### IP Pools (aka Source NAT objects)
- For an overload pool:
	- Name: `<use-case>_overload`
	> E.g.
	> - `user_overload` might be the overload pool for the corporate users
	> - `guest_overload` might be the overload pool for guest Wi-Fi users 
- For a 1-to-1 pool:
	- Name: `<fqdn | use-case>_1-to-1`
	> E.g.
	> - If you had two web servers that you wanted to assign two particular public IP addresses, you might have `web01.example.com_1-to-1` and `web02.example.com_1-to-1`
	> - If you were assigning a contiguous range of public IP addresses to all your servers, you might have `servers_1-to-1`

- As of FortiOS 6.4, neither VIP Objects nor IP Pools can be added to groups

---------------------------------

## Service Objects

### Single Service Object
- Name: `<SERVICE-NAME>` in all caps, because all of the default service objects are in all caps
	- Special case: alternate ports for defauly service should be prefixed with `-ALTNN`, where `NN` is a two-digit number incrementing from 01
> E.g. `HTTPS_ALT01` could be TCP/8443, and `HTTPS-ALT02` could be TCP 9443, etc

### Service Group Objects:
- Name: `<usecase>_services`, lower case
> E.g.
> - `web_services` might contain HTTP, HTTPS, and ALL_ICMP
> - `webmin_services` might contain SSH, HTTP, HTTPS, and ALL_ICMP (where "webmin" is short for web administration)
- N.B. Service groups should be preferred in policies to facilitate easier changes/additions
- N.B. `ALL_ICMP` should generally be added to most service groups to allow for easier troubleshooting and better functionality of services like PMTUD, although security considerations in some cases might warrant restrictions on some ICMP types and codes

------------------------------

## Security Profiles

### Antivirus:
- Name: `<use-case>_av`
> E.g. You might have both `flow_av` and `proxy_av`

### Web Filter:
- Name: `<use-case>_wf`
> E.g. You might have both `corp_wf` and `guest_wf`

### DNS Filter:
- Name: `<use-case>_df`
> E.g. You might have both `corp_df` and `guest_df`

### Data Leak Prevention (DLP)
- Name: `<use-case>_dlp`
> E.g. You might have both `pci_dlp` and `pii_dlp`

### Application Control:
- Name: `<use-case>_ac`
> E.g. You might have both `corp_ac` and `guest_ac`

### Intrusion Prevention System (IPS):
- Name: `<filters>_ips`
> E.g. You might have both `client_ips` (to protect clients that connect outbound to internet) and `win-server-iis_ips` (to protect a Windows IIS server) as well as `lin-server-appache_ips` (to protect a Linux Apache server), etc.
- N.B. Using as specific of an IPS filter as possible for a given traffic flow will greatly improve the efficiency of the IPS Engine's processing

### WAF:
- Name: `<use-case | fqdn>_waf`
> E.g. 
> - You might have `iis_waf` and `apache_waf,` with different configurations for general protection of IIS and Apache, respectively
> - You might have `web01.example.com_waf` and `web02.example.com_waf,` with different configurations specific to the two different web servers that you need to protect

### SSL Inspection:
- Name: `<use-case | fqdn>_ssl`
> E.g. 
> - For basic outbound certificate inspection, you might have `client-cert_ssl`
> - For deep SSL inspection outbound, you might have `client-deep_ssl`
> - For deep SSL inspection inbound protecting a single server (or multiple servers with SNI in FortiOS 7.0 or higher), you might have `web01.example.com_ssl`

### Protocol Options:
- Name: `<use-case>_prot`
> E.g. you might have both `default_prot` and `comfort-clients_prot`

-------------------------------

## Firewall Policies

### For an allow policy:
- Name: `[this]>[that]`, where `[this]` is the name of the source object and `[that]` is the name of the destination object.
- If you have multiple policies with the same or similar destinations, but different services, suffix them with `_[service-group]`
> E.g.
> - `lan_subnets>wan` would allow all subnets on the LAN to get out to the internet
> - `corp_subnets>dc01.example.com_adds` would allow the corporate subnets access Domain Controller on Active Directory Directory Services ports
> - `guest_subnets>dc01.example.com_dns` would allow guest subnets access Domain Controller only for DNS
### For a deny policy:
- Name: `[this]>[that]_deny`, where `[this]` is the name of the source and `[that]` is the name of the destination
> E.g. `lan_subnets>ban-geos_deny`

-------------------------------

## DDoS Policies

### For an monitor-only DDoS policy:
- Name: `[this]>[that]_monitor`, where `[this]` is the name of the source object and `[that]` is the name of the destination object.
> E.g. `lan_subnets>wan_monitor` would monitor for any DDoS attacks originating from your LAN subnets out to the internet

### For a block DDoS policy:
- Name: `[this]>[that]_block`, where `[this]` is the name of the source object and `[that]` is the name of the destination object.
> E.g. `wan>pub-subnets_block` would block any DDoS attacks originating from the internet targeting your public subnets
N.B. Because DDoS policies cannot be applied to SD-WAN zones, if you have multiple WAN interfaces that need DDoS protections, you'll need multiple DDoS policies.
> E.g. `wan1>pub-subnets_block` and `wan2>pub-subnets_block`

-------------------------------

## Traffic Shaping

### Traffic Shaping Profiles
- Name: `<use-case>_shaping-prof`
- N.B. Class ID names should be equal to the FortiGuard Application Categories, one Class ID per Category

### Traffic Shaping Policies
- Name: `<class-id>` (which should be the same as `<app-category>`) for application category Traffic Shaping Policies
- Name: `<use-case>` for special use-case Traffic Shaping Policies

### Traffic Shapers
- N.B. I almost never use Traffic Shapers anymore after Traffic Shaping Profiles were introduced in FOS 6.4.
- This is because Traffic Shaping Profiles are much more dynamic, since they can use bandwidth percentages, which are crucial in SD-WAN configurations, where wan1 and wan2 often have different upload/download bandwidth values. Using static Traffic Shapers, you would need different Shapers for wan1 and wan2, and maybe even for wan1-upload, wan1-download, wan2-upload, wan2-download if they have asymetric bandwidth. Using static Traffic Shapers in a such a scenario could exponentially increase the number of objects you have to create and manage, whereas a Traffic Shaping Profiles can handle all of that complexity in a single object.
- Furthermore, imagine a scenario where wan1 initially has a subscribbed bandwidth of 100x10, but then you have your ISP upgrade your subscribbed bandwindth to 200x20. If you used static Traffic Shapers, you would have to go through and edit all of your Traffic Shaper objects relevant to wan1 and double their max/guarntee values in order to actually realize your bandwidth upgrade. But if you used a Traffic Shaping Profile instead, the only change you would have to make would be to edit the Estimated Bandwidth values on the `wan1` interface object, which will allow the Traffic Shaping Profile to re-calculate the appropriate percentages.
- The only exception to this rule would be if you need to enforce a specific static bandwidth max/guarantee for a certain traffic flow. An example use-case for such a need would be an ISP who is using FortiGates as their CPE. Such an ISP might have a contract to deliver exactly 100 Mbps x 100 Mbps to their customer. In this case, a static Traffic Shaper is required.
- Name: `<use-case>_shaper`
	> E.g. `acme-subscribbed_shaper` would enforce the subscribbed bandwidth for ACME Corp, who is an ISP customer 

-------------------------------

## SD-WAN

### Performance SLAs
- As of FortiOS 7.0, dashes are not allowed in the name of SD-WAN Performance SLA objects, so if an FQDN contains dashes, you will have to use an underscore.
- Name: `<target-fqdn_[ping | http | dns]_sla>`
> E.g.
> - `update.fortiguard.net_ping_sla` would check ICMP peformance to update.fortiguard.net using Ping
> - `google.com_http_sla` would check web performance to google.com using HTTP
> - `cloudflare_dns` would check DNS performance to Cloudflare's DNS servers at 1.1.1.1 and 1.0.0.1

### SD-WAN Rules
- As of FortiOS 7.0, the `>` symbol is not allowed in the name of SD-WAN Rules, so we wont be able to follow exactly the same convention we used for Firewall Policies
- Name: `[this]_to_[that]`, where `[this]` is the name of the source and `[that]` is the name of the destination.
- E.g. `lan_to_google` might route all traffic from LAN to Google's cloud services

### SD-WAN Zones
- The default SD-WAN zone object is named "virtual-wan-link". That doesn't follow any of our conventions. 
- Name: `<use-case>_sdwz` (where "sdwz" is an abbreviation for "SD-WAN Zone")
> E.g. 
> - For your underlay WAN links, you might have `underlay_sdwz`, where "underlay" is the MEF terminology for the physical WAN link you get from your ISP)
> - For your overlay WAN links, you might have `overlay_sdwz`, where "overlay" is the MEF terminology for the IPsec tunnels you build on top of your underlays

-------------------------------

## User and Authentication Objects

### User Group:
- Name: `<group-name>_users`
### LDAP Server:
- Name: `<fqdn-of-server>_ldap`
### RADIUS Server:
- Name: `<fqdn-of-server>_radius`

-------------------------------

## WiFi Controller

### FortiAP Name
- Name: `hostname`
> E.g. You might have `ap01`, `ap02`, etc
- N.B. You can use the Description field to provide more description, like the physical location of the switch, such as "Building 2, Room 307, Rack 4, U27"

### FortiAP Group
- Name: `<use-case>_ap-grp`

FortiAP Profiles
- Name: `<fortiap-model>_ap-prof`
- If you need more than one AP profile for the same FortiAP models to support different use-cases (such as dual-band vs single-band operation), try `<fortiap-model-use-case_ap-prof>`

### SSIDs
- See entry for SSIDs under the Interfaces section of this document

### MPSK Group
- Name: `<use-case>_mpsk-grp`

### WiFi Maps
- Name: <address-floor_wifi-map>
> E.g. `123main-f0_wifi-map` would be the basement of the building at 123 Main, `123main-f1_wifi-map` would be the ground floor of the same building

### WIDS
- Name: `<use-case>_wids`

-------------------------------

## Switch Controller

### FortiSwitch Name
- Name: `hostname`
> E.g. You might have `core-sw01`, `core-sw02`, `dis-sw01`, `dis-sw02`, `acc-sw01`, `acc-sw02`, etc
- N.B. You can use the Description field to provide more description, like the physical location of the switch, such as "Building 2, Room 307, Rack 4, U27"

### FortiSwitch Profile
- Name: `use-case_switch-prof`

### FortiSwitch Trunks
- Name: `trNN`, where NN is a two-digit number that increments from 01
- N.B. You can use the Description field to provide addition details, such as the use-case

### LLDP Profiles
- Name: `<use-case>_lldp`
> E.g. You might have `voip_lldp` and `cam_lldp`

### QoS Policies
- Name: `<use-case>_qos`
> E.g. You might have `voip_qos` and `cam_qos`

### Security Policies (802.1X)
- Name: `<use-case>_d1x`

### Dynamic Port Policies (802.1X)
- Name: `<use-case>_dpp`

### NAC Policies
- Name: `<use-case>_nac`

-------------------------------

## DNS Database
- Name: `zone-fqdn_dns-database`
	> E.g. You might have `ad.acme.com_dns-database`, which a shadow secondary DNS server getting zone transfers for the ad.acme.com zone from dc01.ad.acme.com, for DNS resiliency

-------------------------------

## Automation Stitches

### Trigger
- Name: `<condition>_trigger`

### Action
- Name: `<use-case>_<action-type>`
> E.g. 
> - For a script that cycles POE to revive a down AP, you might have `poe-cycle-ap01_script`
> - For an email that alerts on loss of FortiAnalyzer connectivity, you might have `fgt-faz-conncection-lost_email`

### Stitch
- Name: `<use-case>_stitch`

### Link-Monitor
- Name: `<fqdn-of-target>_link-mon`

-------------------------------

## External Connectors

### IP Address Threat Feed
- Name: `<fqdn | use-case>_ip-list`

### Domain Name Threat Feed
- Name: `<fqdn | use-case>_url-list`

### Malware Hash Threat Feed
- Name: `<fqdn | use-case>_hash-list`

-------------------------------


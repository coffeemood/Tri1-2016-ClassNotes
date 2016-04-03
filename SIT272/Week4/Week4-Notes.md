# SIT272 - Week 4 - IPv4, IPv6 and Routers 

### IPv6 

#### Migration from v4 -> v6 

+ Three mitigation techniques
    * Dual Stack (Running both versions)
    * Tunneling (Encapsulate v6 inside v4)
    * Translation (NAT64)
#### Format 

The IPv6 address: 
    + 128 bits (4 hex)(radix 16)
    + Omit preceding zeroes 
    + Group running zeroes as `::`

| Address | Description | 
| --------| ----------- | 
| Unicast | same as v4 |
| Multicast | save as v4 |
| Anycast | Sent to multiple destinations, routed to nearest devices |
| Link-Local | Similar to v4. Configured locally to route LAN devices |
| Loopback | ::1 |
| Unique-local | Similar to private IPs |
| IPv4 embedded | Used to transition from v4 -> v6 | 
| Unspecified | :: | 

**There are no broadcast addresses in v6** 

#### Global Unicast Address

+ The address is divided into 3 parts
    * The first 48 - Global Routing Prefix (provided by ISP)
    * The next 16 - Subnetting 
    * The final 64 - Interface ID

#### Enabling v6

```
enable
configure terminal
ipv6 unicast-routing
interface fastethernet 0/1 
ipv6 address 2001:db8:acad:1::1/64
no shutdown 
```


#### IPv6 Dynamic IP config 

**Uses SLAAC (Stateless Address Autoconfiguration)**

- Provided are 3 `advertisement options`:
    + **SLAAC only**: Provide prefix, prefix length & default gateway
    + **SLAAC** & DHCPv6: Partial information, need to request the rest from DHCPv6
    + **DHCPv6 only**: Need to request DHCPv6 for everything

#### Link-local 

- Link local address is automatically assigned 
- `link-local` is usually used as the default gateway address
- Must have for device to communicate with others using v6
- Used by routers to **determine next hop** in routing tables

#### Multicast 

- Have the prefix of FF00::/8
- Two types:
    + Assigned
        * FF02::1 “all-nodes” (joined by all IPv6 devices) 
        * FF02::2 “all-routers” (joined by all IPv6 routers)
    + Solicitated
        * Address is combination of FF02:0:0:0:0:FF00::/104 and far right 24 bits of unicast address
        * Used by Neighbor Discovery (MAC)
        * Host joins solicitated address for every configured unicast address





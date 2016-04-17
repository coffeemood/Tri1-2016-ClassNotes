# SIT272 - Week 3 

## Physical Layer 

#### Media Types

There are three main types of media
    + Copper (UTP,TP,Coaxial,...)
    + Fiber Optic (Single Mode, Multi Mode,...)
    + Wireless (Access Points, NICs, ....)

![Medias](http://i.imgur.com/XeFsv4d.png)

![Comparison](http://i.imgur.com/ZdL7jAr.png)

| Name | IEEE | Speed | Desc | 
| ---- | ---- | ---- | :-----: |
| Wifi | 802.11 | Upto 7Gbs | Uses CSMA/CA |
| Bluetooth | 802.15 | Short distance, upto 3Mbps | 
| WiMax | 802.16 | Upto 1Gbs | Point-to-multipoint topology |

![wifi](http://i.imgur.com/b3xRTNs.png)

#### Key concepts 

+ Bandwidth: Capacity of physical media (100Mbs)
+ Througput: Actual transferring rate through the media, which depends on:
    * Amnt of traffic
    * Type of traffic
    * Latency

> + Physical stds guarantee that cables & connectors will work as expected
> + EMI & RFI
> + Crosstalk (Why we need to twist pairs)

## Data Link Layer

Two sublayers: 

+ Logical Link Control - `LLC`  
    * Identify withe network layer protocols (software)
    * Aid layer 3

+ Media Access Control - `MAC`
    * Identify with physical layer protocols (hardware)
    * Aid layer 1

![DL](http://i.imgur.com/e3ASDYL.png)

#### Media Access Control

> Continually searching for next hop's MAC address. Return next router's MAC if not found. 

- Utilise `CSMA/CD` (Carrier Sense Media Access w/ Collision Detection)
    + CSMA:
        * Listen to media
        * Transmit when clear
    + CD: 
        * Sends jam signal (backoff) when collision takes place
        * Retry
        * All other hosts can transmit when time expires

- MAC addresses: 
    + 48 bits 
    + First 24 - OUI
    + Last 24 - Vendor assigned 

> Similar to `CSMA/CA` used in wifi

#### Ethernet Frame 

> Ethernet Frame Model

![frame model](http://i.imgur.com/a4kOwxY.png)

| Preamble |  Used for synchronisation |
| Destination| 48bit destination MAC |
| Source | 48bit source MAC |
| Type | Identify upper layer protocol |
| Data | Typically the PDU |
| FCS | Frame Check Sequence |

## Ethernet 

Most popular LAN tech (802.2 & 802.3)

> High speed ethernet eliminates CSMA/CD -> We need to use switches

Unicast/Broadcast/Multicast: 

![unicast](http://i.imgur.com/wFcNmRq.png)
![broadcast](http://i.imgur.com/SPEms6y.png)
![multicast](http://i.imgur.com/wUufntL.png)

#### LAN Switches

+ Two forwarding techniques
    * Store-and-forward
    * Cut-Through
        - Forward after receiving first 6 bytes (after preamble)
        - Doesn't check FCS
        - Fast forward & fragment-free

+ MDIX: Automatically detect and configure devices

+ Two types of buffer: 
    * Port-based memory
    * Shared memory

> Switches are also available as Layer 3 devices which work with IPs, which eliminate the need for routers

#### ARP - Address Resolution Protocol

> Broadcast requesting all MAC address on the network, in return devices send unicast msg containing its IP 

![arp table](http://i.imgur.com/aE9G32x.png)

+ Key problem: 
    * Overheads (occupy resources)
    * Security (vulnerable to `mitm` attack)




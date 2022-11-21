# CitriX ADC Networking Overview

- Citrix ADC is an application switch that performs application-specific traffic analysis to intelligently distribute, optimize, and secure Layer 4 - Layer 7 (L4–L7) network traffic for web applications.

![Image](/img/basicnetworking1.png)

## Citrix ADC IP Address

To perform a basic setup, the following IP addresses are configured
- **Citrix ADC IP (NSIP) addresses**
- **Virtual IP (VIP) addresses**
- **Subnet IP (SNIP) addresses**
- **Mapped IP (MIP) addresses (legacy)**

## Key Notes

- As soon as we configured a SNIP or a MIP a direct route is created and cannot be deleted.
- All the Citrix ADC owned IP addresses can be removed apart from NSIP
- If SNIP exists, we can remove MIPs. 
- `rm ns ip <IP Address>` can be used to remove the citrix adc owned IP.

## Citrix ADC Ip Address (NSIP)

- The citrix adc ip (nsip) address is the primary ip address used for managing the system.
- The citrix adc ip addresses is:
  - required at initial device configuration to allow system access.
  - used for system to system communication
  - not removable.
- if the nsip is modified , restart the system , in order for the change to be applied.
- MPX initial NSIP : 192.168.100.1/16

## Virtual IP Address

- Virtual IP (VIP) addresses are used for client-to-Citrix ADC communication
  - A VIP address is an IP address associated with a virtual server
  - ARP can be disabled to facilitate migration
  - ICMP can be disabled to turn off ping if it is required 

## Subnet IP Address (SNIP)

- The subnet ip (snip) address functions as a proxy ip and snip is used by the citrix adc system for citrix adc -to- server communication.
- The SNIP addresses can;
  - be bound to vlans
  - be used to monitor the health of servers
  - provide management access
  - Subnet IP (SNIP) address -USNIP must be enabled (if we disable then we must have MIP)
  - With Use SNIP (USNIP) mode enabled, a snip is the source ip address of a packet sent from the citrix adc to the server. This mode is enabled by default.        

## Mapped IP (MIP) Address

- Mapped IP addresses (MIP) are used for server side connections.
  - It has similar to functionality to a snip.
- MIP addresses are deprecated and remain only to support legacy functionality. It is recommended that we use a SNIP instead.
- The MIP address should be available accross all subnets and should never be bound to a VLAN.
- MIPs are used when a SNIP is not available or Use SNIP (USNIP) mode is disabled.

## Use Subnet IP Mode (USNIP)

![Image](/img/usnip.png)

- When use SNIP (USNIP) mode is enabled:
  - a snip is the source ip address of a packet sent from the citrix adc to the server
  - a snip is the ip address that the server uses to access the citrix adc
- USNIP mode is enabled by default
  - if disabled , a mip must be defined.

## Use Source IP Mode (USIP) 

![Image](/img/usip.png)

- when use source ip (usip) mode is enabled :
  - the client ip is used as source ip to server
  - server gateway is set to citrix adc snip
  - monitors are still sourced from snip.

- by default the citrix adc uses a snip or mip to connect to the backend servers
- usip passes the actual client ip address to the server instead of a MIP/SNIP
- USIP is:
  - Not enabled by default
  - must have surge protection disabled for HTTP protcol
  - can be enabled globally or at service level
  - should only be used when required.

## Key Notes :

- Part of the citrix adc system suite of performance enhancements resolves around maintaining one connection to the client and multiplexing another to the server. This requires the citrix adc system to translate the client's IP address to either a MIP address or SNIP address. This behavior will not be desired in some situatons. In these cases, we can enable use source ip mode.The result is that the client's actual IP address is used to connect to the back-end server.
- we shuld consider a number of performance considerations before activating this feature:
  - Multiplexing can only be used for conenctions originating from the same client IP address. This means that significantly more sessions will be established between the citrix adc system and the server. This is inefficient for the Citrix ADC system and requires more overhead for the server.
  - surge protection is also unable to function in this environment
  - usip requires routing in the environment to direct all off the server responce traffic bound for the client ip address through the citrix adc system.
  - usip can be enabled on the globally or virtual server level.
  - usip v4 to v6 supported in load balancing configuration.
  - for http protocols this feature must be used with surge-protection OFF. For non-HTTP protocols, such as service type TCP, FTP and others this restrcition is not applicaple.

  ## Applications that Requires USIP

![Image](/img/usip2.png)

- Ensure L3 mode is enabled and the SNIP is set as the server's default gateway, as the response must pass back to the Citrix ADC.
- Rather than using the MIP/SNIP for the connection , use layer 3 mode to enable the citrix ADC to pass the client ip address to the backend server.

## Key Notes :

- Question : Why do we have layer 3 mode and why is it enabled by default ?
- In these situations, you should use USIP. Howewer since this mode limits other functionality on the citrix adc, it should only be used when absolutely required. If you only want to pass the client IP address to the application for web logging purpose, and the application is http-based, you souldnt use USIP mode. Instead you should use client ip header insertion.

## Client-IP HTTP Header Insertion 

- Client-IP HTTP header insertion is useful when a backend server needs to identify the client that originated a request.
- When the conenction is being proxied by the citrix adc system, it is available for http and https traffic types.
- using this instead of usip still allows the full proxy functionality and enables the use of multiplexing and surge protection.

## IP Set

- An IP Set is a set of IP addresses
- An IP Set can be bound to a net profile
- A net profile can be bound to load balancing or content switching virtual servers, services , services groups or monitors. A net profile has citrix adc owned ip address (SIPs and VIPs) that can be used as the source IP address. It can be a single ip address or a set of ip addresses.
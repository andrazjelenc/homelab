# Design
In this document there is a design of my home network. I currently have one Mikrotik router called Router and one Access Point called AP1. The AP1 acts only as an access point and it broadcasts two SSIDs one for home usage and one for guests.

My ISP provider uses VLAN 3999 for multicasts for IPTV. All interfaces of the ISP router are hybrid interfaces. Untagged traffic is used for normal traffic and tagged 3999 traffic for IPTV. So interface ether1 of my Router is also a part of VLANs. On ingress we keep tagged traffic VLAN 3999 and tag untagged traffic with VLAN 666. The TV BOX has to have access to the tagged traffic for IPTV and also to the untagged for network services.

I am **currently not using IPTV**, so only untagged traffic is allowed betwen ISP and my router.

## VLANs

- Name: MGMT_VLAN_99  
  VLAN ID: 99  
  Subnet: 192.168.99.0/24  
  Description: Used for SSH, SNMP and other management traffic

- Name: WAN_VLAN_666  
  VLAN ID: 666  
  Subnet: 192.168.1.0/24  
  Description: Used for untagged traffic that comes from ISP router

- Name: TRUSTED_VLAN_10  
  VLAN ID: 10  
  Subnet: 192.168.10.0/24  
  Description: Used for user devices like laptops, mobile phones

- Name: GUEST_VLAN_20  
  VLAN ID: 20  
  Subnet: 192.168.20.0/24  
  Description: Used for untrusted devices and guest devices

- Name: STORAGE_VLAN_30  
  VLAN ID: 30  
  Subnet: 192.168.30.0/24  
  Description: Used for network storage

- Name: IPTV_VLAN_3999  
  VLAN ID: 3999  
  Subnet: ---  
  Description: Used for IPTV multicast


## Interfaces

### Router
| Interface | Status   | Type                            | Description                                    |
| --------- | -------- | ------------------------------- | ---------------------------------------------- |
| sfp1      | Disabled | -                               | -                                              |
| ether1    | Enabled  | Access port to WAN_VLAN_666     | Uplink to ISP router                           |
| ether2    | Disabled | -                               | -                                              |
| ether3    | Disabled | -                               | -                                              |
| ether4    | Enabled  | Link Agregation                 | LACP slave interface of interface bond_4-5     |
| ether5    | Enabled  | Link Agregation                 | LACP slave interface of interface bond_4-5     |
| ether6    | Enabled  | Access port to MGMT_VLAN_99     | Interface used for manual management           |
| ether7    | Enabled  | Access port to MGMT_VLAN_99     | Link to RPI-01                                 |
| ether8    | Enabled  | Access port to TRUSTED_VLAN_10  | Link to personal computer                      |
| ether9    | Disabled | -                               | -                                              |
| ether10   | Enabled  | Trunk port                      | Link to AP1 with VLANs 10, 20, 99 (pasive PoE) |
| bond_4-5  | Enabled  | Access port to STORAGE_VLAN_30  | Link to NAS-01                                 |
|           |          |                                 |                                                |

### AP1
| Interface | Status   | Type                            | Description                                    |
| --------- | -------- | ------------------------------- | ---------------------------------------------- |
| ether1    | Enabled  | Trunk port                      | Uplink to Router with VLANs 10, 20, 99         |
| wlan1     | Enabled  | Access port to TRUSTED_VLAN_10  | SSID: JelencAP                                 |
| wlan2     | Enabled  | Access port to GUEST_VLAN_20    | SSID: JelencAP-Guest                           |
|           |          |                                 |                                                |

### NAS-01
| Interface | Status   | Type                            | Description                                    |
| --------- | -------- | ------------------------------- | ---------------------------------------------- |
| eth0      | Enabled  | Link Agregation                 | LACP slave interface of interface bond0        |
| eth1      | Enabled  | Link Agregation                 | LACP slave interface of interface bond0        |
| bond0     | Enabled  | Access port to STORAGE_VLAN_30  | Uplink to Router                               |
|           |          |                                 |                                                |


## IP Addresses

- Router:
    - WAN_VLAN_666: 192.168.1.2/24
    - MGMT_VLAN_99: 192.168.99.1/24
    - TRUSTED_VLAN_10: 192.168.10.1/24
    - GUEST_VLAN_20: 192.168.20.1/24
    - STORAGE_VLAN_30: 192.168.30.1/24

- AP1:
    - MGMT_VLAN_99: 192.168.99.2/24

- NAS-01:
    - STORAGE_VLAN_30: 192.168.30.10/24

- RPI-01:
    - MGMT_VLAN_99: 192.168.99.101/24
# Router
For my main router I have Mikrotik RB2011UiAS-RM.

## Initial access
Just use serial console and remove existing config with following command.
```
/system reset-configuration no-defaults=yes skip-backup=yes
```

## Configuration
```
# Router

##############################################
# Naming
##############################################

/system identity set name=Router


##############################################
# Bridge
##############################################

/interface bridge add name=BR1 protocol-mode=none vlan-filtering=yes


##############################################
# Bonding
##############################################

/interface bonding add mode=802.3ad name=bond_4-5 slaves=ether4,ether5


##############################################
# Bridge Ports
##############################################

# For access ports set:
#   ingress-filtering=yes
#   frame-types=admit-only-untagged-and-priority-tagged
#   pvid=XXX, where XXX is a VLAN ID that will be applied to untagged ingress traffic
#
# For trunk ports set:
#   ingress-filtering=yes
#   frame-types=admit-only-vlan-tagged
#
# For hybrid ports set:
#   pvid=XXX, where XXX is a VLAN ID that will be applied to untagged ingress traffic
#
#
# Special case: IPTV
# Interface ether1 would be hybrid port as we would also getting tagged VLAN 3999 from ISP router
# Interface ether5, where my TV BOX would be, would also be hybrid port as we would be sending tagged VLAN 3999 over
# and also untagged traffic from GUEST_VLAN_20
# The configuration would look like:
# add bridge=BR1 interface=ether1 pvid=666
# add bridge=BR1 interface=ether5 pvid=20

/interface bridge port

add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=ether6 pvid=99
add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=ether7 pvid=10
add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=ether8 pvid=10
add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=ether9 pvid=20
add bridge=BR1 frame-types=admit-only-vlan-tagged ingress-filtering=yes interface=ether10
add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=ether1 pvid=666
add bridge=BR1 frame-types=admit-only-untagged-and-priority-tagged ingress-filtering=yes interface=bond_4-5 pvid=30


##############################################
# Bridge Ports
##############################################

# For each VLAN ID specify on which interfaces you want tagged traffic
# VLANs that need L3 services, include the bridge itself as a tagged interface
#
# Special case: WAN
# Traffic from ISP router enters my router as untagged traffic on ether1 and is tagged to VLAN 666
# So this VLAN 666 is wanted only on bridge so the traffic can be routed to another VLANs.
#
# Special case: IPTV
# In case of IPTV, VLAN 3999 does not need L3 services on our router.
# So as tagged interfaces we only add ether1 and interface on which we have our TV.
# add bridge=BR1 tagged=ether1,ether5 vlan-ids=3999

/interface bridge vlan

add bridge=BR1 tagged=BR1,ether10 vlan-ids=10
add bridge=BR1 tagged=BR1,ether10 vlan-ids=20
add bridge=BR1 tagged=BR1,ether10 vlan-ids=99
add bridge=BR1 tagged=BR1 vlan-ids=666
add bridge=BR1 tagged=BR1 vlan-ids=30


##############################################
# IP Addressing
##############################################

/interface vlan
add interface=BR1 name=MGMT_VLAN_99 vlan-id=99
add interface=BR1 name=WAN_VLAN_666 vlan-id=666
add interface=BR1 name=TRUSTED_VLAN_10 vlan-id=10
add interface=BR1 name=GUEST_VLAN_20 vlan-id=20
add interface=BR1 name=STORAGE_VLAN_30 vlan-id=30

/ip address
add address=192.168.99.1/24 interface=MGMT_VLAN_99 network=192.168.99.0
add address=192.168.10.1/24 interface=TRUSTED_VLAN_10 network=192.168.10.0
add address=192.168.20.1/24 interface=GUEST_VLAN_20 network=192.168.20.0
add address=192.168.30.1/24 interface=STORAGE_VLAN_30 network=192.168.30.0
add address=192.168.1.2/24 interface=WAN_VLAN_666 network=192.168.1.0

# WAN
/ip route add distance=1 gateway=192.168.1.1

# DNS servers will also be pushed to end devices with DHCP
/ip dns set servers=1.1.1.1,8.8.8.8


##############################################
# IP Services
##############################################

# MGMT_VLAN_99 DHCP Server
/ip pool add name=dhcp_pool0 ranges=192.168.99.100-192.168.99.254
/ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=MGMT_VLAN_99 name=dhcp1
/ip dhcp-server network add address=192.168.99.0/24 gateway=192.168.99.1

# TRUSTED_VLAN_10 DHCP Server
/ip pool add name=dhcp_pool1 ranges=192.168.10.100-192.168.10.254
/ip dhcp-server add address-pool=dhcp_pool1 disabled=no interface=TRUSTED_VLAN_10 name=dhcp2
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1

# GUEST_VLAN_20 DHCP Server
/ip pool add name=dhcp_pool2 ranges=192.168.20.100-192.168.20.254
/ip dhcp-server add address-pool=dhcp_pool2 disabled=no interface=GUEST_VLAN_20 name=dhcp3
/ip dhcp-server network add address=192.168.20.0/24 gateway=192.168.20.1

# STORAGE_VLAN_30 DHCP Server
/ip pool add name=dhcp_pool3 ranges=192.168.30.100-192.168.30.254
/ip dhcp-server add address-pool=dhcp_pool3 disabled=no interface=STORAGE_VLAN_30 name=dhcp4
/ip dhcp-server network add address=192.168.30.0/24 gateway=192.168.30.1


##############################################
# Mgmt settings
##############################################

/interface list add name=MGMT
/interface list member add interface=MGMT_VLAN_99 list=MGMT
/ip neighbor discovery-settings set discover-interface-list=MGMT
/tool mac-server mac-winbox set allowed-interface-list=MGMT
/tool mac-server set allowed-interface-list=MGMT

/ip service set ssh address=192.168.99.0/24
/ip service set winbox address=192.168.99.0/24

/ip service set telnet disabled=yes
/ip service set ftp disabled=yes
/ip service set www disabled=yes
/ip service set api disabled=yes
/ip service set api-ssl disabled=yes

#######################################
# Firewall
#######################################

/interface list add name=WAN
/interface list member add interface=WAN_VLAN_666 list=WAN

/ip firewall filter

add chain=input connection-state=established,related action=accept
add chain=input connection-state=invalid action=drop
add chain=input in-interface-list=MGMT action=accept
add chain=input action=drop

add chain=forward connection-state=established,related action=fasttrack-connection 
add chain=forward connection-state=established,related action=accept
add chain=forward connection-state=invalid action=drop
add chain=forward connection-state=new connection-nat-state=!dstnat in-interface-list=WAN action=drop
add action=drop chain=forward connection-state=new dst-address=192.168.1.0/24 in-interface=TRUSTED_VLAN_10 out-interface-list=WAN
add action=drop chain=forward connection-state=new dst-address=192.168.1.0/24 in-interface=GUEST_VLAN_20 out-interface-list=WAN
add action=drop chain=forward connection-state=new dst-address=192.168.1.0/24 in-interface=STORAGE_VLAN_30 out-interface-list=WAW
add chain=forward connection-state=new in-interface=MGMT_VLAN_99 action=accept
add chain=forward connection-state=new in-interface=TRUSTED_VLAN_10 out-interface-list=WAN action=accept
add chain=forward connection-state=new in-interface=GUEST_VLAN_20 out-interface-list=WAN action=accept
add chain=forward connection-state=new in-interface=STORAGE_VLAN_30 out-interface-list=WAN action=accept

add chain=forward action=drop

/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN


#######################################
# Other
#######################################

/lcd set enabled=no
/tool bandwidth-server set enabled=no

# Disable interfaces that are not in use
/interface 
set disabled=yes [find name=ether2]
set disabled=yes [find name=ether3]
set disabled=yes [find name=ether9]
set disabled=yes [find name=sfp1]
```

## Day 3: Wrap it up
Finally, create new admin account with password and try to login using SSH to 192.168.99.1 IP address.
```
/user add name=netadmin password=netadmin group=full
```
If everything works, disable RoMON and remove default admin account.
```
/tool romon set enabled=no
/user remove admin
```
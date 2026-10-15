# Commands

## Access Switch

```bash
# ether-channel
switch(config)# interface range GigabitEthernet 1/0 - 1
switch(config-if)# description LACP Uplink to MLSA
switch(config-if)# channel-group 2 mode active
switch(config-if)# exit
switch(config)# interface range GigabitEthernet 1/2 - 3
switch(config-if)# description LACP Uplink to MLSB
switch(config-if)# channel-group 3 mode active
switch(config-if)# exit
# trunk config for ether-channel ports
switch(config)# interface Port-channel 2
switch(config-if)# description LACP Uplink to MLSA
switch(config-if)# switchport trunk encapsulation dot1q
switch(config-if)# switchport mode trunk
switch(config-if)# switchport trunk allowed vlan 10,20,30,99
switch(config-if)# exit
switch(config)# interface Port-channel 3
switch(config-if)# description LACP Uplink to MLSB
switch(config-if)# switchport trunk encapsulation dot1q
switch(config-if)# switchport mode trunk
switch(config-if)# switchport trunk allowed vlan 10,20,30,99
switch(config-if)# exit
```

## Cisco MLS

```bash
# enable IP routing
router(config)# ip routing
# vlan creation
router(config)# vlan 10
router(config-vlan)# name users
router(config)# vlan 20
router(config-vlan)# name servers
router(config)# vlan 30
router(config-vlan)# name voice
router(config)# vlan 99
router(config-vlan)# name management
# ether-channel
router(config)# interface range GigabitEthernet 0/0 - 1
router(config-if)# description LACP Downlink to SwitchA
router(config-if)# channel-group 1 mode active
router(config-if)# exit
router(config)# interface range GigabitEthernet 0/2 - 3
router(config-if)# description LACP Downlink to SwitchB
router(config-if)# channel-group 2 mode active
router(config-if)# exit
# trunk config for ether-channel ports
router(config)# interface Port-channel 1
router(config-if)# description LACP Downlink to SwitchA
router(config-if)# switchport trunk encapsulation dot1q
router(config-if)# switchport mode trunk
router(config-if)# switchport trunk allowed vlan 10,20,30,99
router(config-if)# exit
router(config)# interface Port-channel 2
router(config-if)# description LACP Downlink to SwitchB
router(config-if)# switchport trunk encapsulation dot1q
router(config-if)# switchport mode trunk
router(config-if)# switchport trunk allowed vlan 10,20,30,99
router(config-if)# exit
# stp
router(config)# spanning-tree vlan 10 root primary
router(config)# spanning-tree vlan 20 root primary
router(config)# spanning-tree vlan 30 root secondary
router(config)# spanning-tree vlan 99 root secondary
# vrrp
router(config)# interface vlan 10
router(config-if)# ip address 192.168.10.2 255.255.255.0
router(config-if)# no shutdown
router(config-if)# vrrp 10 ip 192.168.10.1
router(config-if)# vrrp 10 priority 120
router(config-if)# vrrp 10 preempt
router(config-if)# exit
router(config)# interface vlan 20
router(config-if)# ip address 192.168.20.2 255.255.255.0
router(config-if)# no shutdown
router(config-if)# vrrp 20 ip 192.168.20.1
router(config-if)# vrrp 20 priority 120
router(config-if)# vrrp 20 preempt
router(config-if)# exit
router(config)# interface vlan 30
router(config-if)# ip address 192.168.30.2 255.255.255.0
router(config-if)# no shutdown
router(config-if)# vrrp 30 ip 192.168.30.1
router(config-if)# vrrp 30 priority 100
router(config-if)# exit
router(config)# interface vlan 99
router(config-if)# ip address 192.168.99.2 255.255.255.0
router(config-if)# no shutdown
router(config-if)# vrrp 99 ip 192.168.99.1
router(config-if)# vrrp 99 priority 100
router(config-if)# exit
```

## ArubaCX MLS

```bash
# system hostname and vlan configuration
Aruba-MLS(config)# hostname Aruba-MLS
Aruba-MLS(config)# vlan 10
Aruba-MLS(config-vlan-10)# name users
Aruba-MLS(config-vlan-10)# exit
Aruba-MLS(config)# vlan 20
Aruba-MLS(config-vlan-20)# name servers
Aruba-MLS(config-vlan-20)# exit
Aruba-MLS(config)# vlan 30
Aruba-MLS(config-vlan-30)# name voice
Aruba-MLS(config-vlan-30)# exit
Aruba-MLS(config)# vlan 99
Aruba-MLS(config-vlan-99)# name management
Aruba-MLS(config-vlan-99)# exit

# lag configuration
Aruba-MLS(config)# interface lag 1
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# no routing
Aruba-MLS(config-if)# lacp mode active
Aruba-MLS(config-if)# vlan trunk allowed 10,20,30,99
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface lag 2
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# no routing
Aruba-MLS(config-if)# lacp mode active
Aruba-MLS(config-if)# vlan trunk allowed 10,20,30,99
Aruba-MLS(config-if)# exit

# physical interface assignment to lags
Aruba-MLS(config)# interface 1/1/1
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# lag 1
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface 1/1/2
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# lag 1
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface 1/1/3
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# lag 2
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface 1/1/4
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# lag 2
Aruba-MLS(config-if)# exit

# stp
Aruba-MLS(config)# spanning-tree
Aruba-MLS(config)# spanning-tree mstp
Aruba-MLS(config)# spanning-tree vlan 10
Aruba-MLS(config)# spanning-tree vlan 10 priority 7
Aruba-MLS(config)# spanning-tree vlan 20
Aruba-MLS(config)# spanning-tree vlan 20 priority 7
Aruba-MLS(config)# spanning-tree vlan 30
Aruba-MLS(config)# spanning-tree vlan 30 priority 6
Aruba-MLS(config)# spanning-tree vlan 99
Aruba-MLS(config)# spanning-tree vlan 99 priority 6

# vrrp and svi configuration
Aruba-MLS(config)# router vrrp enable
Aruba-MLS(config)# interface vlan 10
Aruba-MLS(config-if)# ip address 192.168.10.3/24
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# vrrp 10 address-family ipv4
Aruba-MLS(config-vrrp)# no shutdown
Aruba-MLS(config-vrrp)# address 192.168.10.1 primary
Aruba-MLS(config-vrrp)# priority 100
Aruba-MLS(config-vrrp)# exit
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface vlan 20
Aruba-MLS(config-if)# ip address 192.168.20.3/24
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# vrrp 20 address-family ipv4
Aruba-MLS(config-vrrp)# no shutdown
Aruba-MLS(config-vrrp)# address 192.168.20.1 primary
Aruba-MLS(config-vrrp)# priority 100        
Aruba-MLS(config-vrrp)# exit
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface vlan 30
Aruba-MLS(config-if)# ip address 192.168.30.3/24
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# vrrp 30 address-family ipv4
Aruba-MLS(config-vrrp)# no shutdown
Aruba-MLS(config-vrrp)# address 192.168.30.1 primary
Aruba-MLS(config-vrrp)# priority 120
Aruba-MLS(config-vrrp)# preempt
Aruba-MLS(config-vrrp)# exit
Aruba-MLS(config-if)# exit
Aruba-MLS(config)# interface vlan 99
Aruba-MLS(config-if)# ip address 192.168.99.3/24
Aruba-MLS(config-if)# no shutdown
Aruba-MLS(config-if)# vrrp 99 address-family ipv4
Aruba-MLS(config-vrrp)# no shutdown
Aruba-MLS(config-vrrp)# address 192.168.99.1 primary
Aruba-MLS(config-vrrp)# priority 120
Aruba-MLS(config-vrrp)# preempt
Aruba-MLS(config-vrrp)# exit
Aruba-MLS(config-if)# exit
```
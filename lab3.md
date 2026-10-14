# Preparing Ubuntu boot disk template

- Open Cloud Shell

- Enable the **Compute Engine API**: `$ gcloud services enable compute.googleapis.com`

- Create a nested Ubuntu 24.04 image

    ```bash
    $ gcloud compute images create nested-ubuntu-noble 
        --source-image-family=ubuntu-2404-lts-amd64 
        --source-image-project=ubuntu-os-cloud 
        --licenses https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx
    ```

- Verification: `$ gcloud compute images list --project=eve-ng-home-lab --no-standard-images`


# Creating the VM

-  Machine Configuration

    - General Purpose

    - N2D (CPU Platform AMD Milan)

    - 8 vCPU, 4 core, 32 GB (`n2d-standard-8`)

    - Spot (VM Provisioning Model)

- OS and Storage

    - Image: `nested-ubuntu-noble`

    - Storage: 100 GB - new balanced persistent disk


# EVE-NG Pro Installation

```bash
$ sudo -i
$ wget https://www.eve-ng.net/noble/install-eve-pro.sh
$ chmod +x install-eve-pro.sh
$ ./install-eve-pro.sh
$ apt update
$ apt upgrade

# Reboot EVE. Allow some time for reboot and then press "Reconnect"
# Once the IP wizard screen appears, press ctrl +c and type the command "sudo -i" to become root
# Now follow the IP setup wizard: set IP as DHCP
# Reboot

# After EVE is rebooted, reconnect the SSH session:
$ sudo -i
$ apt update
$ apt install eve-ng-dockers
```


# Commands

Access Switch

```bash
inter ran g1/0 - 1
description LACP Uplink to MLSA
channel-group 2 mode active
exit
interface Port-channel 2
description LACP Uplink to MLSA
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
exit


inter ran g1/2 - 3
description LACP Uplink to MLSB
channel-group 3 mode active
exit
interface Port-channel 3
description LACP Uplink to MLSB
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
exit
```

Cisco MLS

```bash
ip routing
vlan 10
name users
vlan 20
name servers
vlan 30
name voice
vlan 99
name management

inter ran g0/0 - 1
description LACP Downlink to SwitchA
channel-group 1 mode active
exit
interface Port-channel 1
description LACP Downlink to SwitchA
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
exit

inter ran g0/2 - 3
description LACP Downlink to SwitchB
channel-group 2 mode active
exit
interface Port-channel 2
description LACP Downlink to SwitchB
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
exit

spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
spanning-tree vlan 30 root secondary
spanning-tree vlan 99 root secondary

interface vlan 10
ip address 192.168.10.2 255.255.255.0
no shut
vrrp 10 ip 192.168.10.1
vrrp 10 priority 120
vrrp 10 preempt

interface vlan 20
ip address 192.168.20.2 255.255.255.0
no shut
vrrp 20 ip 192.168.20.1
vrrp 20 priority 120
vrrp 20 preempt

interface vlan 30
ip address 192.168.30.2 255.255.255.0
no shut
vrrp 30 ip 192.168.30.1
vrrp 30 priority 100

interface vlan 99
ip address 192.168.99.2 255.255.255.0
no shut
vrrp 99 ip 192.168.99.1
vrrp 99 priority 100
```

ArubaCX MLS

```bash
hostname Aruba-MLS

! Create VLANs
vlan 10
    name users
    exit
vlan 20
    name servers
    exit
vlan 30
    name voice
    exit
vlan 99
    name management
    exit

! LAG 1 - Downlink to SwitchA
interface lag 1
    no shutdown
    no routing
    lacp mode active
    vlan trunk allowed 10,20,30,99
    exit

! LAG 2 - Downlink to SwitchB
interface lag 2
    no shutdown
    no routing
    lacp mode active
    vlan trunk allowed 10,20,30,99
    exit

! Physical members for SwitchA LAG
interface 1/1/1
    no shutdown
    lag 1
    exit
interface 1/1/2
    no shutdown
    lag 1
    exit

! Physical members for SwitchB LAG
interface 1/1/3
    no shutdown
    lag 2
    exit
interface 1/1/4
    no shutdown
    lag 2
    exit

! enable spanning tree
conf t
spanning-tree
spanning-tree mstp

! Aruba is secondary for VLANs 10 and 20
spanning-tree vlan 10
spanning-tree vlan 10 priority 7
spanning-tree vlan 20
spanning-tree vlan 20 priority 7

! Aruba is primary for VLANs 30 and 99
spanning-tree vlan 30
spanning-tree vlan 30 priority 6
spanning-tree vlan 99
spanning-tree vlan 99 priority 6


! enable VRRP
router vrrp enable


! SVI VLAN 10
interface vlan 10
    ip address 192.168.10.3/24
    no shutdown
    vrrp 10 address-family ipv4
        address 192.168.10.1 secondary
        priority 100
        exit
    exit

! SVI VLAN 20
interface vlan 20
    ip address 192.168.20.3/24
    no shutdown
    vrrp 20 address-family ipv4
        address 192.168.20.1 secondary
        priority 100        
    exit

! SVI VLAN 30
interface vlan 30
    ip address 192.168.30.3/24
    no shutdown
    vrrp 30 address-family ipv4
        address 192.168.30.1 primary
        priority 120
        preempt
        exit
    exit

! SVI VLAN 99
interface vlan 99
    ip address 192.168.99.3/24
    no shutdown
    vrrp 99 address-family ipv4
        address 192.168.99.1 primary
        priority 120
        preempt
        exit
    exit
```
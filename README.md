# Packet Tracer - Inter-VLAN Routing Challenge

## 📌 Project Overview
This repository contains the completed setup and configuration steps for the **Inter-VLAN Routing Challenge**. The lab demonstrates how to configure Router-on-a-Stick inter-VLAN routing, VLAN assignments, static trunking with a native VLAN, management SVI configuration, and port security via unused port disabling.

---

## 📋 Addressing & Port Assignment Tables

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | G0/0 | 172.17.25.2 | 255.255.255.252 | N/A |
| **R1** | G0/1.10 | 172.17.10.1 | 255.255.255.0 | N/A |
| **R1** | G0/1.20 | 172.17.20.1 | 255.255.255.0 | N/A |
| **R1** | G0/1.30 | 172.17.30.1 | 255.255.255.0 | N/A |
| **R1** | G0/1.88 | 172.17.88.1 | 255.255.255.0 | N/A |
| **R1** | G0/1.99 | 172.17.99.1 | 255.255.255.0 | N/A |
| **S1** | VLAN 99 | 172.17.99.10 | 255.255.255.0 | 172.17.99.1 |
| **PC1** | NIC | 172.17.10.21 | 255.255.255.0 | 172.17.10.1 |
| **PC2** | NIC | 172.17.20.22 | 255.255.255.0 | 172.17.20.1 |
| **PC3** | NIC | 172.17.30.23 | 255.255.255.0 | 172.17.30.1 |
| **Server** | NIC | 172.17.50.254 | 255.255.255.0 | 172.17.50.1 |

---

### VLAN & Port Assignments Table

| VLAN | Name | Interface(s) |
| :--- | :--- | :--- |
| **10** | Faculty/Staff | FastEthernet 0/11 - 17 |
| **20** | Students | FastEthernet 0/18 - 24 |
| **30** | Guest(Default) | FastEthernet 0/6 - 10 |
| **88** | Native | GigabitEthernet 0/1 |
| **99** | Management | VLAN 99 (SVI) |

---

## ⚙️ Device Configurations

### 1. Switch S1 Configuration

#### A. Create and Name VLANs
> ⚠️ **Note:** VLAN names are case-sensitive and must match exactly.

```cisconetwork
S1> enable
S1# configure terminal

! Create VLANs and Assign Names
S1(config)# vlan 10
S1(config-vlan)# name Faculty/Staff
S1(config-vlan)# vlan 20
S1(config-vlan)# name Students
S1(config-vlan)# vlan 30
S1(config-vlan)# name Guest(Default)
S1(config-vlan)# vlan 88
S1(config-vlan)# name Native
S1(config-vlan)# vlan 99
S1(config-vlan)# name Management
S1(config-vlan)# exit
```

#### B. Assign Access Ports to VLANs
```cisconetwork
! VLAN 30 Access Ports
S1(config)# interface range FastEthernet 0/6 - 10
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 30

! VLAN 10 Access Ports
S1(config)# interface range FastEthernet 0/11 - 17
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 10

! VLAN 20 Access Ports
S1(config)# interface range FastEthernet 0/18 - 24
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 20
S1(config-if-range)# exit
```

#### C. Configure Trunking and Native VLAN
```cisconetwork
! Configure G0/1 as a static trunk connected to R1
S1(config)# interface GigabitEthernet 0/1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 88
S1(config-if)# exit
```

#### D. Configure Management Interface & Default Gateway
```cisconetwork
! Configure SVI for Management
S1(config)# interface vlan 99
S1(config-if)# ip address 172.17.99.10 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit

! Configure Default Gateway
S1(config)# ip default-gateway 172.17.99.1
```

#### E. Disable Unused Ports
Unused active ports on S1 are `FastEthernet 0/1 - 5` and `GigabitEthernet 0/2`.

```cisconetwork
S1(config)# interface range FastEthernet 0/1 - 5, GigabitEthernet 0/2
S1(config-if-range)# shutdown
S1(config-if-range)# exit
```

---

### 2. Router R1 Configuration

#### A. Configure WAN/Point-to-Point Interface (G0/0)
```cisconetwork
R1> enable
R1# configure terminal

S1(config)# interface GigabitEthernet 0/0
R1(config-if)# ip address 172.17.25.2 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### B. Enable Physical Interface G0/1
```cisconetwork
R1(config)# interface GigabitEthernet 0/1
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### C. Configure Subinterfaces (Router-on-a-Stick)
```cisconetwork
! Subinterface VLAN 10 (Faculty/Staff)
R1(config)# interface GigabitEthernet 0/1.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0

! Subinterface VLAN 20 (Students)
R1(config)# interface GigabitEthernet 0/1.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 172.17.20.1 255.255.255.0

! Subinterface VLAN 30 (Guest)
R1(config)# interface GigabitEthernet 0/1.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 172.17.30.1 255.255.255.0

! Subinterface VLAN 88 (Native VLAN)
R1(config)# interface GigabitEthernet 0/1.88
R1(config-subif)# encapsulation dot1Q 88 native
R1(config-subif)# ip address 172.17.88.1 255.255.255.0

! Subinterface VLAN 99 (Management)
R1(config)# interface GigabitEthernet 0/1.99
R1(config-subif)# encapsulation dot1Q 99
R1(config-subif)# ip address 172.17.99.1 255.255.255.0
R1(config-subif)# exit
```

---

## 🔍 Verification Commands

1. **Verify VLANs on Switch S1:**
   ```cisconetwork
   S1# show vlan brief
   ```
2. **Verify Trunk Configuration on S1:**
   ```cisconetwork
   S1# show interfaces trunk
   ```
3. **Verify Routing Table and Subinterfaces on R1:**
   ```cisconetwork
   R1# show ip route
   R1# show ip interface brief
   ```
4. **Connectivity Test Matrix:**
   * Ping PC1 (`172.17.10.21`) ➡️ PC2 (`172.17.20.22`) *(Success)*
   * Ping PC1 (`172.17.10.21`) ➡️ S1 (`172.17.99.10`) *(Success)*
   * Ping PC1 (`172.17.10.21`) ➡️ Server (`172.17.50.254`) *(Success)*
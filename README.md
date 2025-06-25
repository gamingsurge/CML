# 🛡️ Site-to-Site IPsec VPN Lab (KG-175 Simulation)

This lab simulates a site-to-site IPsec VPN tunnel similar to the behavior of NSA Type 1 encryption devices such as the KG-175 TACLANE. It uses two Cisco IOSv routers in Cisco Modeling Labs (CML) to establish a secure encrypted tunnel between two remote subnets.

---

## 🎯 Objective

Create a fully functional IPsec VPN between two LANs:

- Site A (RED Network A): `10.0.10.0/24`
- Site B (RED Network B): `10.0.20.0/24`
- Securely route traffic between PCs in each subnet via encrypted IPsec tunnel

---

## 🖥️ Topology Overview

```
[PC1] --- [R1] =====IPsec VPN===== [R2] --- [PC2]
           |                       |
       10.0.10.1              10.0.20.1
          (WAN: 172.16.1.1 ↔ 172.16.1.2)
```

- **PC1:** 10.0.10.10/24
- **R1 LAN:** 10.0.10.1/24
- **R1 WAN:** 172.16.1.1/30
- **R2 WAN:** 172.16.1.2/30
- **R2 LAN:** 10.0.20.1/24
- **PC2:** 10.0.20.10/24

---

## 🔧 What We Did

### 1. **Configured LAN and WAN IPs** on each router.

### 2. **Set up ISAKMP (IKE Phase 1) Policy**

```cisco
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 2
crypto isakmp key cml123 address <peer WAN IP>
```

### 3. **Defined IPsec Transform Set (IKE Phase 2)**

```cisco
crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
```

### 4. **Created Access Lists to Match "Interesting Traffic"**

```cisco
access-list 100 permit ip 10.0.10.0 0.0.0.255 10.0.20.0 0.0.0.255
```

### 5. **Built and Applied the Crypto Map**

```cisco
crypto map VPN 10 ipsec-isakmp
 set peer <remote WAN IP>
 set transform-set TS
 match address 100
interface GigabitEthernet0/1
 crypto map VPN
```

### 6. **Configured Static Routes**

```cisco
ip route 10.0.20.0 255.255.255.0 172.16.1.2  ! On R1
ip route 10.0.10.0 255.255.255.0 172.16.1.1  ! On R2
```

### 7. **Configured PC IPs and Default Routes**

```sh
sudo ip addr add 10.0.X.10/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 10.0.X.1 dev eth0
```

### 8. **Validated Tunnel Establishment and Traffic**

- Used `show crypto isakmp sa` and `show crypto ipsec sa`
- Verified `#pkts encrypt` and `#pkts decrypt`
- Confirmed end-to-end pings from PC1 to PC2 and vice versa

---

## 🛠️ Tools Used

- Cisco Modeling Labs (CML 2.x)
- Cisco IOSv
- Linux PC containers (BusyBox / Alpine-style nodes)

---

## 📊 Diagram

---

## 📘 Notes

- This is a simulation of functionality — not real Type 1 encryption
- Useful for practicing VPNs, ACLs, crypto maps, and basic IPsec tunnel logic
- Can be extended with NAT, third sites, or dynamic routing protocols

---

## ✅ Key Verification Commands

```cisco
show crypto isakmp sa
show crypto ipsec sa
show access-lists
ping <remote internal IP>
```


# Cisco Packet Tracer Network Design & Configuration

## 1. IP Addressing Scheme

| Network Segment         | Description                 | IP Class | Network Address    | Subnet Mask       | Usable IP Range                  |
| ----------------------- | --------------------------- | -------- | ------------------ | ----------------- | -------------------------------- |
| Router0 <-> Router1     | Inter-router WAN connection | Class B  | 172.16.0.0/30      | 255.255.255.252   | 172.16.0.1 - 172.16.0.2          |
| Router0 <-> Router2     | Inter-router WAN connection | Class A  | 10.0.1.0/30        | 255.255.255.252   | 10.0.1.1 - 10.0.1.2              |
| Router1 <-> Router2     | Inter-router WAN connection | Class A  | 10.0.2.0/30        | 255.255.255.252   | 10.0.2.1 - 10.0.2.2              |
| Router0 LAN (Switch1)   | Servers & PC1 LAN           | Class C  | 192.168.10.0/24    | 255.255.255.0     | 192.168.10.1 - 192.168.10.254    |
| Router1 LAN (Switch2)   | Laptops LAN                 | Class C  | 192.168.20.0/24    | 255.255.255.0     | 192.168.20.1 - 192.168.20.254    |
| Router2 LAN (Switch0)   | PCs & Laptops LAN           | Class C  | 192.168.30.0/24    | 255.255.255.0     | 192.168.30.1 - 192.168.30.254    |

## 2. Server Configuration (Static IPs)
These servers reside on the Router0 LAN (192.168.10.0/24).
- **Default Gateway:** 192.168.10.1
- **DNS Server:** 192.168.10.2 (Self)

| Server        | IP Address     | Subnet Mask   | Notes                                      |
| ------------- | -------------- | ------------- | ------------------------------------------ |
| **DNS Server**| 192.168.10.2   | 255.255.255.0 | A Record: www.testsite.com -> 192.168.10.3 |
| **Website**   | 192.168.10.3   | 255.255.255.0 | Runs default web page                      |
| **DHCP Server**| 192.168.10.4  | 255.255.255.0 | Hosts DHCP scopes for all LANs             |

### DHCP Server Scopes:
- **Pool "LAN-R0":**
  - Network: 192.168.10.0/24
  - Default Gateway: 192.168.10.1
  - DNS Server: 192.168.10.2
- **Pool "LAN-R1":**
  - Network: 192.168.20.0/24
  - Default Gateway: 192.168.20.1
  - DNS Server: 192.168.10.2
- **Pool "LAN-R2":**
  - Network: 192.168.30.0/24
  - Default Gateway: 192.168.30.1
  - DNS Server: 192.168.10.2

## 3. Router CLI Configuration

### Router0 (Hostname: Router0)
```text
enable
configure terminal
hostname Router0

! LAN Interface
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

! WAN Interface to Router1
interface FastEthernet1/0
 ip address 172.16.0.1 255.255.255.252
 no shutdown

! WAN Interface to Router2
interface FastEthernet2/0
 ip address 10.0.1.1 255.255.255.252
 no shutdown

! Static Routes (Targeting R1 LAN and R2 LAN)
ip route 192.168.20.0 255.255.255.0 172.16.0.2
ip route 192.168.30.0 255.255.255.0 10.0.1.2
! Inter-router R1-R2 route (Optional but good for full mesh reachability)
ip route 10.0.2.0 255.255.255.252 172.16.0.2
end
```

### Router1 (Hostname: Router1)
```text
enable
configure terminal
hostname Router1

! LAN Interface
interface FastEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.10.4
 no shutdown

! WAN Interface to Router0
interface FastEthernet1/0
 ip address 172.16.0.2 255.255.255.252
 no shutdown

! WAN Interface to Router2
interface FastEthernet2/0
 ip address 10.0.2.1 255.255.255.252
 no shutdown

! Static Routes
ip route 192.168.10.0 255.255.255.0 172.16.0.1
ip route 192.168.30.0 255.255.255.0 10.0.2.2
! Inter-router R0-R2 route
ip route 10.0.1.0 255.255.255.252 172.16.0.1
end
```

### Router2 (Hostname: Router2)
```text
enable
configure terminal
hostname Router2

! LAN Interface
interface FastEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.10.4
 no shutdown

! WAN Interface to Router0
interface FastEthernet1/0
 ip address 10.0.1.2 255.255.255.252
 no shutdown

! WAN Interface to Router1
interface FastEthernet2/0
 ip address 10.0.2.2 255.255.255.252
 no shutdown

! Static Routes
ip route 192.168.10.0 255.255.255.0 10.0.1.1
ip route 192.168.20.0 255.255.255.0 10.0.2.1
! Inter-router R0-R1 route
ip route 172.16.0.0 255.255.255.252 10.0.1.1
end
```

## 4. Verification Plan
1. **DHCP Verification:** On any PC or Laptop, open the command prompt and run `ipconfig`. Verify that it has received an IP in the correct subnet (192.168.20.X or 192.168.30.X) along with the correct gateway and DNS server.
2. **Ping Tests:**
   - From PC0, ping the DHCP server (`ping 192.168.10.4`) to test end-to-end connectivity across routers.
   - From Laptop1, ping PC0 (`ping 192.168.30.X`) to test connectivity between R1's LAN and R2's LAN.
3. **Web Server Test:** Open the web browser on any PC or Laptop and navigate to `http://www.testsite.com`. The default Packet Tracer webpage should load successfully, verifying both DNS resolution and HTTP connectivity.

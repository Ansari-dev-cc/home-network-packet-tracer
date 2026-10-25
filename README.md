# Home Network Packet Tracer Lab

**Goal:** Built and troubleshot a functional small-office/home-office (SOHO) local area network in Cisco Packet Tracer featuring automated IP assignment (DHCP), local name resolution (DNS), default gateway routing, and OSI Layer 1–7 failure testing.

---

## 1. Topology Diagram

Below is the logical network topology built in Cisco Packet Tracer, connecting end-user PCs through a central Layer 2 switch to a local server (DHCP/DNS) and default gateway router.

![Network Topology Diagram](screenshot_home_network/Screenshot%202026-10-25%20231507.png)

---

## 2. IP Addressing Table

| Device Name | Interface | IPv4 Address | Subnet Mask | Default Gateway | Assignment Method |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Router (Gateway)** | `GigabitEthernet0/0` | `192.168.1.1` | `255.255.255.0` | `N/A` | Static |
| **Server0 (DHCP & DNS)** | `FastEthernet0` | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` | Static |
| **Switch0** | `VLAN 1` | `N/A (Layer 2)` | `N/A` | `N/A` | N/A |
| **PC0** | `FastEthernet0` | `192.168.1.100` | `255.255.255.0` | `192.168.1.1` | DHCP |
| **PC1** | `FastEthernet0` | `192.168.1.101` | `255.255.255.0` | `192.168.1.1` | DHCP |

*(Note: Update the exact IPv4 addresses above if your subnet uses a different range such as `192.168.0.x` or `10.0.0.x`).*

---

## 3. Configuration

### Router Commands (Cisco IOS CLI)
```ios
enable
configure terminal
hostname Home-Router

interface GigabitEthernet0/0
 description LAN Default Gateway
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

### Server Services Configuration
* **DHCP Service (`Services > DHCP`):**
  * **Pool Name:** `serverPool`
  * **Default Gateway:** `192.168.1.1`
  * **DNS Server:** `192.168.1.10`
  * **Start IP Address:** `192.168.1.100`
  * **Subnet Mask:** `255.255.255.0`
  * **Maximum Number of Users:** `50`
* **DNS Service (`Services > DNS`):**
  * **Service State:** `On`
  * **Record Name:** `home.server` (or `www.example.com`)
  * **Record Type:** `A Record`
  * **Address:** `192.168.1.10`

---

## 4. Test Results & Verification Screenshots

*(Note: URLs use `%20` to properly render spaces in GitHub Markdown. Feel free to reorder captions if a specific timestamp corresponds to a different test step).*

### Test 1: Unplugged Cable / Physical Link Check
![Test Result 1](screenshot_home_network/Screenshot%202026-10-25%20232147.png)
*Caption: Verifying physical connectivity failure when the Ethernet link between the PC and switch is disconnected.*

### Test 2: DHCP Service Disabled (Release & Renew)
![Test Result 2](screenshot_home_network/Screenshot%202026-10-25%20232219.png)
*Caption: Disabling the DHCP service on the server and running `ipconfig /release` and `ipconfig /renew`, resulting in a failed DHCP request.*

### Test 3: APIPA Fallback Address (`169.254.x.x`)
![Test Result 3](screenshot_home_network/Screenshot%202026-10-25%20232241.png)
*Caption: Observing the PC automatically assign itself a `169.254.x.x` link-local APIPA address when no DHCP server responds.*

### Test 4: DHCP Restored & Valid Lease Assigned
![Test Result 4](screenshot_home_network/Screenshot%202026-10-25%20232326.png)
*Caption: Re-enabling the DHCP server and confirming the PC receives a valid IPv4 address, subnet mask, default gateway, and DNS server.*

### Test 5: Misconfigured Default Gateway Test
![Test Result 5](screenshot_home_network/Screenshot%202026-10-25%20232336.png)
*Caption: Configuring an invalid default gateway IP on the PC and verifying that local LAN pings succeed while external network pings time out.*

### Test 6: Restoring Default Gateway Connectivity
![Test Result 6](screenshot_home_network/Screenshot%202026-10-25%20232509.png)
*Caption: Fixing the default gateway address on the PC and verifying full end-to-end ping connectivity across the network.*

### Test 7: Missing DNS Record (Ping by IP vs. Ping by Name)
![Test Result 7](screenshot_home_network/Screenshot%202026-10-25%20232754.png)
*Caption: Deleting the DNS A-record and proving that pinging the server by IP address works while pinging by hostname fails.*

### Test 8: DNS Record Restored & Name Resolution Working
![Test Result 8](screenshot_home_network/Screenshot%202026-10-25%20232810.png)
*Caption: Re-adding the DNS record on the server and confirming that hostname resolution and ping by domain name succeed.*

---

## 5. Troubleshooting Log

| Problem | Symptom | Cause | Fix |
| :--- | :--- | :--- | :--- |
| **Unplugged Ethernet Cable** | Link lights turn red/off; `ping` fails and `ipconfig` shows media disconnected. | Layer 1 physical path between the PC NIC and switch port was severed. | Reconnected the Copper Straight-Through cable and waited for STP port convergence. |
| **DHCP Service Turned Off** | `DHCP request failed` in CLI; PC falls back to an APIPA address (`169.254.x.x`) with no gateway. | The DHCP server was disabled and did not respond to the client's `DHCPDISCOVER` broadcast. | Turned the DHCP service back **On** on the server and ran `ipconfig /renew` on the PC. |
| **Wrong Default Gateway** | PC can ping the local server on the same subnet, but pings to outside networks time out. | Local traffic uses Layer 2 ARP via the switch, whereas off-subnet traffic is sent to a non-existent gateway IP. | Updated the Default Gateway on the PC to match the router's actual LAN interface IP. |
| **Deleted DNS Record** | `ping <server-IP>` succeeds with 0% loss, but `ping <hostname>` returns `could not find host`. | IP routing is intact, but the DNS server lacks the `A Record` to translate the domain name into an IPv4 address. | Re-created the `A Record` mapping the hostname to the server's IP and ran `ipconfig /flushdns`. |

---

## 6. What I Learned

Building a network from scratch is helpful, but intentionally breaking each layer taught me how to read network symptoms instead of guessing. When a PC gets a `169.254.x.x` address, I now immediately know it is a DHCP communication failure rather than a broken NIC. I also learned why a computer with a bad default gateway can still talk to local devices on the same switch, and how to isolate DNS failures by comparing `ping` by IP address against `ping` by hostname.

---

## 7. How to Upload This to GitHub

Run these commands inside your project folder (where your `.pkt` file and `screenshot_home_network/` folder are saved):

```bash
git init
git add .
git commit -m "Initial commit: Home network Packet Tracer lab, screenshots, and documentation"
git branch -M main
git remote add origin https://github.com/<your-username>/home-network-packet-tracer.git
git push -u origin main
```

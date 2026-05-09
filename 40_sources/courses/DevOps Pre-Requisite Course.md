---
tags:
  - devops
  - networking
  - linux
  - kubernetes
feature: 
type: course
author: KodeKloud
source: 
---
# DevOps Pre-Requisite Course

## Networking
RhqFL?ZD2kwVAEW
### Swithing
#### ip link 
The `ip link` command is used to display and configure network interfaces (like Ethernet, Wi-Fi, loopback, etc.) on Linux systems.

**Basic usage:**
- `ip link` or `ip link show` - Lists all network interfaces and their status
- `ip link set <interface> up/down` - Brings an interface up or down
- `ip link set <interface> name <newname>` - Renames an interface

**What you'll see in the output:**

- Interface names (eth0, wlan0, lo, etc.)
- MAC addresses
- Interface state (UP/DOWN)
- MTU (Maximum Transmission Unit)
- Link type (Ethernet, loopback, etc.)

It's part of the newer `ip` command suite that's gradually replacing older tools like `ifconfig`. The `ip link` command specifically deals with the data link layer (Layer 2) of networking - managing the physical/virtual network interfaces themselves rather than IP addresses or routing.

#### ip addr add
The `ip addr add` command assigns an IP address to a network interface.

**Basic syntax:** `ip addr add <IP_address>/<subnet_mask> dev <interface_name>`

**In your example: `ip addr add 192.168.1.10/24 dev eth0`**

- `192.168.1.10/24` - The IP address and subnet mask (24-bit netmask = 255.255.255.0)
- `dev eth0` - Specifies which network interface to assign this IP to

**The "dev" parameter:** `dev` stands for "device" and tells the command which network interface (device) should receive this IP address. It's a required parameter because you need to specify which physical or virtual network interface you're configuring.

**Other examples:**

- `ip addr add 10.0.0.5/16 dev wlan0` - Assigns IP to Wi-Fi interface
- `ip addr add 127.0.0.2/8 dev lo` - Assigns IP to loopback interface

You can assign multiple IP addresses to the same interface by running the command multiple times with different IPs. Use `ip addr show` to view all assigned addresses.

After adding IPs address to a network different computers can comunicate inside the same network
### Routing
The router helps to connect two networks together 
#### route
The `route` command displays and manipulates the kernel's IP routing table, which determines how network traffic is directed.

**Basic usage:**

- `route` or `route -n` - Display the routing table (-n shows numerical addresses instead of resolving hostnames)
- `route add` - Add a route
- `route del` - Delete a route

**Common examples:**

- `route add default gw 192.168.1.1` - Set default gateway
- `route add -net 10.0.0.0/8 gw 192.168.1.1` - Add route to specific network
- `route del -net 10.0.0.0/8` - Delete a route

**Routing table columns:**

- **Destination** - Target network or host
- **Gateway** - Next hop router (or 0.0.0.0 for direct connection)
- **Genmask** - Subnet mask
- **Flags** - Route properties (U=up, G=gateway, H=host, etc.)
- **Iface** - Network interface to use

**Modern alternative:** The `route` command is being replaced by `ip route`, which offers more features:

- `ip route show` - Display routes
- `ip route add default via 192.168.1.1` - Add default route
- `ip route add 10.0.0.0/8 via 192.168.1.1` - Add specific route

The routing table determines where packets go when they leave your system - either to a directly connected network or through a gateway to reach remote networks.


`echo 1 > /proc/sys/net/ipv4/ip_forward`

## DNS



![[DevOps-Pre-Requisites-v2-1.pdf]]


![[Additional-Resources.pdf]]


![[Networking-Basics.pdf]]
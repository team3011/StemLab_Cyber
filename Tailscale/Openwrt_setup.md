### Setup OpenWrt to act as a Tailscale exit node with subnet access
### Requirements!
- none
  
### 1. Install Tailscale on OpenWrt:
  SSH into your router and run:
  ```
  opkg update
  opkg install tailscale
  ```
### 2. Start and Authenticate:
  ```
  /etc/init.d/tailscaled enable
  /etc/init.d/tailscaled start
  tailscale up
  ```
  You’ll get a URL to authenticate in your browser.<br>
  After login, your router appears in the Tailscale admin console.
### 3. Share Local Network (LAN) — Subnet Router:
  Let’s assume your LAN subnet is 10.42.1.0/24.<br>
  Enable subnet routing and make the router route all remote traffic through it:
  ```
  tailscale up --advertise-routes=10.42.1.0/24 --advertise-exit-node
  ```
  Enable Tailscale IP forwarding:<br>
  ```
  uci set network.globals.forwarding=1
  uci commit network
  /etc/init.d/network restart
  ```
  Then, in the Tailscale admin console, approve the subnet route.<br>
  Now from any Tailscale-connected device, you can access your LAN devices (e.g., 10.42.1.x).
### 4. Persistent Tailscale Configuration
  Edit the file:
  ```
  vi /etc/rc.local
  ```
  Replace all the contents of the file with this:
  ```
  #!/bin/sh
  # /etc/rc.local – Tailscale auto setup for OpenWrt

  ### --- 1️⃣ Enable IP forwarding ---
  # Make sure IPv4 forwarding is on so Tailscale can route traffic
  echo 1 > /proc/sys/net/ipv4/ip_forward

  ### --- 2️⃣ Ensure firewall rules are ready ---
  # Allow NAT for Tailscale traffic out the WAN
  # Replace 'eth0' below if your WAN interface is named differently (e.g. pppoe-wan, wan)
  iptables -t nat -A POSTROUTING -s 100.64.0.0/10 -o eth0 -j MASQUERADE

  # Add Tailscale interface to LAN zone if not already
  # (this command is safe to run even if it's already added)
  uci add_list firewall.@zone[0].network='tailscale0' 2>/dev/null
  uci commit firewall
  /etc/init.d/firewall restart

  ### --- 3️⃣ Start Tailscale if not already running ---
  /etc/init.d/tailscaled enable
  /etc/init.d/tailscaled start

  ### --- 4️⃣ Bring Tailscale up with your configuration ---
  # Advertise your LAN subnet and act as an exit node
  # Replace 192.168.1.0/24 with your LAN subnet
  /usr/sbin/tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node --reset

  ### --- 5️⃣ Done ---
  exit 0
  ```
  To save and exit type press {ESC} then:
  ```
  :wq
  ```
  Then press {Enter}
  


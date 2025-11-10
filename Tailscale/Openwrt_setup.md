### Setup OpenWrt to act as a Tailscale exit node with subnet access
### Requirements!
- none
  
### 1. Install Tailscale on OpenWrt:
  SSH into your router and run:
  ```
  opkg update
  opkg install tailscale
  ```
### 2. Persistent Tailscale Configuration
  Edit the file:
  ```
  vi /etc/rc.local
  ```
  To modify the file press i then {enter}
  Replace all the contents of the file with this:
  ```
  #!/bin/sh
# /etc/rc.local ... Tailscale auto setup for OpenWrt

### --- 1...... Enable IP forwarding ---
# Make sure IPv4 forwarding is on so Tailscale can route traffic
echo 1 > /proc/sys/net/ipv4/ip_forward

### --- 2...... Ensure firewall rules are ready ---
# Allow NAT for Tailscale traffic out the WAN
# Replace 'eth0' below if your WAN interface is named differently (e.g. pppoe-wan, wan)
iptables -t nat -A POSTROUTING -s 192.168.179.0/24 -o wwan -j MASQUERADE

### --- 3...... Start Tailscale if not already running ---
/etc/init.d/tailscaled enable
/etc/init.d/tailscaled start

### --- 4...... Bring Tailscale up with your configuration ---
# Advertise your LAN subnet and act as an exit node
# Replace 192.168.1.0/24 with your LAN subnet
/usr/sbin/tailscale up --advertise-routes=10.42.1.0/24 --advertise-exit-node --reset

### --- 5...... Done ---
exit 0
  ```
  To save and exit type press {ESC} then:
  ```
  :wq
  ```
  Then press {Enter}<br>
  Reboot the router<br>
  Then, in the Tailscale admin console, approve the subnet route.<br>
  Now from any Tailscale-connected device, you can access your LAN devices (e.g., 10.42.1.x).


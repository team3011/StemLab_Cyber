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
  Save and exit (Esc, then :wq)<br>
  Then press {Enter}<br>
  run the following command:
  ```
  chmod +x /etc/rc.local
  ```
  Reboot the router<br>
  Run the following command:
  ```
  tailscale up
  ```
  Authenticate your Tailscale instance<br>
  Then stop Tailscale
  ```
  /etc/init.d/tailscaled stop
  ```
  Create a permanent directory
  ```
  mkdir -p /etc/tailscale
  ```
  Move existing state files
  ```
  mv /var/lib/tailscale/* /etc/tailscale/ 2>/dev/null
  ```
  Tell tailscaled to use that directory.
  Open /etc/init.d/tailscaled in vi:
  ```
  vi /etc/init.d/tailscaled
  ```
  Replace the file with
  ```
  #!/bin/sh /etc/rc.common
  # Tailscaled init script for OpenWrt
  START=99
  USE_PROCD=1
  
  start_service() {
      procd_open_instance
      procd_set_param command procd_set_param command /usr/sbin/tailscaled --state=/etc/tailscale/tailscaled.state
      procd_set_param respawn
      procd_close_instance
  }
  
  stop_service() {
      killall tailscaled 2>/dev/null
  }
  ```
  Save and exit (Esc, then :wq)<br>
  Set permissions:
  ```
  chown -R root:root /etc/tailscale
  chmod 700 /etc/tailscale
  ```
  Restart and authenticate one more time<br>
  Then, in the Tailscale admin console, approve the subnet route.<br>
  Now from any Tailscale-connected device, you can access your LAN devices (e.g., 10.42.1.x).

### 3. Test it
  SSH back into the router
  run the following:
  ```
  tailscale status
  ```
  You should still be logged in, no re-auth required.
  

### Setup OpenWrt to act as a Tailscale exit node with subnet access
### Requirements!
- none
  
#### 1. Install Tailscale on OpenWrt:
  SSH into your router and run:
  ```
  opkg update
  opkg install tailscale
  ```
#### 2. Start and Authenticate:
  ```
  /etc/init.d/tailscaled enable
  /etc/init.d/tailscaled start
  tailscale up
  ```
  You’ll get a URL to authenticate in your browser.<br>
  After login, your router appears in the Tailscale admin console.
#### 3. Share Local Network (LAN) — Subnet Router:
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

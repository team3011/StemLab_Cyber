### Setup OpenWrt to act as a Tailscale exit node with subnet access
### Requirements!
- none

### 1. Flash OpenWRT onto microSD card using balenaEtcher
### 2. connect the rpi to a computer
  using a network cable<br>
  manually set computer to 102.168.1.2/255.255.255.0/192.168.1.1/8.8.8.8<br>
  open browser and go to 192.168.1.1<br>
  username is root/[blank]<br>
  set password
### 3. Setup ALFA usb dongle
  goto Network -> Wireless<br>
  click Scan<br>
  connect to an SSID<br>
  click save and apply<br>
  goto System -> Software<br>
  click update lists<br>
  install kmod-mt76x2u<br>
  reboot<br>
  goto Network -> Wireless<br>
  remove everything associated with the built in nic<br>
  click scan on radio1 and connect to SSID<br>
  click save and apply<br>
### 4. Setup bridge
  goto Network -> Interfaces<br>
  click on edit on the lan<br>
  set ip address to 10.42.1.1<br>
  click save and apply<br>
  set computer network to dhcp<br>
  open browser and go to 10.42.1.1<br>
  
  
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
  ### --- 1...... Enable IP forwarding ---
  # Make sure IPv4 forwarding is on so Tailscale can route traffic
  echo 1 > /proc/sys/net/ipv4/ip_forward

  ### --- 2...... Start Tailscale if not already running ---
  /etc/init.d/tailscaled enable
  /etc/init.d/tailscaled start

  ### --- 3...... Bring Tailscale up with your configuration ---
  # Advertise your LAN subnet and act as an exit node
  # Replace 192.168.1.0/24 with your LAN subnet
  /usr/sbin/tailscale up --advertise-routes=10.42.1.0/24 --advertise-exit-node --reset

  ### --- 4......
  nft -f /etc/nftables.conf
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
    procd_set_param command /usr/sbin/tailscaled --state=/etc/tailscale/tailscaled.state
    procd_set_param caps "net_admin net_bind_service"
    procd_close_instance
  }
  
  stop_service() {
      killall tailscaled 2>/dev/null
  }
  ```
  Save and exit (Esc, then :wq)<br>
  Set permissions:
  ```
  mkdir -p /etc/tailscale
  touch /etc/tailscale/tailscaled.state
  chown -R root:root /etc/tailscale
  chmod 700 /etc/tailscale
  ```
  Restart and authenticate one more time<br>
  ```
  /etc/init.d/tailscaled start
  tailscale up
  ```
  Then, in the Tailscale admin console, approve the subnet route.<br>
  Now from any Tailscale-connected device, you can access your LAN devices (e.g., 10.42.1.x).

### 3. Test it
  SSH back into the router
  run the following:
  ```
  tailscale status
  ```
  You should still be logged in, no re-auth required.
  

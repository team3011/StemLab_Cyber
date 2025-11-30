### Setup OpenWrt to act as a Tailscale exit node with subnet access
### Requirements!
- none

### 1. Flash OpenWRT onto microSD card using balenaEtcher
  after you have flashed the microSD card with openwrt
  insert the usb reader into a ubuntu workstation
  open a terminal and complete the following:
  ```
  sudo apt-get update
  sudo apt-get update
  sudo apt install gparted
  ```
  
### 2. connect the rpi to a computer
  using a network cable<br>
  manually set computer to 192.168.1.2/255.255.255.0/192.168.1.1/8.8.8.8<br>
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
  verify connection by pinging openwrt.org
### 4. Setup bridge
  goto Network -> Interfaces<br>
  click on edit on the lan<br>
  set ip address to 10.42.1.1/24<br>
  click save and apply<br>
  set computer network to dhcp<br>
  open browser and go to 10.42.1.1<br>
  verify connection by pinging openwrt.org on both the router and the computer
### 5. Setup Tailscale
  goto System -> Software<br>
  click update lists<br>
  install tailscale<br>
  ssh in to openwrt<br>
  Install iptables<br>
  ```
  opkg install iptables iptables-mod-tproxy kmod-nf-nat kmod-nf-conntrack
  ```
  Then enable and start the daemon:
  ```
  tailscale up --advertise-routes=10.42.1.0/24 --advertise-exit-node
  ```
  Then make persistent on boot:
  ```
  uci set tailscale.tailscale=service
  uci set tailscale.tailscale.enabled='1'
  uci set tailscale.tailscale.flags='--advertise-routes=10.42.1.0/24 --advertise-exit-node'
  uci commit tailscale
  ```
  restart tailscale
  ```
  /etc/init.d/tailscale restart
  tailscale status
  ```
  update tailscale
  ```
  tailscale update
  ```

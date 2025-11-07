### Operating System Upgrade and Aruba Instant Access Points Configuration Documentation
### Requirements!
- [TFTP Server](https://github.com/team3011/StemLab_Cyber/blob/main/Aruba_APIN0205_Documentation/tftp%20server.md)
- [Aruba IAP Firmware](https://github.com/team3011/StemLab_Cyber/blob/main/Aruba_APIN0205_Documentation/ArubaInstant_Taurus_6.5.4.15_73677)
- Ubuntu Desktop

#### 1. Upgrade OS Command:

During the upgrade process of the operating system (OS) for Aruba Instant Access Points (IAPs), follow these steps:

- **Step 1**: Configure your computer with a static IP (I used 192.168.1.10).
- **Step 2**: Install and connect to AP using screen

  ```
  sudo apt-get install screen
  sudo screen /dev/ttyUSB0 9600
  ```
  
- **Step 3**: Power cycle the AP:
- **Step 4**: As the AP boots, stop autoboot by pressing 'Enter'. You should see:

  ```
  apboot>
  ```
  
- **Step 5**: Configure network settings in apboot
  Set your TFTP and IP parameters
  ```
  setenv ipaddr 192.168.1.20
  setenv netmask 255.255.255.0
  setenv serverip 192.168.1.10
  ```
  Verify with:
  ```
  printenv
  ```

- **Step 6**: Flash the Instant image
  ```
  upgrade os 0 ArubaInstant_Taurus_6.5.4.15_73677
  ```
  Wait for it to transfer and flash - it can take several minutes.

- **Step 7**: Save and reboot
  Once the upgrade is complete:
  ```
  saveenv
  reset
  ```
- **Step 8**: Verify Instant Mode enabled
  After the reboot:
    The AP should broadcast a WiFi SSID like SetMeUp-xx:xx:xx
    You can connect and access the web UI at:
      http://instant.arubanetworks.com
      https://192.168.1.1
    Default login is admin/[blank]


   stopped here


  
  
- Replace 172.16.1.13 with the ip address of youre tftp server. 
### This takes a long time..

  ```
  upgrade os 1 172.16.1.13:/ArubaInstant_Taurus_6.5.4.15_73677
  ```
### This takes a long time..

  - These commands initiate the OS upgrade on the IAPs.
  - `os 0` and `os 1` specify separate partitions for the new OS on each AP.
  - `172.16.1.13` is the IP address where the new OS image is located.
  - `/ArubaInstant_Taurus_6.5.4.15_73677` denotes the exact location and filename of the new OS image.

#### 2. Proginv System Ccode Command:

To modify the system's configuration code (ccode), essential for licensing and features management, use the command:
```
proginv system ccode CCODE-RW-de6fdb363ff04c13ee261ec04fbb01bdd482d1cd
```

#### 3. Invent -w Command:

This command likely interacts with the device's inventory or wireless configuration settings. Execute it to prepare the device for operation:
```
invent -w
```

#### 4. Factory Reset Command:

To reset the device to its factory default settings, use:
```
factory_reset
```

#### 5. Saveenv Command:

Execute the following command to save current configurations:
```
saveenv
```

#### 6. Boot Command:

Initiate the device's boot process to load OS and configurations:
```
boot
```

#### 7. Master AP Configuration:

To configure an IAP as the master:
```
(Instant AP)# iap-master
```
To verify the master configuration:
```
(Instant AP)# show ap-env
```

### Accessing the Virtual Controller:

To access the virtual controller on the master AP, follow this link: [Aruba Networks Virtual Controller](https://setmeup.arubanetworks.com:4343/#home)

- **User**: admin
- **Password**: admin

After setting up the master AP, additional Aruba APs will automatically assume slave roles, simplifying network management and ensuring uniform configurations.

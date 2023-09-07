starting with fresshh ASA 
To configure a Cisco ASA 5510 with default settings to allow internet access while separating the outside and inside networks with basic NAT and DHCP, follow these steps:

**Important Note**: Before proceeding, ensure that you have console access to the Cisco ASA 5510 through a console cable and terminal emulation software like PuTTY or a similar tool.

1. **Connect to the Cisco ASA 5510**:
   - Connect one end of the console cable to the console port on the ASA 5510.

2. **Enter privileged EXEC mode by typing**:

   ```shell
   ciscoasa> enable
   ```

   - Enter global configuration mode:

   ```shell
   ciscoasa# conf t
   ```

3. **Set Hostname** (Optional):
   - You can set a hostname for the ASA if desired:

   ```shell
   ciscoasa(config)# hostname G5-ASA-5510
   G5-ASA-5510(config)# wr mem

   ```

   

4. **Configure Interfaces**:
   - Define the outside and inside interfaces.

   ```shell
   G5-ASA-5510(config)# int Ethernet0/0
   G5-ASA-5510(config)# nameif outside
   G5-ASA-5510(config)# ip address 192.168.0.50 255.255.255.0
   G5-ASA-5510(config)# security-level 0
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# int Ethernet0/1
   G5-ASA-5510(config)# nameif inside
   G5-ASA-5510(config)# ip address 10.0.0.1 255.255.255.0
   G5-ASA-5510(config)# security-level 100
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# wr mem
   ```

   - The above configuration says that Ethernet0/0 is the outside interface and Ethernet0/1 is the inside interface.


5. **Configure Routing**:

   ```shell
   G5-ASA-5510(config)# route 0 0 192.168.0.0 

   ```


6. **Configure DHCP inside**:

   ```shell
   G5-ASA-5510(config)# dhcpd address 10.0.0.100-10.0.0.200 inside
   G5-ASA-5510(config)# dhcpd dns 10.0.0.2 interface inside   
   G5-ASA-5510(config)# dhcpd lease 7200                          
   G5-ASA-5510(config)# dhcpd enable inside 
   G5-ASA-5510(config)# wr mem
   ```




5. **Configure NAT**:
   - Configure basic NAT to allow inside devices to access the internet:

   ```shell
   G5-ASA-5510(config)# object network myNatPool
   G5-ASA-5510(config-network-object)# range 192.168.0.51 192.168.0.52
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# object network myInsNet  
   G5-ASA-5510(config-network-object)# subnet 10.0.0.0 255.255.255.0  
   G5-ASA-5510(config-network-object)# nat (inside,outside) dynamic myNatPool
   G5-ASA-5510(config-network-object)# exit  
   G5-ASA-5510(config)# object network myLinuxServer
   G5-ASA-5510(config-network-object)# host 192.168.0.50  
   G5-ASA-5510(config-network-object)# nat (outside,inside) static 192.168.2.13
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# same-security-traffic permit inter-interface
   G5-ASA-5510(config)# wr mem 

   ```
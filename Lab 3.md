# IT&C 247 - Lab 3

## Introduction

In this lab, we'll delve into static routes, spanning tree protocol and port security. We will navigate configuring static routes to direct network traffic efficiently, implement spanning tree protocol to prevent loops and ensure network redundancy, and secure switch ports by specifying the MAC addresses of devices that are allowed to connect.

## Objectives

1. Configure spanning tree protocol (STP) to establish redundancy and prevent network loops among the floor switches.
1. Establish static routes between the network's two sides to facilitate efficient communication and seamless data transfer.
1. Implement  port security on switch access ports

## Network Tables
We will be working with the same base topology you created in labs 1 and 2 and adding to it. You won't be adding any more hardware in this lab just links to bring everything together and add more redundancy into the network.

![](./Lab-3-images/topology.png)

   <div style="page-break-after: always"></div>

These tables contain all of the network information you will need to set up the network for Lab 2. They are the same tables from lab 1 just with the extra devices added.

| PC Name | IP Address  | Gateway | Subnet | VLAN |  
| - | - | - | - | - |  
| PC1 | 10.1.10.2 | 10.1.10.1 | /24 | 10  
| PC2 | 10.1.11.2 | 10.1.11.1 | /24 | 11  
| PC3 | 10.1.12.2 | 10.1.12.1 | /24 | 12  
| PC4 | 10.1.10.3 | 10.1.10.1 | /24 | 10  
| PC5 | 10.1.10.4 | 10.1.10.1 | /24 | 10  
| PC6 | 10.1.12.3 | 10.1.12.1 | /24 | 12  
| PC7 | 10.2.13.2 | 10.2.13.1 | /24 | 13  
| PC8 | 10.2.14.2 | 10.2.14.1 | /24 | 14  
| PC9 | 10.2.15.2 | 10.2.15.1 | /24 | 15  
| PC10   | 10.2.13.3 | 10.2.13.1 | /24 | 13  
| PC11   | 10.2.14.3 | 10.2.14.1 | /24 | 14  
| PC12   | 10.2.15.3 | 10.2.15.1 | /24 | 15
| SRV1 | 10.2.16.2 | 10.2.16.1 | /24 | 16
| LT1  | 10.1.10.5 | 10.1.10.1 | /24 | 10
| LT2  | 10.1.10.6 | 10.1.10.1 | /24 | 10
| LT3  | 10.1.10.7 | 10.1.10.1 | /24 | 10

| Switch Name |   IP Address | Subnet | VLAN |  
| - | - | - | - |  
| Floor-SW-1   | 10.1.11.3 | /24 | 11
| Floor-SW-2   | 10.1.11.4 | /24 | 11  
| Floor-SW-3   | 10.2.11.3 | /24 | 11  
| Floor-SW-4   | 10.2.11.4 | /24 | 11  
| Distro-SW-1  | 10.1.11.5 | /24 | 11  
| Distro-SW-2  | 10.2.11.5 | /24 | 11  

   <div style="page-break-after: always"></div>

| VLAN | IP Range |
| - | - |
| 10 | 10.1.10.0/24 |
| 11 | 10.1.11.0/24 |   
| 12 | 10.1.12.0/24 |  
| 11 | 10.2.11.0/24 | 
| 13 | 10.2.13.0/24 |   
| 14 | 10.2.14.0/24 |   
| 15 | 10.2.15.0/24 | 
| 16 | 10.2.16.0/24 |

| Device | Interface | IP Address |
| - | - | - |
| Core-1 | GigabitEthernet0/0/0 | 192.168.1.1/30 |
| Core-2 | GigabitEthernet0/0/0 | 192.168.1.2/30 |

| Switch       | Switch Interface | End Device | End Device Interface |
| ------------ | ---------------- | ---------- | ---------------------| 
| Floor-SW-1   | Fa1              | PC1        | Fa0                  |
| Floor-SW-1   | Fa2              | PC2        | Fa0                  |
| Floor-SW-1   | Fa3              | PC3        | Fa0                  |
| Floor-SW-2   | Fa1              | PC4        | Fa0                  |
| Floor-SW-2   | Fa2              | PC5        | Fa0                  |
| Floor-SW-2   | Fa3              | PC6        | Fa0                  |
| Floor-SW-2   | Fa4              | AP         | Port0                |
| Floor-SW-3   | Fa1              | PC7        | Fa0                  |
| Floor-SW-3   | Fa2              | PC8        | Fa0                  |
| Floor-SW-3   | Fa3              | PC9        | Fa0                  |
| Floor-SW-4   | Fa1              | PC10       | Fa0                  |
| Floor-SW-4   | Fa2              | PC11       | Fa0                  |
| Floor-SW-4   | Fa3              | PC12       | Fa0                  |
| Floor-SW-4   | Fa4              | SRV        | Fa0                  |



| Floor Switch | Switch Interface | Floor Switch Port Channel | Distro Switch | Distro Switch Interface | Distro Switch Port Channel |
| ------------ | ---------------- | ------------------------- | --------------| ----------------------- | -------------------------- |
| Floor-SW-1   | Fa23             | Po1                       | Distro-SW-1   | Fa23                    | Po1                        |
| Floor-SW-1   | Fa24             | Po1                       | Distro-SW-1   | Fa24                    | Po1                        |
| Floor-SW-2   | Fa21             | Po1                       | Distro-SW-1   | Fa21                    | Po2                        |
| Floor-SW-2   | Fa22             | Po1                       | Distro-SW-1   | Fa22                    | Po2                        |
| Floor-SW-3   | Fa23             | Po1                       | Distro-SW-2   | Fa23                    | Po1                        |
| Floor-SW-3   | Fa24             | Po1                       | Distro-SW-2   | Fa24                    | Po1                        |
| Floor-SW-4   | Fa21             | Po1                       | Distro-SW-2   | Fa21                    | Po2                        |
| Floor-SW-4   | Fa22             | Po1                       | Distro-SW-2   | Fa22                    | Po2                        |
| Floor-SW-1   | Fa20             | Po2                       | Floor-SW-2    | Fa20                    | Po2                        |
| Floor-SW-1   | Fa21             | Po2                       | Floor-SW-2    | Fa21                    | Po2                        |
| Floor-SW-3   | Fa20             | Po2                       | Floor-SW-4    | Fa20                    | Po2                        |
| Floor-SW-3   | Fa21             | Po2                       | Floor-SW-4    | Fa21                    | Po2                        |

| Distro Switch | Distro Switch Interface | Router Number| Router Interface |
| ------------- | ------------------------| -------------| ---------------- |
| Distro-SW-1   | Gig 0/1                 | Router-1     | Gig 0/1/0        |
| Distro SW-2   | Gig 0/1                 | Router-2     | Gig 0/1/0        |

## Spanning Tree Protocol (STP)

Spanning Tree Protocol (STP) is a network protocol used to prevent loops in Ethernet networks. When multiple switches are connected in a network, there is a risk of creating a loop, where data packets can circulate endlessly, causing network congestion and failures, which are called broadcast storms. Since we are creating a loop between our 3 floor switches, we need to enable STP to prevent broadcast storms. STP helps prevent broadcast storms by looking for loops, and temporarily disabling ports. It temporarily disables redundant paths and reactivates them if the primary path fails, thus maintaining network reliability and efficiency. This process is crucial for stable and loop-free network operations.

### Configuring Spanning Tree Protocol (STP)

1. **Add the redundant cables**
   - Add in the additional cables between the two floor switches as shown in the above network topology. 
   - Trunk the appropriate VLANs across the cables. As an example, you can check to the other trunks you have set up in previous labs.
   - The new cables will be configured in a port channel, like you have done in previous labs.

1. **Selecting STP Mode**
   - Determine which STP mode best suits your network: RSTP, MSTP, or PVST+.
     - RSTP (Rapid Spanning Tree Protocol): For faster convergence in larger networks.
     - MSTP (Multiple Spanning Tree Protocol): Conglomerates multiple VLANs into one Spanning Tree instance.
     - PVST+ (Per VLAN Spanning Tree Protocol): Provides a separate spanning tree for each VLAN.
     - Consider that we are in a smaller network with multiple VLANs, use PVST+ to create a tree for each VLAN.

1. **Enabling STP on Interfaces**

   - Enable STP using the command `spanning-tree mode pvst` for the interfaces connecting the floor switches to one another. Tip: Use the `int range` command. 

1. **Verifying STP Configuration**
   - Verify the STP configuration using commands like `show spanning-tree` on the switch's CLI.
   - Ensure that the STP topology is correctly formed and that there are no loops in the network.
   - Shut down some of the redundant connections to verify that traffic can still be routed.

   <div style="page-break-after: always"></div>

## Static Routes

### Configuring Static Routes

*Note: Before you configure Static routing, assign IP addresses to the ports connecting the core router to the other. **The IP addresses and gateways you use will be found in the IP table table above.**  Refer to this command below to help, and note its similarity to creating the SVIs.*
   ```
      interface GigabitEthernet0/0/0
      ip address 192.168.1.1 255.255.255.252
   ```
1. **Identifying Route Destinations**
   - Determine the destination networks for which static routes are required. This includes remote networks and specific subnets. Check the pass-off requirements (and the IP addressing tables above) to see what static routes need to be created.

1. **Adding Static Routes**
   - Use the router's CLI to add static routes using the `ip route` command. For example:
     ```
     ip route <destination_network> <subnet_mask> <next_hop_ip_address>
     ```
     Let's break this command down:
     - The destination network is the traffic you want to send across the core routers.
     - The subnet mask will remain consistent across all networks you route.
     - The next hop is the IP address of the router interface on the other side.

     Replace `<destination_network>`, `<subnet_mask>`, and `<next_hop_ip_address>` with the appropriate values for your network.
     For example, to get traffic from the left side of the network to the right side you would run this command on core-1 :
     ```
     ip route 10.2.15.0 255.255.255.0 192.168.1.2
     ```
     This will send any traffic that is in the destination range 10.2.15.0/24 from core-1 to core-2 but you also need to send the response back and that will require its own static route. You will also need to summarize the routes between the core routers to make your routes as efficient as possible. Here is a helpful link https://www.davidc.net/sites/default/subnets/subnets.html

1. **Verifying Static Routes**
   - Verify the static routes configuration using the `show ip route` command on the router's CLI.
   - Ensure that the added static routes are displayed correctly in the routing table.
   - Ping the different IP addresses to make sure you can reach them. Consider trying to ping the router's interfaces as well if it helps.

   <div style="page-break-after: always"></div>

## Port Security

Port security is a feature commonly found in network switches that enables administrators to control which devices can connect to specific switch ports based on MAC addresses. It helps prevent unauthorized access to the network and protects against certain types of attacks, such as MAC address spoofing. You will need to add port security to:

- 6 PCs
- 1 SRV

### Configuring Port Security:

Here's a basic example of how to configure port security on a Cisco switch:

1. **Enable Port Security**:
   You will first have to enable the port security on the switch.
   ```
   enable
   configure terminal
   interface <interface name>
   switchport port-security
   ```

1. **Set Maximum Number of Allowed MAC Addresses**:
   The next step is to configure the maximum number of MAC addresses that are allowed to connect to a specific switch port when port security is enabled. For this lab will allow only 1 MAC address per port. We also need to set what will happen when that limit is reached. The three options are:

    - **shutdown**: The port is immediately disabled if a violation occurs.
    - **restrict**: Traffic from the violating MAC address is dropped, but the port remains enabled.
    - **protect**: Traffic from the violating MAC address is dropped, and no notification is sent. The port remains enabled.

   ```
   switchport port-security maximum <No of MAC addresses>
   switchport port-security violation <shutdown | restrict | protect>
   ```

1. **Specify Allowed MAC Addresses**:
   You now need to specify the MAC address of the device that is connected to the port. Make sure you use the MAC address of the PC or SRV and not the MAC address on the switch.
   ```
   switchport port-security mac-address <MAC address>
   ```

1. **Verify Configuration**:
    You can check your configuration with this command. If you are still in configuration mode remember to add `do` to the start of the command.
   ```
   show port-security interface <interface_id>
   ```
1. **Verify Configuration**:
    You can test the port security by:
        - going into the command prompt and pinging the device gateway which should still work. 
        - Change the MAC address of the device
        - Try pinging again and it should no longer work

### Important Notes:

- Port security is typically applied to access ports, not trunk ports.
- It's important to consider the number of devices that legitimately need to connect to the port when setting the maximum allowed MAC addresses.
- Regular monitoring and maintenance of port security configurations are necessary to ensure the security of the network and avoid unintended disruptions.

## Non routable VLANS

   - To make VLAN 12 unroutable, you need to ensure that the devices on this VLAN cannot communicate with devices on other VLANs through routing. This can be achieved by removing the Switch Virtual Interface (SVI) for VLAN 12 or by not configuring one in the first place. 

   <div style="page-break-after: always"></div>


## Write Up Questions

1. What are the notable benefits and drawbacks associated with the implementation of static routes in a network infrastructure?
1. How does Spanning Tree Protocol (STP) contribute to network stability and resilience, and what role does it play in preventing network loops?
1. Explain the concept of port security on network switches. What is its primary purpose?
1. What are the different violation modes available in port security? Describe each mode and explain how they handle security violations.

## TA Pass Off Requirements

- VLAN 12 can not access VLANs 10-11 and 13-16 (15 Points)
- VLAN 16 can not access VLANs 10-12 (15 Points)
- VLAN 11 can ping VLANs 10-11 (10 Points)
- VLAN 11 can ping VLANs 13-15 (10 Points)
- Static routes have all been summarized (10 Points)
- Shutting down the connection between Floor Switch 2 and Dist 1 still allows PC4 to reach its gateway (5 Points)
- Shutting down the connection between Floor Switch 1 and Dist 1 still allows PC1 to reach its gateway (5 Points)
- Shutting down the connection between Floor Switch 3 and Dist 2 still allows PC8 to reach its gateway (5 Points)
- Shutting down the connection between Floor Switch 4 and Dist 2 still allows PC12 to reach its gateway (5 Points)
- There are 6 PCs and 1 SRV with port security enabled (20 points):
   - There is only one MAC address allowed for each
   - A MAC address is set for each interface
   - The violation policy is set to shutdown, restrict, or protect for each interface
   - Changing the MAC address prevents the PC or server from accessing the network

## Lab Submission

- This lab will be passed off in person with a TA. The write-up questions must be submitted to Learning Suite as a PDF document.

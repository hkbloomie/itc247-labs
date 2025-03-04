# IT&C 247 - Lab 4

## Introduction 

In this lab, we'll be diving into enhancing network security and optimizing routing efficiency. We will build upon the foundational knowledge gained from previous labs and focus on strengthening security measures, port forwarding, network address translation, and implementing dynamic routing protocols within our network infrastructure.

## Objectives

- Implement password authentication and SSH access on network devices to ensure secure remote management and access control.
- Port forward traffic from a WAN IP and port to an internal system
- Configure OSPF (Open Shortest Path First) routing protocol to optimize routing efficiency and dynamic network updates.

<div style="page-break-after: always"></div>

## Network Tables

We will be working with the same base topology you created in labs 1-3 and adding to it. You'll need to add the third router for this lab. Ignore the firewalls in the diagram.

![](./Lab-4-Images/topology1.png)


These tables contain all of the network information you will need to set up the network for Lab 4. They are the same tables from the previous labs but with added information for this lab.

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

| VLAN | IP Range |
| - | - |
| 10 | 10.1.10.0/24 |
| 11 | 10.1.11.0/24 |   
| 12 | 10.1.12.0/24 |   
| 13 | 10.2.13.0/24 |   
| 14 | 10.2.14.0/24 |   
| 15 | 10.2.15.0/24 | 
| 16 | 10.2.16.0/24 |

<div style="page-break-after: always"></div>

| Device | Interface | IP Address | Area |
| - | - | - | - |
| Core-3 | loopback0            | 1.1.1.1/32     | 0 |
| Core-1 | GigabitEthernet0/0/0 | 192.168.1.1/24 | 1 |
| Core-2 | GigabitEthernet0/0/0 | 192.168.1.2/24 | 1 |
| Core-1 | GigabitEthernet0/0/1 | 192.168.2.1/24 | 1 |
| Core-3 | GigabitEthernet0/0/1 | 192.168.2.2/24 | 1 |
| Core-2 | GigabitEthernet0/0/1 | 192.168.3.1/24 | 1 |
| Core-3 | GigabitEthernet0/0/0 | 192.168.3.2/24 | 1 |

## OSPF

OSPF is a dynamic routing protocol used in computer networks to efficiently exchange routing information between routers. It is widely used in large-scale networks due to its scalability, fast convergence, and support for variable-length subnet masking (VLSM). OSPF operates within an autonomous system (AS) and calculates the shortest path to destination networks based on link-state information.

In OSPF (Open Shortest Path First), Area 0 (also known as the backbone area) is a fundamental component of the OSPF hierarchical network design. It serves as the central routing domain through which all other OSPF areas must connect. Here's an overview of Area 0 in OSPF:

- Note: You will need to remove the static routes from the last lab.

### Backbone Area (Area 0)

**Loopback Address**
Loopback interfaces are commonly used in routing protocols such as OSPF (Open Shortest Path First) and BGP (Border Gateway Protocol). They provide a stable and predictable interface for routing updates and neighbor relationships. Loopback interfaces are preferred for this purpose because they have inherent benefits such as always being up and not subject to physical link failures.

```
  interface loopback <loopback interface number>
  ip address <IP> <subnet mask>
```

### Example Configuration:

Here's an example of setting up a loopback interface with IP address 10.0.0.1/24:

```
interface loopback 0
ip address 10.0.0.1 255.255.255.0
```

### Main Area (Area 1)

1. **Enable IP Routing**: Activate the feature of IP routing on the router. This enables the router to forward packets between different networks by analyzing their destination IP addresses and determining the best path for delivery.

2. **Enable OSPF with a Process ID of 1**: Initiate the OSPF routing protocol on the router, assigning it a unique process ID of 1. The process ID helps differentiate OSPF instances if multiple OSPF processes are running on the same router.

3. **Add All of the Networks that the Router Needs in Area 0 or 1**:
   - **Core-3 is the Only One that Should Have Area 0**: Designate the core router named "core-3" as the backbone router by configuring it to be in OSPF Area 0. Area 0 is the backbone area of OSPF, and it must exist in every OSPF network.
   - **The Subnet Masks for OSPF Network Commands are Inverted**: Specify the networks that the router is connected to within the OSPF configuration. When using OSPF network commands, the subnet masks are represented in inverted form. For example, a subnet mask of /24, typically written as 255.255.255.0, becomes 0.0.0.255 in OSPF configuration. This ensures OSPF recognizes the correct network boundaries.
   
### OSPF Authentication

1. **Configure OSPF Authentication**

   To secure OSPF routing updates, authentication can be enabled within OSPF configurations. This ensures that routers only accept OSPF updates from trusted neighbors by verifying the authenticity of received OSPF packets.

   - **Enable OSPF Authentication and Specify the Authentication Type and Key**

     Authentication in OSPF is typically achieved by using a simple password known as the authentication key. This key is shared between OSPF neighbors and used to authenticate OSPF packets. These commands are run on the configuration of the physical port that connects the router together.

### Verify OSPF Configuration

  You can use these commands to verify the configurations you set up in the previous steps.
   ```
   Router# show ip ospf
   Router# show ip ospf interface
   Router# show ip ospf neighbor
   ```

<div style="page-break-after: always"></div>

## Passwords && SSH

Passwords and SSH (Secure Shell) play critical roles in securing access to Cisco devices such as routers and switches. They are essential components of network security and help protect sensitive information from unauthorized access and malicious attacks. You will need to set up a password and SSH access on 3 switches, and 1 router. In a real environment, you would set these up on all devices but for the purposes of this lab, it will just be a selection. SSH access should work from PC2 to all the devices you will set up SSH on, all of these devices will need to be on the left side of your network and in the 10.1.11.0/24 range.

### Passwords

Passwords are authentication credentials used to verify the identity of users accessing a Cisco device. They control access to various levels of privilege and resources within the device's operating system.

#### Types of Passwords

1. **Console Password**: Controls access to the device's console port, typically used for direct local access.
2. **Enable Password/Secret**: Controls access to privileged EXEC mode (enable mode), which grants administrative privileges for configuring the device.
3. **User Passwords**: Control access to user accounts configured on the device, allowing users to log in and access specific resources.

#### Best Practices for Passwords

- Use Strong Passwords: Choose passwords that are complex with a minimum length of 8 characters, combining uppercase and lowercase letters, numbers, and special characters.
- Regularly Rotate Passwords: Change passwords periodically to minimize the risk of unauthorized access.
- Avoid Default Passwords: Change default passwords immediately after device deployment to prevent unauthorized access.
- Use Encryption: Encrypt passwords stored in the device's configuration to prevent unauthorized users from viewing them.

### SSH (Secure Shell):

SSH is a cryptographic network protocol used to securely access and manage network devices remotely over an unsecured network. It provides a secure alternative to traditional Telnet, encrypting data transmitted between the client and the server.

#### Benefits of SSH

1. **Encryption**: Encrypts data transmitted between the client and the server, protecting it from eavesdropping and interception.
2. **Authentication**: Utilizes cryptographic keys and passwords to authenticate users, ensuring secure access to the device.
3. **Integrity**: Verifies the integrity of transmitted data, detecting any modifications or tampering attempts.

#### Configuring SSH

To enable SSH on a Cisco device, you need to generate RSA key pairs, configure the hostname, enable the SSH server, and configure user authentication.

#### Best Practices for SSH

- Use SSH Version 2: SSH version 2 offers stronger encryption and better security compared to SSH version 1.
- Limit Access: Restrict SSH access to authorized users and IP addresses using access control lists (ACLs) or firewall rules.
- Disable Unnecessary Services: Disable unused services such as Telnet to reduce the attack surface and enhance security.

### Important Notes

- It's recommended to use the enable secret command as it encrypts the password.
- If both the enable password and enable secret are configured, the enable secret takes precedence.
- Ensure to keep the password secure and avoid using easily guessable passwords.
- Always remember to save the configuration after making changes:

<div style="page-break-after: always"></div>


## Port forwarding

### Configuring Port Forwarding

1. **Identifying Internal Services**
   - Determine which internal services need to be accessible from the external network. This could include services like web servers, email servers, or FTP servers. In this case, it will be the web server.

1. **Configuring Port Forwarding**
   - Use the router's CLI to configure port forwarding using the `ip nat` command, in this case, it will be core-2. 
   - A few things to consider/work out:
        - What interface will act as the inside section of the NAT process?
        - What interface will act as the outside section of the NAT process?
        - What IP address and port needs to be forwarded to what port and IP address?
   - This command is one you will need to complete the port forwarding process.
    ```
    ip nat inside source static <tcp/udp> <Local IP address> <Local port> <Global IP Address> <Global Port>
    ```

1. **Verifying Port Forwarding:**
   - Verify the port forwarding configuration using the `show ip nat translations` command on the router's CLI.
   - Ensure that the configured port forwarding rules are correctly translating external requests to internal IP addresses and ports.

<div style="page-break-after: always"></div>

## Write Up Questions

1. What are the different types of passwords utilized in Cisco switches, and how do they contribute to access control and device security?
1. Why is SSH often favored over protocols like Telnet for remote system connections, and what advantages does it offer in terms of security and functionality?
1. What is the purpose of OSPF, and what are some advantages and disadvantages associated with its implementation in network routing?
1. How do stateful firewalls, packet filtering firewalls, and web application firewalls differ in their functionalities, and what roles do they play in network security strategies?
1. Where in the network architecture from this lab is an ideal placement for firewalls, and what are the reasons behind this positioning?

## TA Pass Off Requirements

- Passwords & SSH (40 Points)
    - 3 switches and 1 router all require a password to enter exec mode. (10 Points)
    - The set exec password has a minimum length of 8 characters, combining uppercase and lowercase letters, numbers, and special characters. ( 10 Points)
    - 3 switches and 1 router can be accessed via ssh from PC2(10.1.11.2)  (10 Points)
    - The SSH password has a minimum length of 8 characters, combining uppercase and lowercase letters, numbers, and special characters.  (5 Points)
    - A strong encryption algorithm has been use for the SSH connection  (5 Points)
- OSPF (50 Points)
  - Core-3 has the loopback address set for area 0  (10 Points)
  - All 3 routers have been set up to use area 1 for OSPF with the correct networks  (10 Points)
  - All 3 routers have passwords set for OSPF authentication  (10 Points)
  - Disabling the connection between any 2 core routers still allows the network to function (10 Points)
  - Any PC in VLAN 10-11 & 13-15 can ping each other. (10 Points)
- Port Forwarding (10 Points)
   - Any PC in the 10.1.0.0/16 range can access the website through a WAN IP of core 2

## Lab Submission

- This lab will be passed off in person with a TA. The write-up questions must be submitted to Learning Suite as a PDF document.

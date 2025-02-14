# Configuring OpenWrt to work with Japan SoftBank hikari IPv6 high-speed internet "IPv4 over IPv6" service

## The Brackground 

SoftBank requires renting an ISP-specific router, the "BB Unit," to enable IPv6 IPoE for stable Internet access and to avoid PPPoE congestion during peak hours. However, the BB Unit has significant performance and firmware limitations, the most critical being its lack of NAT loopback support. This means that accessing services bound to domain names within the local network requires manually mapping the domain to the LAN IP address via the hosts file.

![img-bbunit](https://github.com/user-attachments/assets/b9adf782-2414-46d1-988a-91a92888ea7e)

Connecting a standard router directly to the ONU without the BB Unit allows access to the IPv6 Internet via IPoE, but IPv4 remains restricted to PPPoE, which suffers from instability due to NTT’s legacy infrastructure. Currently, no commercially available routers in Japan natively support SoftBank's 4in6 tunneling, and online resources for configuring IPoE IPv6 + IPv4 are scarce. Most guides suggest enabling DMZ, allowing IPv6 passthrough, and disabling Wi-Fi transmission to achieve a bridge-like setup.

However, this approach is not an ideal solution for obtaining a stable IPv4 connection with a fixed IP address. After extensive research, I successfully configured high-speed IPv6 on OpenWrt. This article outlines the configuration process in hopes of assisting other SoftBank users facing similar challenges.

---
# SoftBank IPv4 over IPv6 Configuration Guide  

## Preparation  

To establish a connection using IPv4 over IPv6, you will need the following:  

- **Local IPv6 address**: `2400:0000:0000:a000:1111:1111:1111:1111`  
- **Local IPv4 address**: `126.000.00.000`  
- **Remote IPv6 address (Peer)**: `2400:2000:0:0:a000::00`  

## Accessing SoftBank BB Unit  

You can easily find your **local IPv6 and IPv4 addresses** after connecting the SoftBank BB Unit to the Internet.  

### Step 1: Open the Router Admin Page  
In your web browser, go to one of the following URLs:  

- [http://172.16.255.254/index.html](http://172.16.255.254/index.html)  
- [http://192.168.3.1](http://192.168.3.1)  

### Step 2: Login Credentials  
Use the following login details to access the SoftBank BB router:  

- **Username**: `user`  
- **Password**: `user`  

### Example Interface  
![img-01](https://github.com/user-attachments/assets/25773fef-bc81-4a80-97a3-44da659730a0)  

Record these two addresses for future use.

## Getting the Remote IPv6 Address (Peer)  

Obtaining the **remote IPv6 address (peer)** can be challenging, as SoftBank Hikari customer support does not provide this information. However, you can determine the peer address by capturing network packets using **Wireshark**.  

### Steps to Capture the Peer IPv6 Address  

1. **Connect your PC to the ONU**  
   - Use an Ethernet cable to connect the **ONU LAN port** directly to your Windows PC.  

2. **Start Packet Capture in Wireshark**  
   - Open **Wireshark** and select the network adapter corresponding to your **PC’s LAN connection**.  
   - Click **Start** to begin capturing packets.  

3. **Stop and Analyze the Captured Packets**  
   - Let the capture run for a while, then click **Stop**.  
   - In the **Destination** column, look for packets sent to your previously identified **IPv4 address** (retrieved from the SoftBank BB router).  

4. **Find the Peer IPv6 Address**  
   - Click on the relevant packet.  
   - Inside the packet details, locate **Internet Protocol Version 6 (IPv6)**.  
   - Find the fields labeled **"Src"** (Source) and **"Dst"** (Destination).  
   - The **"Src" (Source)** address is your peer IPv6 address—**note it down** for later use.  

### Example:  
![final](https://github.com/user-attachments/assets/1af7abd8-213c-4bc3-9246-99beb25a16d5)  

## Configure OpenWrt
To configure OpenWRT router you need to install flowing package to install `kmod-ip6-tunnel` `ds-lite` `ip-full`

--
  
For a detailed step-by-step guide, refer to the full documentation in this repository.  

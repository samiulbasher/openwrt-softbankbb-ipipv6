# Configuring OpenWrt to work with Japan SoftBank Hikari IPv6 high-speed internet ("IPv4 over IPv6") service

## The Brackground 

SoftBank requires renting an ISP-specific router, the "BB Unit," to enable IPv6 IPoE for stable Internet access and to avoid PPPoE congestion during peak hours. However, the BB Unit has significant performance and firmware limitations, the most critical being its lack of NAT loopback support. This means that accessing services bound to domain names within the local network requires manually mapping the domain to the LAN IP address via the hosts file.

![img-bbunit](https://github.com/user-attachments/assets/b9adf782-2414-46d1-988a-91a92888ea7e)

Connecting a standard router directly to the ONU without the BB Unit allows access to the IPv6 Internet via IPoE, but IPv4 remains restricted to PPPoE, which suffers from instability due to NTT’s legacy infrastructure. Currently, no commercially available routers in Japan natively support SoftBank's 4in6 tunneling, and online resources for configuring IPoE IPv6 + IPv4 are scarce. Most guides suggest enabling DMZ, allowing IPv6 passthrough, and disabling Wi-Fi transmission to achieve a bridge-like setup.

However, this approach is not an ideal solution for obtaining a stable IPv4 connection with a fixed IP address. After extensive research, I successfully configured high-speed IPv6 on OpenWrt. This article outlines the configuration process in hopes of assisting other SoftBank users facing similar challenges.

For a detailed step-by-step guide, refer to the full documentation in this repository.  

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
![final](https://github.com/user-attachments/assets/bdd7926e-1314-4bd7-a1f0-c562c4b8abbb)

## Configuring OpenWrt  

To set up your OpenWrt router, you need to install the following packages:  

- `kmod-ip6-tunnel`  
- `ds-lite`  
- `ip-full`  
- `luci-app-ttyd` (optional, but useful for enabling an SSH terminal in the LuCI web interface)  

### Installation Steps  

1. **Access the OpenWrt Web Interface (LuCI)**  
   - Open the web interface ([http://192.168.1.1](http://192.168.1.1) or your default gateway IP) and navigate to **System > Software**.  

2. **Update the Package List**  
   - Click the **Update list** button to refresh the available packages.  

3. **Install Required Packages**  
   - Search for each of the packages listed above and install them one by one.  

4. **Reboot the Router**  
   - After installation, restart your router to apply the changes.  

Installing `luci-app-ttyd` is optional but highly recommended, as it allows you to access the SSH terminal directly from LuCI, making configuration easier.  

## Changing the WAN `MAC` Address  

To ensure proper connectivity, you need to change your OpenWrt router's **WAN MAC address** to match the **SoftBank BB Unit's MAC address**.  

### Steps to Change the WAN MAC Address  

1. **Find the SoftBank BB MAC Address**  
   - Access your **SoftBank BB Unit** and locate the **MAC address** (usually found in the router's label).  
   - **Note it down** for later use.  

2. **Update the MAC Address in OpenWrt**  
   - Navigate to **Network > Device** in the OpenWrt web interface.  
   - Find and **Configure** the WAN interface.
     
![MAC Address Config 1](https://github.com/user-attachments/assets/7591fdb6-e7a6-4812-b239-54e4c8e0b427)  

   - In the **MAC address** section, enter the **SoftBank BB Unit’s MAC address**.  
   - Save the changes and apply the configuration.
     
![MAC Address Config 2](https://github.com/user-attachments/assets/4f5d6d52-e391-4f28-bdf3-ba7e47a4feed)  
                                                               
---
 
## Configuring WAN6 (IPv6 Pass-Through)  

**Navigate to WAN6 Settings**  Go to **Network > Interfaces** and click **Edit** on the `wan6` interface.  

1. **Enable DHCP Server**  
   - Open the **DHCP Server** tab.  
   - If it's not active, enable **Set up DHCP Server**.  

2. **General Setup**  
   - Check **Ignore interface**.  

3. **IPv6 Settings**  
   - Go to the **IPv6 Settings** tab.  
   - Check **Designated master**.  
   - Set **Relay mode** to all options.  

   ![WAN6 Config](https://github.com/user-attachments/assets/992cf2ed-8191-4de4-a4ba-ef77aa7e554b)  

4. **Firewall Settings**  
   - Go to the **Firewall Settings** tab and set the **wan** zone.  

   ![WAN Firewall](https://github.com/user-attachments/assets/f6357ba3-93dd-4b47-8678-7e8f778c724e)  

---

## Configuring LAN  

**Navigate to LAN Settings** Go to **Network > Interfaces** and click **Edit** on the `lan` interface.  

1. **General Setup**  
   - **Uncheck** **Ignore interface**.  

2. **Advanced Settings**  
   - Set the DNS servers to be used by DHCP under **DHCP-Options**:  
     ```
     6,8.8.8.8,1.1.1.1
     ```  

   ![LAN DHCP Options](https://github.com/user-attachments/assets/9f3c4fcb-0cef-4589-857c-16425a574aee)  

3. **IPv6 Settings**  
   - Go to the **IPv6 Settings** tab.  
   - **Uncheck** **Designated master**.  
   - Set **DHCPv6 Service** to **Server mode**.  
   - Everything else should be set to **Relay mode**.  
   - In **Announced IPv6 DNS servers**, enter the public IPv6 DNS addresses:  

     ```
     2001:4860:4860::8888  # Google
     2606:4700:4700::1111  # Cloudflare
     ```  

   - Do not check **NDP-Proxy slave**.  

   ![LAN IPv6 Settings](https://github.com/user-attachments/assets/0b752448-eb08-4bcd-ae52-e9c63f6de4b8)  

4. **Firewall Settings**  
   - Go to the **Firewall Settings** tab and set the **lan** zone.  

   ![LAN Firewall](https://github.com/user-attachments/assets/152f1fad-06b5-4657-a8b4-7ca4efa94bba)

---

## Configuring WAN  

Follow these steps to configure the **WAN** settings in OpenWrt.  

1. **Navigate to WAN Interface**  
   - Go to **Network > Interfaces** in the OpenWrt web interface.  
   - Click **Edit** on the `wan` interface.  

2. **General Settings**
   - Ensure the correct **protocol** is selected for your WAN connection.  
   - Enter the **IPv4 address** that you found earlier and also add **IPv4 netmask**.  

![Step 1](https://github.com/user-attachments/assets/8066839c-c917-4c37-8865-ff80c6afe674)  

3. **Advanced Settings**
   - Add a **custom DNS server**. Here, Google’s DNS is used.  
   - Leave other settings as **default**.  

![Step 2](https://github.com/user-attachments/assets/78e47432-8b8d-4471-9f7d-c4bd2a253b82)  

4. **Firewall Settings**
   - Assign the **WAN** interface to the appropriate **firewall zone**.  
   - This ensures proper routing and security for inbound and outbound traffic.  

![Step 3](https://github.com/user-attachments/assets/840a885b-d5db-47bf-aae0-11b2848a0527)  

5. DHCP Server  
   - **Uncheck** **Designated master**.  
   - Set all options to **disabled**.  

![Step 4](https://github.com/user-attachments/assets/caa82f5d-6955-4923-b702-d4ba8029a020)  

---

## Creating and Configuring IPIPv6  

Follow these steps to set up **IPIPv6** on OpenWrt.  

### 1. Create a New Interface  
- Navigate to **Network > Interfaces** in the OpenWrt web interface.  
- Click **Add new interface** Name **v6tun0** and select **IPv4 over IPv6**.  

![Step 1](https://github.com/user-attachments/assets/16304545-3216-423b-a997-351bdaa75540)  

### 2. Configure the IPIPv6 Tunnel  
- Enter the **Remote IPv6 Address(peer), Local IPv4 Address and Local IPv6 Address** that you found earlier.  
- Click **Save & Apply**.  

![Step 2](https://github.com/user-attachments/assets/b082f736-c341-4fd5-afa4-dfca7b29b169)  

### 3. Advanced Settings and Adjust Firewall Settings  
- Keep most **Advanced Settings** at their **default** values.
- Assign the new **IPIPv6 interface** to the correct firewall zone.  

![Step 3](https://github.com/user-attachments/assets/19862950-87b3-4d32-afbf-fbd0b0910ba7)  
![Step 4](https://github.com/user-attachments/assets/1adfb34a-79d6-41f8-a878-5b3f9bf0513a)  

### 4. Add a startup script  
- This script configures an **IPv6-in-IPv6 (IPIPv6) tunnel** to enable **IPv4 over IPv6** connectivity.
1. **Access the OpenWrt Terminal**  
   You can add the script using either the **Web Interface** or **SSH**.  

   - **Web Interface**:  
     - Navigate to **Services > Terminal** in OpenWrt.  
     - *(Requires the installation of the `luci-app-ttyd` package)*.  
     ![Step 5](https://github.com/user-attachments/assets/7fffc8a4-4769-4449-bf41-41591199d329)  

   - **SSH**: *(Only required if using SSH; if using the Web Interface, this is not necessary)*  
     - Connect to the router via SSH:  
       ```ssh root@your-default-ip ```
     - Enter your root password.

2. **Add the Startup Script**  
   - Open the startup script file using `vi` or `nano`:  
     ```sh
     vi /etc/rc.local
     ```
     *(Some systems may use `/etc/rc.d/rc.local`)*.  

   - Replace `your_peer_address`, `your_ipv4_address`, and `your_ipv6_address` with the respective values and then paste the following script **before** `exit 0`:  
     ```sh
     INTERFACE=wan
     IPV6_PEER="your_peer_address"
     IPV4_ENDPOINT="your_ipv4_address"
     IPV6_ENDPOINT="your_ipv6_address"
     
     ip -6 addr add $IPV6_ENDPOINT dev $INTERFACE
     ip -6 tunnel add v6tun0 mode ipip6 remote $IPV6_PEER local $IPV6_ENDPOINT dev $INTERFACE encaplimit none
     ip addr add $IPV4_ENDPOINT dev v6tun0
     ip route add default dev v6tun0
     ip link set dev v6tun0 up
     ```

     ![Step 6](https://github.com/user-attachments/assets/43e789ff-8787-46cc-84de-93e95c6b9016)  

   - Reboot the router if necessary to apply all changes.  
 

Your **IPIPv6 tunnel** is now successfully configured! 🚀 

## All Interface
![Screenshot 2025-02-16 at 11 49 52 AM](https://github.com/user-attachments/assets/38b8f500-14b3-4f05-a968-dc801ff40eca)
## Confirm that the configuration is normal
![Screenshot 2025-02-16 at 11 50 36 AM](https://github.com/user-attachments/assets/a224727f-9f2e-4ef3-8e83-1735c3f6cd1c)

## Check connection[SUCCESS!!]
![checkConnction](https://github.com/user-attachments/assets/fcb78a1e-0ca1-4682-8321-dc847960c26e)



## Testing tool 
* https://ip.sb/
* https://test-ipv6.com/index.html.ja_JP

## Acknowledgments
Inspiration, code snippets, etc.
* https://wp.hima-jin.info/openwrt-ds-lite/
* https://layer3.info/softbankhikari-bbunit-less/
* https://akashisn.hatenablog.com/entry/2022/03/19/152726
* https://qiita.com/kouhei-ioroi/items/cf0c6228c5c1faef415a
* https://blog.missing233.com/2023/09/16/softbank-hikari-openwrt-configuration/

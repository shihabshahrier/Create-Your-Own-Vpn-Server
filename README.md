# Create Your Own VPN Server Using Google Cloud Console

This guide will walk you through creating a **VPN server** using **Google Cloud Console** (GCP). We'll be setting up a **WireGuard** VPN server, which is fast, secure, and easy to configure.

---

## Prerequisites

- A **Google Cloud** account with **$300 free credits** (or an existing Google Cloud account)
- Basic knowledge of terminal commands
- A **valid credit card** for account verification

---

## Steps

### 1. Sign Up for Google Cloud Free Tier

1. Visit the [Google Cloud Free Tier](https://cloud.google.com/free) page.
2. Sign up for the free tier and get **$300 credits** for 90 days.
3. Provide your credit card details for verification (you will not be charged if you stay within free limits).

### 2. Create a Virtual Machine (VM)

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **Compute Engine > VM Instances**.
3. Click **Create Instance**.
   - **Name**: `vpn-server`
   - **Region**: Choose a region near you (e.g., `us-central1`).
   - **Machine type**: Select the **`e2-micro`** machine type (this is part of the **Always Free Tier**).
   - **Boot disk**: Select **Ubuntu 22.04 LTS**.
4. Scroll to **Networking**, click **Add network interface** and select your default VPC.
5. **Add tags**: `wireguard-vpn` (for easier firewall management).
6. Click **Create**.

---

### 3. Open Firewall Ports

1. Go to **VPC Network > Firewall rules**.
2. Click **Create Firewall Rule**:
   - **Name**: `allow-wireguard`
   - **Targets**: `Specified target tags`
   - **Target tags**: `wireguard-vpn`
   - **Source IP ranges**: `0.0.0.0/0`
   - **Protocols and ports**: **UDP 51820**
3. Save the rule.

---

### 4. SSH into Your VM

Once the VM is created:

1. Go to **Compute Engine > VM Instances**.
2. Click **SSH** to open a terminal window on the VM.

---

### 5. Install and Configure WireGuard

1. Update your VM:
   ```bash
   sudo apt update
   sudo apt upgrade -y
````

2. Install **WireGuard**:

   ```bash
   sudo apt install -y wireguard
   ```
3. Create the WireGuard configuration script:

   ```bash
   curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
   chmod +x wireguard-install.sh
   sudo ./wireguard-install.sh
   ```
4. Follow the prompts:

   * **VPN Interface name**: `wg0` (default)
   * **Port**: `51820` (default)
   * **VPN subnet**: Use the default (10.66.66.0/24)
   * **DNS**: Use the default or `1.1.1.1`
5. Once completed, **WireGuard** will generate a `.conf` file for your client.

---

### 6. Configure Your Client (Phone/PC)

1. Install the **WireGuard** app on your phone or computer:

   * [WireGuard for Android](https://play.google.com/store/apps/details?id=com.wireguard.android)
   * [WireGuard for Windows](https://www.wireguard.com/install/)
2. Import the generated `.conf` file or use the provided QR code in the WireGuard app.
3. **Modify the `Endpoint`** in the `.conf` file:

   * Replace `10.128.0.3:51820` with your **Google Cloud public IP address** (found in the **VM instances** dashboard).
   * Example:

     ```ini
     Endpoint = <your-server-ip>:51820
     ```
4. **Allow all traffic through the VPN**:

   * Ensure your `AllowedIPs` section looks like this:

     ```ini
     AllowedIPs = 0.0.0.0/0, ::/0
     ```
5. Save and activate the VPN.

---

### 7. Check VPN Connection

1. After connecting to the VPN, open a browser and visit: [https://whatismyipaddress.com/](https://whatismyipaddress.com/)

   * It should show your **Google Cloud IP**.
2. Run `sudo wg` on the server to check for data transfer and connection status.

---

### 8. Enable IP Forwarding (If Needed)

If the internet is not working after connecting, ensure **IP forwarding** is enabled:

1. Run the following:

   ```bash
   echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   sudo iptables -t nat -A POSTROUTING -o ens4 -s 10.66.66.0/24 -j MASQUERADE
   ```

   Replace `ens4` with the correct network interface name if needed.

---

## Conclusion

You now have a **secure, personal VPN server** running on Google Cloud!

* Your server is protected by **WireGuard**, which provides high performance and strong encryption.
* You can use the VPN on your phone or PC to **secure your internet connection**.

---

### Troubleshooting

If your connection doesn’t work:

* Check that your **public IP** is correctly configured in the client config.
* Ensure **NAT** and **IP forwarding** are properly configured on the server.
* Verify that your **firewall rule** is applied correctly.


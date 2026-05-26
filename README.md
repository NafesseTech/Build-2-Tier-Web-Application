# Build-2-Tier-Web-Application


In this lab, we'll build a classic "IaaS" architecture. We'll create a Virtual Network with two subnets: a "Public" subnet for a Web Server and a "Private" subnet for a Database Server. We'll configure Network Security Groups (NSGs) to ensure the Database Server is protected from the internet and can only be accessed by the Web Server.





# Lab 02: Building a Secure 2-Tier Web Application




## 1. Architecture Diagram

*(Imagine a diagram with two boxes: Top box is "Public Subnet" containing vm-web. Bottom box is "Private Subnet" containing vm-db. An arrow connects Web to DB. The Internet connects only to Web.)*



## 2. Prerequisites

- [ ] Active Azure Subscription
- [ ] Completed Week 2 Video Modules
- [ ] Terminal installed



## 3. Lab Variables (Naming Convention)

| Resource | Name |
|---|---|
| Resource Group | rg-lab02-[yourname] |
| VNet Name | vnet-lab02 |
| Subnet 1 (Public) | snet-web (10.0.1.0/24) |
| Subnet 2 (Private) | snet-db (10.0.2.0/24) |
| Web VM | vm-web-01 |
| Database VM | vm-db-01 |



## 4. Step-by-Step Instructions

### Phase 1: The Network Foundation

1. Search for **Virtual Networks** -> **Create**.
2. **Basics:**
   - Resource Group: Create New -> `rg-lab02-[yourname]`
   - Name: `vnet-lab02`
   - Region: `East US`
3. **IP Addresses:**
   - Space: `10.0.0.0/16`
   - Subnet 1: Name: `snet-web`, Range: `10.0.1.0/24`
   - Subnet 2: Name: `snet-db`, Range: `10.0.2.0/24`
4. Click **Review + create** -> **Create**.



### Phase 2: Deploying the Web Server (The Front End)

1. Search for **Virtual Machines** -> **Create**.
2. **Basics:**
   - Resource Group: `rg-lab02-[yourname]`
   - Name: `vm-web-01`
   - Region: `East US`
   - Image: `Ubuntu Server 20.04 LTS`
   - Size: `Standard_B1s`
   - Key pair name: `key-lab02`
   - Public inbound ports: Allow selected -> HTTP (80) and SSH (22).
3. **Networking:**
   - Subnet: Ensure `snet-web` is selected.
   - Public IP: Create New (Standard).
4. Click **Review + create** -> **Create**.
5. Download the Private Key (`.pem`) if prompted.



### Phase 3: Deploying the Database Server (The Back End)

Now we deploy the server that represents our database.

1. Create another **Virtual Machine**.
2. **Basics:**
   - Resource Group: `rg-lab02-[yourname]`
   - Name: `vm-db-01`
   - Region: `East US`
   - Image: `Ubuntu Server 20.04 LTS`
   - Size: `Standard_B1s`
   - Key pair name: Use existing key stored in Azure -> `key-lab02` (Or create a new one if you lost it).
   - Public inbound ports: Allow selected -> SSH (22).
3. **Networking (CRITICAL STEP):**
   - Virtual Network: `vnet-lab02`
   - Subnet: Change this to `snet-db`.
   - Public IP: Select **None**. *(This is a private server; it should not have a public IP).*
4. Click **Review + create** -> **Create**.



### Phase 4: Validating Connectivity (The "Jump")

Since `vm-db-01` has no public IP, you cannot connect to it directly from your home computer. You must connect to the Web Server first, and "jump" to the DB server.

1. **Get the Private IP of the DB Server:**
   - Go to the `vm-db-01` resource. Look for "Private IP address". It should be `10.0.2.4`. Copy this.

2. **SSH into the Web Server:**
   - Open your terminal and SSH into `vm-web-01` using its Public IP (just like Phase 2).
   ```bash
   ssh -i key-lab02.pem azureuser@[public-ip-of-web]
   ```

3. **Test Connectivity:**
   - From inside the Web Server, try to ping the Database Server:
   ```bash
   ping 10.0.2.4
   ```
   - You should see replies! This proves the two servers are connected inside the VNet.
   - Press `Ctrl + C` to stop the ping.



### Phase 5: Configuring the Firewall (NSG)

Right now, `vm-db-01` is accepting traffic. We want to ensure ONLY the Web Subnet can talk to it.

1. Go to the **Networking** tab of `vm-db-01`.
2. Click on the **Network Security Group** (it will have a random name like `vm-db-01-nsg`).
3. Click **Inbound security rules** -> **+ Add**.
4. **Create the "Allow Web" Rule:**
   - Source: `IP Addresses`
   - Source IP addresses/CIDR ranges: `10.0.1.0/24` *(This is the Web Subnet range).*
   - Source port ranges: `*`
   - Destination: `Any`
   - Service: `Custom`
   - Destination port ranges: `*` *(Or 3306/5432 if we installed a DB).*
   - Action: `Allow`
   - Priority: `100`
   - Name: `Allow-Web-Subnet`
5. Click **Add**.

> **Note:** Because we have no Public IP on this VM, internet traffic is already blocked, but this rule explicitly allows our internal traffic.



## 6. Troubleshooting

**Issue: "Ping fails."**
- Fix: Check if you deployed `vm-db-01` into the correct subnet (`snet-db`). Check if both VMs are in the same VNet.

**Issue: "I can't SSH into vm-db-01."**
- Reason: You cannot SSH directly from your home computer because there is no Public IP. You must SSH into `vm-web-01` first. *(To SSH from Web to DB, you would need to copy your `.pem` key to the Web server, which is an advanced topic for later).*



## 7. Clean Up

Delete `rg-lab02-[yourname]`.









ALL DONE!!!
















# How to Connect to Your Azure VM Using PuTTY

## Prerequisites
- PuTTY and PuTTYgen installed ([download from putty.org](https://www.putty.org))
- Your `.pem` private key file (from Azure)
- Your VM's **Public IP address** (found in Azure Portal → Your VM → Overview)
- Your **admin username** (e.g. `azureuser`)

---

## Step 1: Convert Your .pem Key to .ppk Using PuTTYgen

1. Open **PuTTYgen** from the Start menu
2. Click **Load**
3. In the file browser, change the filter (bottom-right dropdown) from `PuTTY Private Key Files (*.ppk)` to **All Files (*.*)**
4. Navigate to your `.pem` file and select it, then click **Open**
5. Click **OK** on the success message
6. Click **Save private key**
7. When asked *"Save without a passphrase?"*, click **Yes**
8. Save the file somewhere easy to find (e.g. Desktop) as `key-lab02.ppk`

---

## Step 2: Configure PuTTY

1. Open **PuTTY**
2. In the left panel, go to **Connection → SSH → Auth → Credentials**
3. Under **"Private key file for authentication"**, click **Browse**
4. Select the `.ppk` file you just created
5. In the left panel, click **Session**
6. Under **"Host Name (or IP address)"**, enter your VM's **Public IP address** (e.g. `172.208.68.241`)
7. Make sure:
   - **Port** is set to `22`
   - **Connection type** is set to `SSH`
8. Under **Saved Sessions**, give your session a name (e.g. `MyAzureVM`)
9. Click **Save**

---

## Step 3: Connect to Your VM

1. Click **Open**
2. If prompted with a security alert, click **Accept**
3. At the `login as:` prompt, type your admin username:
   ```
   azureuser
   ```
4. Press **Enter**

If the key is set up correctly, you will be logged in and see a prompt like:
```
azureuser@my-vm:~$
```

You are now connected to your Azure VM!

---

## Troubleshooting

| Error | Solution |
|---|---|
| `No supported authentication methods available` | Your `.ppk` key is not loaded in PuTTY. Go to **Connection → SSH → Auth → Credentials** and browse for your `.ppk` file |
| `Identity file not accessible: No such file or directory` | The key file path is wrong or the file is missing. Make sure your `.ppk` file exists and the correct path is entered |
| `Permission denied (publickey)` | Wrong username or wrong key file. Double-check both in Azure Portal |
| `Connection timed out` | Azure firewall/NSG may be blocking port 22. Check your VM's **Networking** settings in Azure Portal |

---

## Alternative: Connect Using Command Line (SSH)

If you prefer using the terminal instead of PuTTY:

1. Open **Command Prompt** or **PowerShell**
2. Run the following command:
   ```
   ssh -i "C:\Users\YourUsername\Downloads\key-lab02.pem" azureuser@172.208.68.241
   ```
   > Replace `YourUsername` with your Windows username (run `whoami` to find it)
3. If prompted *"Are you sure you want to continue connecting?"*, type `yes` and press **Enter**
4. You are now connected!

---

*Tip: To find your Windows username, open Command Prompt and type `whoami`. Use the name after the backslash `\`.*

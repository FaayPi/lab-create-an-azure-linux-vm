<!-- # LAB: Creating and Connecting to an Azure Linux Virtual Machine -->

<!-- # Lab: Create and Connect to an Azure Linux Virtual Machine -->

In this lab, you will learn how to **create a Linux Virtual Machine (VM)** on **Microsoft Azure** and **connect** to it using **SSH**. By the end of this tutorial, you'll have a running VM where you can deploy applications, experiment with configurations, or simply explore the Linux environment in the cloud. *Get ready to set up your Azure VM and enjoy some confetti for a successful connection!*


## Prerequisites

Before you begin, ensure you have the following:

1. **Azure Account**:  
   - If you don't have one, [sign up for a free Azure account](https://azure.microsoft.com/free/) which offers credits to get you started.

2. **Azure Portal Access**:  
   - Familiarity with navigating the [Azure Portal](https://portal.azure.com/).

3. **SSH Client**:  
   - **Windows**: Use [PuTTY](https://www.putty.org/) or the built-in **OpenSSH** client in PowerShell.
   - **macOS/Linux**: The terminal typically includes SSH by default.

4. **SSH Key Pair** (Optional but Recommended):  
   - If you don't have an SSH key pair, you can generate one. This enhances security by using key-based authentication instead of passwords.

   ```bash
   # Generate SSH keys on macOS/Linux
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/azure_vm_key
   ```

   - **Windows** users can use PuTTYgen or PowerShell to generate keys.


## Step 1: Create an Azure Linux Virtual Machine

You can create an Azure VM using the **Azure Portal** or the **Azure CLI**. This lab will guide you through both methods.

### Option A: Using the Azure Portal

1. **Sign In to Azure Portal**:
   - Navigate to [https://portal.azure.com](https://portal.azure.com) and sign in with your Azure account credentials.

2. **Navigate to Virtual Machines**:
   - In the left-hand menu, click on **"Virtual machines"**.
   - Click on the **"Create"** button and select **"Azure virtual machine"**.

3. **Configure Basic Settings**:

   - **Subscription**: Choose your Azure subscription.
   - **Resource Group**:  
     - Select an existing resource group or click **"Create new"** and enter a name (e.g., `MyResourceGroup`).
   - **Virtual Machine Name**: Enter a name for your VM (e.g., `MyLinuxVM`).
   - **Region**: Choose a region close to you or your users (e.g., `East US`).
   - **Availability Options**:  
     - For simplicity, select **"No infrastructure redundancy required"**.
   - **Image**:  
     - Choose a Linux distribution (e.g., **Ubuntu Server 20.04 LTS**).
   - **Size**:  
     - Click **"See all sizes"** to choose a VM size. For beginners, **Standard B1s** is cost-effective.
   - **Authentication Type**:  
     - Select **"SSH public key"** for key-based authentication.
   - **Username**: Enter a username (e.g., `azureuser`).
   - **SSH Public Key**:  
     - Paste your **public SSH key** (e.g., contents of `~/.ssh/azure_vm_key.pub`).
   - **Inbound Port Rules**:  
     - Ensure **SSH (port 22)** is allowed. By default, **"Allow selected ports"** is chosen with SSH enabled.
     - *Optional (for Web Server Testing)*: To test HTTP traffic (e.g., for NGINX), select “HTTP” or port 80 here as well.

4. **Configure Disks**:
   - **OS Disk Type**:  
     - **Standard SSD** is a good balance between cost and performance.
   - Click **"Next: Disks >"** to proceed.

5. **Configure Networking**:
   - **Virtual Network (VNet)**:  
     - Use the default VNet or create a new one.
   - **Subnet**:  
     - Use the default subnet or create a new one.
   - **Public IP**:  
     - Ensure a public IP is assigned to allow SSH (and HTTP, if needed) access.
   - **NIC Network Security Group**:  
     - Confirm that **SSH (port 22)** is allowed. If you plan on using a web server, also allow **HTTP (port 80)**.
   - Click **"Next: Management >"** to proceed.

6. **Management, Monitoring, and Advanced Settings**:
   - For this lab, you can accept the default settings.
   - Click **"Review + create"**.

7. **Review and Create**:
   - Azure will validate your configuration. If all checks pass, click **"Create"**.
   - Deployment may take a few minutes. You can monitor progress in the **"Notifications"** panel.

8. **Access Your VM's Public IP**:
   - Once deployed, navigate to **"Virtual machines"** > **Your VM (`MyLinuxVM`)**.
   - Locate the **"Public IP address"** on the VM's overview page.

### Option B: Using the Azure CLI

1. **Install Azure CLI** (if not already installed):
   - Follow the [official installation guide](https://docs.microsoft.com/cli/azure/install-azure-cli) for your operating system.

2. **Sign In to Azure**:
   ```bash
   az login
   ```
   - Follow the prompts to authenticate.

3. **Generate SSH Key Pair** (if you haven't already):
   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/azure_vm_key
   ```
   - This creates `azure_vm_key` (private key) and `azure_vm_key.pub` (public key).

4. **Create a Resource Group**:
   ```bash
   az group create --name MyResourceGroup --location eastus
   ```

5. **Create the Virtual Machine**:
   ```bash
   az vm create \
     --resource-group MyResourceGroup \
     --name MyLinuxVM \
     --image UbuntuLTS \
     --admin-username azureuser \
     --ssh-key-values ~/.ssh/azure_vm_key.pub \
     --size Standard_B1s \
     --public-ip-address-dns-name mylinuxvmdns
   ```

   - **Parameters Explained**:
     - `--image UbuntuLTS`: Uses the latest Ubuntu LTS release.
     - `--size Standard_B1s`: Cost-effective VM size.
     - `--public-ip-address-dns-name`: Assigns a DNS name (optional).

6. **Open Port 22 for SSH** (if not already done):
   ```bash
   az vm open-port --port 22 --resource-group MyResourceGroup --name MyLinuxVM
   ```

7. **Optional: Open Port 80 for HTTP** (needed if you plan to test a web server like NGINX):
   ```bash
   az vm open-port --port 80 --resource-group MyResourceGroup --name MyLinuxVM
   ```

8. **Retrieve Public IP or DNS**:
   ```bash
   az vm list-ip-addresses --resource-group MyResourceGroup --name MyLinuxVM --output table
   ```
   - Note the **Public IP address** or **DNS name** for SSH access.


## Step 2: Connect to Your Azure Linux VM via SSH

Now that your VM is up and running, it's time to connect to it using SSH.

### 2.1 Using SSH on macOS/Linux

1. **Open Terminal**.

2. **Connect to the VM**:
   ```bash
   ssh -i ~/.ssh/azure_vm_key azureuser@<Public-IP>
   ```
   - Replace `<Public-IP>` with your VM's public IP address or DNS name.

3. **Accept the Host Key**:
   - On first connection, you'll be prompted to accept the VM's SSH host key. Type **`yes`** and press **Enter**.

4. **You're In!**
   - You should now be logged into your Azure Linux VM.

### 2.2 Using SSH on Windows

**Option A: Using Windows Terminal (with OpenSSH)**

1. **Open Windows Terminal** or **PowerShell**.

2. **Connect to the VM**:
   ```powershell
   ssh -i C:\path\to\azure_vm_key azureuser@<Public-IP>
   ```
   - Replace `C:\path\to\azure_vm_key` with the path to your private SSH key.
   - Replace `<Public-IP>` with your VM's public IP address or DNS name.

3. **Accept the Host Key**:
   - Type **`yes`** when prompted and press **Enter**.

4. **You're In!**
   - You should now be logged into your Azure Linux VM.

**Option B: Using PuTTY**

1. **Convert SSH Key to PuTTY Format**:
   - Open **PuTTYgen**.
   - Click **"Load"** and select your `azure_vm_key` private key.
   - Save the key as a **`.ppk`** file.

2. **Open PuTTY**.

3. **Configure Connection**:
   - **Host Name**: Enter `azureuser@<Public-IP>`.
   - **Port**: `22`.

4. **Load SSH Key**:
   - In the left pane, navigate to **"Connection"** > **"SSH"** > **"Auth"**.
   - Click **"Browse"** and select your **`.ppk`** file.

5. **Connect**:
   - Click **"Open"**.
   - If prompted with a security alert, click **"Yes"** to trust the host.

6. **You're In!**
   - You should now be logged into your Azure Linux VM.


## Step 3: Verify Your Connection

Once connected, perform a few basic checks to ensure everything is working correctly.

1. **Check System Information**:
   ```bash
   uname -a
   ```
   - Confirms you're on the correct OS and kernel version.

2. **Update the Package List**:
   ```bash
   sudo apt-get update
   ```
   - Ensures your package list is up-to-date (for Ubuntu/Debian-based systems).

3. **Install Essential Packages** (Optional):
   ```bash
   sudo apt-get install -y git curl vim
   ```
   - Installs Git, Curl, and Vim editors for development and management.

4. **Check Disk Usage**:
   ```bash
   df -h
   ```
   - Ensures you have adequate disk space.

5. **Verify Network Connectivity**:
   ```bash
   ping -c 4 google.com
   ```
   - Checks if your VM can reach the internet.


## Step 4: (Optional) Configure Additional Settings

Depending on your use case, you might want to configure additional settings such as:

### 4.1 Configure a Firewall with UFW

1. **Install UFW** (Uncomplicated Firewall):
   ```bash
   sudo apt-get install -y ufw
   ```

2. **Allow SSH**:
   ```bash
   sudo ufw allow OpenSSH
   ```

3. **Allow HTTP**:
   ```bash
   sudo ufw allow http
   ```

4. **Enable UFW**:
   ```bash
   sudo ufw enable
   ```
   - Confirm by typing **`y`** when prompted.

5. **Check UFW Status**:
   ```bash
   sudo ufw status
   ```

### 4.2 Set Up a Web Server (e.g., NGINX)

1. **Install NGINX**:
   ```bash
   sudo apt-get install -y nginx
   ```

2. **Start and Enable NGINX**:
   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

3. **Verify Installation**:
   - Open a browser and navigate to `http://<Public-IP>` (ensure port 80 is open in both your NSG and UFW).
   - You should see the default NGINX welcome page.


## Step 5: Cleanup (Optional)

If you no longer need the VM and want to avoid incurring charges, you can delete the VM and its associated resources.

### 5.1 Delete via Azure Portal

1. **Navigate to Virtual Machines** in the Azure Portal.
2. **Select** your VM (`MyLinuxVM`).
3. Click on **"Delete"**.
4. Confirm deletion. This removes the VM but may leave other resources like the public IP or resource group.

### 5.2 Delete via Azure CLI

1. **Delete the VM**:
   ```bash
   az vm delete --resource-group MyResourceGroup --name MyLinuxVM --yes --no-wait
   ```

2. **Delete the Resource Group** (This removes all resources within it):
   ```bash
   az group delete --name MyResourceGroup --yes --no-wait
   ```

## **Deliverables**

* Submit the following artifacts:

  * Screenshot or exported view from the **Azure Portal** showing your created **Linux Virtual Machine** (VM name, region, and status “Running”).
  * Screenshot or terminal output showing a successful **SSH connection** to your VM (e.g., shell prompt of your remote machine).
  * (Optional if completed) Screenshot showing:

    * **NGINX** welcome page accessed via your VM’s public IP, or
    * **UFW** status if you configured a firewall.

* If you used the **Azure CLI**, include:

  * The full command(s) used to create the VM and open ports (`az vm create`, `az vm open-port`, etc.).
  * Screenshot of the CLI output confirming successful VM creation.

## **Submission**

Upon completion, add your deliverables to git. Then commit git and push your branch to the remote. Make a pull request and paste the PR link in the submission field in the Student Portal.

## Summary

In this lab, you successfully:

1. **Created an Azure Linux VM** using both the Azure Portal and Azure CLI.
2. **Configured networking and security** to allow SSH access (and optionally HTTP) access.
3. **Connected** to your VM via SSH from different operating systems.
4. **Verified** the VM's setup and performed basic configurations.
5. **Optionally** set up additional security measures and services like UFW and NGINX.
6. **Cleaned up** resources to prevent unwanted charges.

**Congratulations!** You've set up a Linux Virtual Machine on Azure and connected to it successfully. This foundation allows you to deploy applications, experiment with configurations, and integrate with other Azure services. Whether you're developing, testing, or running production workloads, your Azure VM is now ready for action. *Celebrate with some confetti—you've taken a significant step into cloud computing!*


## Next Steps

- **Explore Azure VM Extensions**: Automate post-deployment configurations like installing software or running scripts.
- **Implement SSH Key Rotation**: Enhance security by periodically updating your SSH keys.
- **Scale Your VM**: Adjust the size of your VM based on performance needs.
- **Integrate with Azure DevOps or GitHub Actions**: Automate deployments and manage your infrastructure as code.
- **Secure Your VM**: Set up monitoring, backups, and advanced security features using Azure Security Center.

Dive deeper into Azure's vast ecosystem and continue building robust, scalable, and secure cloud solutions!
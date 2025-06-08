
------------------------------------
# **Week- 3 { Assignment }**
------------------------------------

-------------------
## Task 1:-
Observe assigned Subscriptions Observe Azure Entra ID or create own Azure Entra ID in personal Azure account Create test users and groups Assign a RBAC role to user and test Create a custom role and assigned to users and test
------------------


### **1. Observe Assigned Subscriptions**

#### **Steps:**

1. Log in to the **[Azure Portal](https://portal.azure.com)**.
2. Click on your **profile icon** in the top-right corner.
3. Select **“Switch Directory”** to view all available directories.
4. Navigate to **“Subscriptions”** from the left-hand navigation menu.
5. Observe the details of assigned subscriptions including:

   * **Subscription Name**
   * **Subscription ID**
   * **Associated Directory (Tenant)**

---

###  **2. Observe or Create Microsoft Entra ID (Azure Active Directory)**

#### **To View Existing Entra ID (Directory):**

1. From the Azure Portal, search for and open **“Microsoft Entra ID”**.
2. The Overview section will show:

   * **Tenant Name**
   * **Tenant ID**
   * **Directory Type (e.g., Work/School Account or Personal Account)**

####  **To Create a New Microsoft Entra ID Tenant:**

> *(Useful if using a personal Microsoft account or for testing multiple tenants)*

1. Go to **Microsoft Entra ID** → Click **“Manage tenants”**.
2. Click **“+ Create”** → Select **“Azure Active Directory”**.
3. Enter:

   * **Organization Name** (e.g., `MyTestOrg`)
   * **Initial Domain Name** (e.g., `myorg123.onmicrosoft.com`)
4. Click **Review + Create**, then **Create** to provision the new tenant.

---

### **3. Create Test Users and Groups**

#### **Create Test Users:**

1. Go to **Microsoft Entra ID** → **Users** → Click **+ New User**.
2. Fill in the user details:

   * **Username**: `testuser1`
   * **Name**: Test User 1
   * **Password**: Auto-generate or custom (record this for login)
3. Click **Create**.

####  **Create Security Group:**

1. Navigate to **Microsoft Entra ID** → **Groups** → **+ New Group**.
2. Choose:

   * **Group Type**: Security
   * **Group Name**: `TestGroup`
   * **Members**: Add `testuser1`
3. Click **Create**.

---

###  **4. Assign RBAC Role to User and Test Access**

#### **Assign Reader Role to Test User:**

1. Go to **Subscriptions** → Select your active subscription.
2. Navigate to **Access Control (IAM)** → Click **+ Add** → **Add role assignment**.
3. Select:

   * **Role**: Reader
   * **Assign Access To**: User
   * **Members**: Select `testuser1`
4. Click **Next → Review + Assign → Assign**.

#### **Test the Access:**

1. Log out and log in using `testuser1` credentials.
2. Go to the Azure Portal and navigate to the assigned subscription.
3. Verify that:

   * The user can **view** resources.
   * The user **cannot create, delete, or modify** any resources.

---

###  **5. Create a Custom Role and Assign to User**

####  **Create a Custom Role via Azure Portal:**

1. Go to **Subscriptions** → Select the subscription → **Access Control (IAM)**.
2. Click **+ Add** → **Add custom role**.
3. In the custom role wizard:

   * **Name**: `CustomReaderPlus`
   * **Description**: Reader role with permission to start VMs.
   * **Baseline**: Start from `Reader` built-in role.
4. In the **Permissions** tab, add:

   * `Microsoft.Compute/virtualMachines/start/action`
5. Click **Next → Review + Create → Create**.

####  **Assign Custom Role to User:**

1. Go back to **IAM → Add Role Assignment**.
2. Select the **CustomReaderPlus** role.
3. Assign to:

   * **User**: `testuser1`

####  **Test Custom Access:**

* Log in with `testuser1`.
* Confirm:

  * User **can view resources**.
  * User **can start virtual machines**.
  * User **cannot stop/delete/create** VMs.

---



---------------------------------------------------------------
## Task:-2
Create Virtual maching and Vnet from Azure CLI
---------------------------------------------------------------


###  **Step 1: Log In to Azure**

Before you can run Azure CLI commands, you need to authenticate your session.

```bash
az login
```

* This will open a browser window.
* Log in with your Azure credentials.
* Upon successful login, your subscription details will be displayed in the terminal.

---

###  **Step 2: Create a Resource Group**

A resource group is a container that holds related resources for an Azure solution.

```bash
az group create --name MyResourceGroup --location eastus
```

* `--name`: Name of the resource group (`MyResourceGroup` in this case).
* `--location`: Region where resources will be deployed (e.g., `eastus`).

 *Output confirms that the resource group has been created successfully.*

---

### **Step 3: Create a Virtual Network (VNet) and Subnet**

A VNet is required to host your VM, and a subnet provides an address range for that network.

```bash
az network vnet create \
  --resource-group MyResourceGroup \
  --name MyVNet \
  --subnet-name MySubnet
```

* `--resource-group`: Specifies which resource group to place the VNet in.
* `--name`: Name of the Virtual Network.
* `--subnet-name`: Name of the subnet within the VNet.

 *This command creates both the VNet and its first subnet in one go.*

---

### **Step 4: Create a Virtual Machine**

Now that we have networking in place, we’ll create a Linux VM (Ubuntu LTS) and connect it to the created subnet.

```bash
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --vnet-name MyVNet \
  --subnet MySubnet \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

* `--name`: Name of the virtual machine.
* `--vnet-name`: Reference to the previously created Virtual Network.
* `--subnet`: Reference to the existing subnet.
* `--image`: The VM image to use (`UbuntuLTS` for latest Ubuntu long-term support).
* `--admin-username`: Username for admin access.
* `--generate-ssh-keys`: Auto-generates SSH keys for secure login.

 *After the VM is successfully created, the CLI will return:*

* Public IP address
* SSH command to connect
* OS details and provisioning state

---

### **Step 5: Connect to the VM via SSH**

Once the VM is created, connect to it using the following command (replace `PUBLIC_IP` with the actual IP returned):

```bash
ssh azureuser@<PUBLIC_IP>
```

* Accept the SSH fingerprint prompt.
* You will now be logged into your Ubuntu VM.

---


------------------------------------------------------------------------
## Task:-3
Create and assign a any policy at subscription level
------------------------------------------------------------------------




### **Step-by-Step: Assign a Policy in Azure Portal**

####  **Step 1: Access Azure Policy Service**

1. Go to the **[Azure Portal](https://portal.azure.com)**.
2. In the search bar at the top, type **“Policy”** and click on the result titled **Policy**.

---

#### **Step 2: Choose the Subscription Scope**

1. In the Policy blade, select **“Assignments”** from the left-hand menu.
2. Click the **“+ Assign Policy”** button at the top of the page.
3. Under the **Scope** section:

   * Click the **“…” (ellipsis)** button next to Scope.
   * In the popup window, **select your Subscription**.
   * *(Optional)* Select a specific **Resource Group** if you want to narrow the scope.
   * Click **Select** to confirm.

---

#### **Step 3: Select a Policy Definition**

1. Under **Policy Definition**, click the **“…”** to browse available built-in policies.

2. In the policy picker, search for:

   ```
   Deny public IP addresses
   ```

3. Select the built-in policy titled:

   ```
   Deny public IP addresses
   ```

   * This policy prevents users from creating resources with public IPs.
   * Policy Definition ID: \*(e.g., f71d2637-...)

4. Click **Select** to confirm your choice.

---

#### **Step 4: Configure the Policy Assignment**

1. Fill in the required fields:

   * **Policy Assignment Name**: `DenyPublicIP`
   * **Description** *(optional)*: `Prevent users from creating public IP addresses in the selected scope.`
2. Leave the remaining options at their **default settings**.

---

#### **Step 5: Review and Create**

1. Click **Next** to proceed through:

   * **Parameters** (skip if not required)
   * **Remediation** (skip if not configuring at this stage)
2. On the **Review + Create** page:

   * Validate the configuration.
   * Click **Create**.

 **You have now successfully assigned a policy at the subscription level to deny public IP creation.**

---

### **Verification**

To test the policy:

1. Attempt to deploy a VM or another resource that includes a public IP address in the targeted subscription.
2. Azure should automatically **deny** the deployment with a policy violation message.

---


--------------------------------------------------------------------------------
## Task:-4
Create an Azure key vault and store secrets. Configure access policies for the Key Vault to allow authorized users or applications to manage keys and secrets. retrieve secret from key vault using azure CLI
-------------------------------------------------------------------------------



#### Step 1: Create an Azure Key Vault

Use the Azure CLI to create a Key Vault named `MyKeyVault` in the specified resource group and location.

```bash
az keyvault create \
  --name MyKeyVault \
  --resource-group MyResourceGroup \
  --location eastus
```


---

### Step 2: Store a Secret in Key Vault

Add a secret (e.g., a password, API key, or connection string) to the vault.

```bash
az keyvault secret set \
  --vault-name MyKeyVault \
  --name MySecret \
  --value "mySecretValue"
```

 This command stores a secret named `MySecret` with the value `mySecretValue`.

---

### Step 3: Retrieve the Secret Using Azure CLI

You can retrieve the secret value using the `az keyvault secret show` command.

```bash
az keyvault secret show \
  --name MySecret \
  --vault-name MyKeyVault \
  --query value \
  -o tsv
```

 Output:

```
mySecretValue
```

 The `--query value -o tsv` ensures that only the value is returned in plain text format.

---

### Step 4: Configure Access Policies for the Key Vault

To allow users or applications to access secrets (read/write), configure access policies via the Azure portal:

#### Portal Steps:

1. Go to **Azure Portal** → **Key Vaults** → Select `MyKeyVault`.
2. In the left-hand menu, click on **Access policies**.
3. Click on **+ Add Access Policy**.
4. Under **Secret permissions**, choose required actions (e.g., `Get`, `Set`, `List`).
5. In the **Principal** field, select the user or application.
6. Click **Add** and then **Save**.

This step is crucial to enforce secure and controlled access to your secrets.

---


----------------------------------------------------------
# Task:-5
Create a VM from Powershell
---------------------------------------------------------


### Step-by-Step Process

#### Step 1: Login to Azure

Authenticate your PowerShell session with your Azure account.

```powershell
Connect-AzAccount
```

A browser window will prompt you to log in.

---

#### Step 2: Create a Resource Group

A resource group is a logical container for Azure resources.

```powershell
New-AzResourceGroup -Name "MyRG" -Location "EastUS"
```

 Replace `"MyRG"` with your preferred resource group name and `"EastUS"` with your desired Azure region.

---

#### Step 3: Create a Virtual Machine

This step creates a complete VM deployment including the virtual network, subnet, network security group, public IP, and VM itself.

```powershell
New-AzVM `
  -ResourceGroupName "MyRG" `
  -Name "MyVM" `
  -Location "EastUS" `
  -VirtualNetworkName "MyVNet" `
  -SubnetName "MySubnet" `
  -SecurityGroupName "MyNSG" `
  -PublicIpAddressName "MyPublicIP" `
  -Credential (Get-Credential)
```

 Breakdown of Parameters:

* `-ResourceGroupName`: The name of the resource group.
* `-Name`: The name of the virtual machine.
* `-VirtualNetworkName`: Creates a new virtual network if not already present.
* `-SubnetName`: Subnet within the VNet.
* `-SecurityGroupName`: Network security group (NSG) with default rules.
* `-PublicIpAddressName`: A new public IP for external access.
* `-Credential`: Prompts for a username and password for the VM.

 This command uses default configurations for OS, size, and image (usually Windows Server unless specified).

---

### (Optional) Step 4: Verify Deployment

You can list your VM to confirm successful deployment:

```powershell
Get-AzVM -ResourceGroupName "MyRG"
```

---

---------------------------------------------------------
#Task:-6
A. Schedule a Daily backup of VM at 3:AM using vault 1. Create an Alert rule for VM CPU percentage: Criteria: CPU% MoreThan 80 There Should be analert on Email." B.Provision backups in backup center 2. Schedule a Daily backup of VM at 3:AM using vault. Configure Retention period in backup policy and retain an old backup
----------------------------------------------------------


### Section A: Schedule a Daily Backup at 3:00 AM Using Recovery Services Vault

#### Step 1: Create a Recovery Services Vault

1. Go to **Azure Portal** → Search for **Recovery Services vaults**.
2. Click **+ Create**.
3. Configure the following:

   * **Name**: `MyBackupVault`
   * **Resource Group**: Choose an existing one or create a new one.
   * **Region**: Must be the **same** as your VM (e.g., `East US`).
4. Click **Review + Create**, then **Create**.

---

#### Step 2: Set Up Backup for the Virtual Machine

1. Open the created **Recovery Services Vault**.
2. Navigate to **Backup**.
3. Under configuration:

   * **Where is your workload running?** → `Azure`
   * **What do you want to back up?** → `Virtual Machine`
4. Click **+ Backup**.
5. Select the VM(s) to be backed up from the same region.
6. Click **Enable Backup**.

---

#### Step 3: Configure Backup Policy (Schedule at 3:00 AM)

1. In the vault, go to **Backup Policies** → Click **+ Add** or edit an existing policy.
2. Set:

   * **Policy Name**: `Daily3AMPolicy`
   * **Backup Frequency**: `Daily`
   * **Time**: `3:00 AM`
   * **Time Zone**: Match your region (e.g., `Asia/Kolkata`)
   * **Retention**: Keep backups for 30 days (or as required)
3. Save the policy.
4. Go to **Backup Items → Azure Virtual Machines**, locate your VM, and ensure the policy `Daily3AMPolicy` is assigned.

---

### Section B: Create an Alert Rule for CPU > 80% with Email Notification

#### Step 1: Open Alerts Configuration

1. Navigate to **Azure Portal** → Go to your **Virtual Machine**.
2. From the left-hand menu, select **Monitoring → Alerts**.
3. Click **+ Create Alert Rule**.

---

#### Step 2: Define Alert Condition

1. **Scope**: Should be automatically filled with your VM.
2. Click **Add Condition**:

   * **Signal Name**: `Percentage CPU`
   * **Condition**: `Greater than`
   * **Threshold Value**: `80`
   * **Aggregation**: `Average` over `5 minutes`
3. Click **Done**.

---

#### Step 3: Create an Action Group (Email Notification)

1. In the **Action Group** section, click **+ Create action group**.
2. Fill in:

   * **Name**: `EmailAlertGroup`
   * **Short Name**: `emailgrp`
3. Under **Notifications**:

   * Select **Email/SMS/Push/Voice**
   * Enter your email address.
4. Click **OK**, then **Review + Create**.

---

#### Step 4: Finalize Alert Rule

1. Set:

   * **Alert Rule Name**: `HighCPUAlert`
   * **Severity**: `Sev 2 (Warning)`
2. Click **Create**.

Now, if CPU usage exceeds 80% for more than 5 minutes, an email alert will be triggered.

---

### Section C: Provision and Monitor Backups via Backup Center

1. Go to **Azure Portal** → Search for **Backup Center**.
2. Here you can:

   * View all backup items
   * Monitor job statuses
   * Manage Recovery Vaults and policies
3. Optionally, click **+ Backup** to initiate new backup configurations from here.

---

### (Optional) CLI Equivalents

#### Create Recovery Services Vault

```bash
az backup vault create \
  --resource-group MyResourceGroup \
  --name MyBackupVault \
  --location eastus
```

#### Register a VM for Backup

```bash
az backup protection enable-for-vm \
  --resource-group MyResourceGroup \
  --vault-name MyBackupVault \
  --vm MyVM \
  --policy-name DefaultPolicy
```

#### Create Alert Rule for CPU > 80%

```bash
az monitor metrics alert create \
  --name HighCPUAlert \
  --resource-group MyResourceGroup \
  --scopes /subscriptions/<subscription-id>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyVM \
  --condition "avg Percentage CPU > 80" \
  --description "High CPU alert" \
  --action-groups <action-group-id>
```

Replace:

* `<subscription-id>` with your subscription ID
* `MyVM` with your VM name
* `<action-group-id>` with the actual Action Group Resource ID

---






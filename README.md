# Complete AZ-104 Guide: Microsoft Azure Administrator

## 📋 Certification Overview

The AZ-104 exam validates your skills as an Azure Administrator. It covers subscription management, security, storage, networking, and compute resources.

**Duration:** 150 minutes  
**Questions:** 40-60 questions  
**Passing Score:** 700/1000  
**Cost:** $165 USD

---

## 🎯 Exam Domains (Weight Distribution)

### 1. Manage Azure identities and governance (15-20%)
### 2. Implement and manage storage (15-20%)
### 3. Deploy and manage Azure compute resources (20-25%)
### 4. Configure and manage virtual networking (25-30%)
### 5. Monitor and back up Azure resources (10-15%)

---

## 🔐 1. Azure Identities and Governance

### Azure Active Directory (Azure AD / Entra ID)

#### Key Concepts
- Tenants and directories
- Users, groups, and administrative units
- Built-in vs custom roles
- Multi-factor authentication (MFA)
- Conditional Access

#### User Management Commands

**Azure CLI:**
```bash
# Create a user
az ad user create --display-name "John Doe" --password "TempPassword123!" --user-principal-name "john.doe@yourdomain.com"

# List all users
az ad user list --output table

# Get user details
az ad user show --id "john.doe@yourdomain.com"

# Update user
az ad user update --id "john.doe@yourdomain.com" --display-name "John Smith"

# Delete user
az ad user delete --id "john.doe@yourdomain.com"

# Reset password
az ad user update --id "john.doe@yourdomain.com" --password "NewPassword123!" --force-change-password-next-sign-in true
```

**PowerShell:**
```powershell
# Connect to Azure AD
Connect-AzureAD

# Create a user
$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$passwordProfile.Password = "TempPassword123!"
$passwordProfile.ForceChangePasswordNextLogin = $true
New-AzureADUser -DisplayName "John Doe" -PasswordProfile $passwordProfile -UserPrincipalName "john.doe@yourdomain.com" -AccountEnabled $true -MailNickName "johndoe"

# List all users
Get-AzureADUser

# Get user details
Get-AzureADUser -ObjectId "john.doe@yourdomain.com"

# Update user
Set-AzureADUser -ObjectId "john.doe@yourdomain.com" -DisplayName "John Smith"

# Delete user
Remove-AzureADUser -ObjectId "john.doe@yourdomain.com"
```

#### Group Management Commands

**Azure CLI:**
```bash
# Create a security group
az ad group create --display-name "IT Department" --mail-nickname "ITDept"

# List all groups
az ad group list --output table

# Add user to group
az ad group member add --group "IT Department" --member-id "john.doe@yourdomain.com"

# Remove user from group
az ad group member remove --group "IT Department" --member-id "john.doe@yourdomain.com"

# List group members
az ad group member list --group "IT Department" --output table
```

**PowerShell:**
```powershell
# Create a security group
New-AzureADGroup -DisplayName "IT Department" -MailEnabled $false -SecurityEnabled $true -MailNickName "ITDept"

# List all groups
Get-AzureADGroup

# Add user to group
Add-AzureADGroupMember -ObjectId "group-object-id" -RefObjectId "user-object-id"

# Remove user from group
Remove-AzureADGroupMember -ObjectId "group-object-id" -MemberId "user-object-id"

# List group members
Get-AzureADGroupMember -ObjectId "group-object-id"
```

### Role-Based Access Control (RBAC)

#### Built-in Roles
- **Owner:** Full access including access management
- **Contributor:** Full access except access management
- **Reader:** View only access
- **User Access Administrator:** Manage user access only

#### RBAC Commands

**Azure CLI:**
```bash
# List all role definitions
az role definition list --output table

# Get specific role definition
az role definition show --name "Virtual Machine Contributor"

# Create custom role
az role definition create --role-definition @customrole.json

# List role assignments
az role assignment list --output table

# Assign role to user
az role assignment create --assignee "user@domain.com" --role "Virtual Machine Contributor" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# Assign role to group
az role assignment create --assignee-object-id "group-object-id" --role "Contributor" --scope "/subscriptions/subscription-id"

# Remove role assignment
az role assignment delete --assignee "user@domain.com" --role "Virtual Machine Contributor" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"
```

**PowerShell:**
```powershell
# List all role definitions
Get-AzRoleDefinition

# Get specific role definition
Get-AzRoleDefinition -Name "Virtual Machine Contributor"

# Create custom role
New-AzRoleDefinition -InputFile "customrole.json"

# List role assignments
Get-AzRoleAssignment

# Assign role to user
New-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName "myResourceGroup"

# Assign role to group
New-AzRoleAssignment -ObjectId "group-object-id" -RoleDefinitionName "Contributor" -Scope "/subscriptions/subscription-id"

# Remove role assignment
Remove-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName "myResourceGroup"
```

### Azure Policy

#### Policy Commands

**Azure CLI:**
```bash
# List policy definitions
az policy definition list --output table

# Get policy definition
az policy definition show --name "policy-definition-name"

# Create policy definition
az policy definition create --name "my-policy" --display-name "My Policy" --description "Policy description" --rules @policy-rules.json --params @policy-params.json

# List policy assignments
az policy assignment list --output table

# Assign policy
az policy assignment create --name "my-assignment" --display-name "My Assignment" --policy "policy-definition-id" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# List policy compliance
az policy state list --output table

# Trigger policy evaluation
az policy state trigger-scan --resource-group "myResourceGroup"
```

**PowerShell:**
```powershell
# List policy definitions
Get-AzPolicyDefinition

# Get policy definition
Get-AzPolicyDefinition -Name "policy-definition-name"

# Create policy definition
New-AzPolicyDefinition -Name "my-policy" -DisplayName "My Policy" -Description "Policy description" -Policy "policy-rules.json" -Parameter "policy-params.json"

# List policy assignments
Get-AzPolicyAssignment

# Assign policy
New-AzPolicyAssignment -Name "my-assignment" -DisplayName "My Assignment" -PolicyDefinition $policyDef -Scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# Get policy compliance
Get-AzPolicyState
```

### Subscription Management

**Azure CLI:**
```bash
# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "subscription-id"

# Show current subscription
az account show

# List locations
az account list-locations --output table

# Create management group
az account management-group create --name "my-mg" --display-name "My Management Group"

# List management groups
az account management-group list --output table
```

**PowerShell:**
```powershell
# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -SubscriptionId "subscription-id"

# Show current subscription
Get-AzContext

# List locations
Get-AzLocation

# Create management group
New-AzManagementGroup -GroupName "my-mg" -DisplayName "My Management Group"

# List management groups
Get-AzManagementGroup
```

---

## 💾 2. Azure Storage

### Storage Account Management

#### Storage Account Commands

**Azure CLI:**
```bash
# Create storage account
az storage account create --name "mystorageaccount" --resource-group "myResourceGroup" --location "eastus" --sku "Standard_LRS" --kind "StorageV2"

# List storage accounts
az storage account list --output table

# Show storage account
az storage account show --name "mystorageaccount" --resource-group "myResourceGroup"

# Update storage account
az storage account update --name "mystorageaccount" --resource-group "myResourceGroup" --sku "Standard_GRS"

# Delete storage account
az storage account delete --name "mystorageaccount" --resource-group "myResourceGroup"

# Get storage account keys
az storage account keys list --account-name "mystorageaccount" --resource-group "myResourceGroup"

# Regenerate storage account key
az storage account keys renew --account-name "mystorageaccount" --resource-group "myResourceGroup" --key "key1"
```

**PowerShell:**
```powershell
# Create storage account
New-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -Location "East US" -SkuName "Standard_LRS" -Kind "StorageV2"

# List storage accounts
Get-AzStorageAccount

# Show storage account
Get-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Update storage account
Set-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -SkuName "Standard_GRS"

# Delete storage account
Remove-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Get storage account keys
Get-AzStorageAccountKey -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Regenerate storage account key
New-AzStorageAccountKey -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -KeyName "key1"
```

### Blob Storage

**Azure CLI:**
```bash
# Create container
az storage container create --name "mycontainer" --account-name "mystorageaccount"

# List containers
az storage container list --account-name "mystorageaccount" --output table

# Upload blob
az storage blob upload --file "localfile.txt" --container-name "mycontainer" --name "myblob.txt" --account-name "mystorageaccount"

# Download blob
az storage blob download --container-name "mycontainer" --name "myblob.txt" --file "downloadedfile.txt" --account-name "mystorageaccount"

# List blobs
az storage blob list --container-name "mycontainer" --account-name "mystorageaccount" --output table

# Delete blob
az storage blob delete --container-name "mycontainer" --name "myblob.txt" --account-name "mystorageaccount"

# Set blob access tier
az storage blob set-tier --container-name "mycontainer" --name "myblob.txt" --tier "Cool" --account-name "mystorageaccount"

# Generate SAS token
az storage blob generate-sas --container-name "mycontainer" --name "myblob.txt" --permissions "r" --expiry "2024-12-31" --account-name "mystorageaccount"
```

**PowerShell:**
```powershell
# Get storage context
$ctx = (Get-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount").Context

# Create container
New-AzStorageContainer -Name "mycontainer" -Context $ctx

# List containers
Get-AzStorageContainer -Context $ctx

# Upload blob
Set-AzStorageBlobContent -File "localfile.txt" -Container "mycontainer" -Blob "myblob.txt" -Context $ctx

# Download blob
Get-AzStorageBlobContent -Container "mycontainer" -Blob "myblob.txt" -Destination "downloadedfile.txt" -Context $ctx

# List blobs
Get-AzStorageBlob -Container "mycontainer" -Context $ctx

# Delete blob
Remove-AzStorageBlob -Container "mycontainer" -Blob "myblob.txt" -Context $ctx

# Set blob access tier
Set-AzStorageBlobContent -File "localfile.txt" -Container "mycontainer" -Blob "myblob.txt" -Context $ctx -StandardBlobTier "Cool"
```

### Azure Files

**Azure CLI:**
```bash
# Create file share
az storage share create --name "myfileshare" --account-name "mystorageaccount"

# List file shares
az storage share list --account-name "mystorageaccount" --output table

# Upload file
az storage file upload --share-name "myfileshare" --source "localfile.txt" --path "myfile.txt" --account-name "mystorageaccount"

# Download file
az storage file download --share-name "myfileshare" --path "myfile.txt" --dest "downloadedfile.txt" --account-name "mystorageaccount"

# List files
az storage file list --share-name "myfileshare" --account-name "mystorageaccount" --output table

# Delete file
az storage file delete --share-name "myfileshare" --path "myfile.txt" --account-name "mystorageaccount"

# Create directory
az storage directory create --share-name "myfileshare" --name "mydirectory" --account-name "mystorageaccount"
```

**PowerShell:**
```powershell
# Create file share
New-AzStorageShare -Name "myfileshare" -Context $ctx

# List file shares
Get-AzStorageShare -Context $ctx

# Upload file
Set-AzStorageFileContent -ShareName "myfileshare" -Source "localfile.txt" -Path "myfile.txt" -Context $ctx

# Download file
Get-AzStorageFileContent -ShareName "myfileshare" -Path "myfile.txt" -Destination "downloadedfile.txt" -Context $ctx

# List files
Get-AzStorageFile -ShareName "myfileshare" -Context $ctx

# Delete file
Remove-AzStorageFile -ShareName "myfileshare" -Path "myfile.txt" -Context $ctx

# Create directory
New-AzStorageDirectory -ShareName "myfileshare" -Path "mydirectory" -Context $ctx
```

---

## 🖥️ 3. Azure Compute Resources

### Virtual Machines

#### VM Management Commands

**Azure CLI:**
```bash
# Create VM
az vm create --resource-group "myResourceGroup" --name "myVM" --image "Ubuntu2204" --admin-username "azureuser" --generate-ssh-keys --size "Standard_B2s"

# Create Windows VM
az vm create --resource-group "myResourceGroup" --name "myWindowsVM" --image "Win2022Datacenter" --admin-username "azureuser" --admin-password "MyPassword123!" --size "Standard_B2s"

# List VMs
az vm list --output table

# Show VM details
az vm show --resource-group "myResourceGroup" --name "myVM"

# Start VM
az vm start --resource-group "myResourceGroup" --name "myVM"

# Stop VM
az vm stop --resource-group "myResourceGroup" --name "myVM"

# Restart VM
az vm restart --resource-group "myResourceGroup" --name "myVM"

# Deallocate VM
az vm deallocate --resource-group "myResourceGroup" --name "myVM"

# Delete VM
az vm delete --resource-group "myResourceGroup" --name "myVM"

# Resize VM
az vm resize --resource-group "myResourceGroup" --name "myVM" --size "Standard_B4ms"

# List available VM sizes
az vm list-sizes --location "eastus" --output table

# List VM images
az vm image list --output table

# Get VM status
az vm get-instance-view --resource-group "myResourceGroup" --name "myVM" --query "instanceView.statuses[1].displayStatus"
```

**PowerShell:**
```powershell
# Create VM
$cred = Get-Credential
New-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" -Location "East US" -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -Credential $cred

# List VMs
Get-AzVM

# Show VM details
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Start VM
Start-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Stop VM
Stop-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Restart VM
Restart-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Delete VM
Remove-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Resize VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
$vm.HardwareProfile.VmSize = "Standard_B4ms"
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# List available VM sizes
Get-AzVMSize -Location "East US"

# Get VM status
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" -Status
```

### VM Disks

**Azure CLI:**
```bash
# Create managed disk
az disk create --resource-group "myResourceGroup" --name "myDataDisk" --size-gb 128 --sku "Premium_LRS"

# List disks
az disk list --output table

# Attach disk to VM
az vm disk attach --resource-group "myResourceGroup" --vm-name "myVM" --name "myDataDisk"

# Detach disk from VM
az vm disk detach --resource-group "myResourceGroup" --vm-name "myVM" --name "myDataDisk"

# Update disk size
az disk update --resource-group "myResourceGroup" --name "myDataDisk" --size-gb 256

# Create snapshot
az snapshot create --resource-group "myResourceGroup" --name "mySnapshot" --source "myDataDisk"

# Create disk from snapshot
az disk create --resource-group "myResourceGroup" --name "myNewDisk" --source "mySnapshot"
```

**PowerShell:**
```powershell
# Create managed disk
$diskConfig = New-AzDiskConfig -Location "East US" -CreateOption "Empty" -DiskSizeGB 128 -SkuName "Premium_LRS"
New-AzDisk -ResourceGroupName "myResourceGroup" -DiskName "myDataDisk" -Disk $diskConfig

# List disks
Get-AzDisk

# Attach disk to VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
$disk = Get-AzDisk -ResourceGroupName "myResourceGroup" -Name "myDataDisk"
$vm = Add-AzVMDataDisk -VM $vm -Name "myDataDisk" -CreateOption "Attach" -ManagedDiskId $disk.Id -Lun 1
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# Detach disk from VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
Remove-AzVMDataDisk -VM $vm -Name "myDataDisk"
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# Create snapshot
$disk = Get-AzDisk -ResourceGroupName "myResourceGroup" -Name "myDataDisk"
$snapshotConfig = New-AzSnapshotConfig -SourceUri $disk.Id -CreateOption "Copy" -Location "East US"
New-AzSnapshot -ResourceGroupName "myResourceGroup" -SnapshotName "mySnapshot" -Snapshot $snapshotConfig
```

### Azure App Service

**Azure CLI:**
```bash
# Create App Service plan
az appservice plan create --name "myAppServicePlan" --resource-group "myResourceGroup" --sku "B1" --location "eastus"

# List App Service plans
az appservice plan list --output table

# Create web app
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "myWebApp" --runtime "NODE|14-lts"

# List web apps
az webapp list --output table

# Show web app
az webapp show --resource-group "myResourceGroup" --name "myWebApp"

# Start web app
az webapp start --resource-group "myResourceGroup" --name "myWebApp"

# Stop web app
az webapp stop --resource-group "myResourceGroup" --name "myWebApp"

# Restart web app
az webapp restart --resource-group "myResourceGroup" --name "myWebApp"

# Delete web app
az webapp delete --resource-group "myResourceGroup" --name "myWebApp"

# Deploy code
az webapp deployment source config --resource-group "myResourceGroup" --name "myWebApp" --repo-url "https://github.com/user/repo" --branch "main"

# Create deployment slot
az webapp deployment slot create --resource-group "myResourceGroup" --name "myWebApp" --slot "staging"

# Swap deployment slots
az webapp deployment slot swap --resource-group "myResourceGroup" --name "myWebApp" --slot "staging" --target-slot "production"
```

**PowerShell:**
```powershell
# Create App Service plan
New-AzAppServicePlan -ResourceGroupName "myResourceGroup" -Name "myAppServicePlan" -Location "East US" -Tier "Basic" -NumberofWorkers 1 -WorkerSize "Small"

# List App Service plans
Get-AzAppServicePlan

# Create web app
New-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp" -Location "East US" -AppServicePlan "myAppServicePlan"

# List web apps
Get-AzWebApp

# Show web app
Get-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Start web app
Start-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Stop web app
Stop-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Restart web app
Restart-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Delete web app
Remove-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Create deployment slot
New-AzWebAppSlot -ResourceGroupName "myResourceGroup" -Name "myWebApp" -Slot "staging"

# Swap deployment slots
Switch-AzWebAppSlot -ResourceGroupName "myResourceGroup" -Name "myWebApp" -SourceSlotName "staging" -DestinationSlotName "production"
```

### Container Instances

**Azure CLI:**
```bash
# Create container instance
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "mcr.microsoft.com/azuredocs/aci-helloworld" --dns-name-label "my-container-dns" --ports 80

# List container instances
az container list --output table

# Show container instance
az container show --resource-group "myResourceGroup" --name "mycontainer"

# Get container logs
az container logs --resource-group "myResourceGroup" --name "mycontainer"

# Execute command in container
az container exec --resource-group "myResourceGroup" --name "mycontainer" --exec-command "/bin/bash"

# Delete container instance
az container delete --resource-group "myResourceGroup" --name "mycontainer"

# Create container with environment variables
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "myimage" --environment-variables "ENV_VAR1=value1" "ENV_VAR2=value2"

# Create container with persistent storage
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "myimage" --azure-file-volume-account-name "mystorageaccount" --azure-file-volume-account-key "key" --azure-file-volume-share-name "myshare" --azure-file-volume-mount-path "/mnt/azurefile"
```

**PowerShell:**
```powershell
# Create container instance
New-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer" -Image "mcr.microsoft.com/azuredocs/aci-helloworld" -Location "East US" -DnsNameLabel "my-container-dns" -Port 80

# List container instances
Get-AzContainerGroup

# Show container instance
Get-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer"

# Get container logs
Get-AzContainerInstanceLog -ResourceGroupName "myResourceGroup" -ContainerGroupName "mycontainer" -ContainerName "mycontainer"

# Delete container instance
Remove-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer"
```

---

## 🌐 4. Virtual Networking

### Virtual Networks

**Azure CLI:**
```bash
# Create virtual network
az network vnet create --resource-group "myResourceGroup" --name "myVnet" --address-prefix "10.0.0.0/16" --subnet-name "mySubnet" --subnet-prefix "10.0.1.0/24"

# List virtual networks
az network vnet list --output table

# Show virtual network
az network vnet show --resource-group "myResourceGroup" --name "myVnet"

# Update virtual network
az network vnet update --resource-group "myResourceGroup" --name "myVnet" --dns-servers "8.8.8.8" "8.8.4.4"

# Delete virtual network
az network vnet delete --resource-group "myResourceGroup" --name "myVnet"

# Create subnet
az network vnet subnet create --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet2" --address-prefix "10.0.2.0/24"

# List subnets
az network vnet subnet list --resource-group "myResourceGroup" --vnet-name "myVnet" --output table

# Update subnet
az network vnet subnet update --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet" --network-security-group "myNSG"

# Delete subnet
az network vnet subnet delete --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet2"
```

**PowerShell:**
```powershell
# Create virtual network
$subnet = New-AzVirtualNetworkSubnetConfig -Name "mySubnet" -AddressPrefix "10.0.1.0/24"
New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Location "East US" -Name "myVnet" -AddressPrefix "10.0.0.0/16" -Subnet $subnet

# List virtual networks
Get-AzVirtualNetwork

# Show virtual network
Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"

# Update virtual network
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
$vnet.DhcpOptions.DnsServers = @("8.8.8.8", "8.8.4.4")
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Delete virtual network
Remove-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"

# Create subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
Add-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "mySubnet2" -AddressPrefix "10.0.2.0/24"
Set-AzVirtualNetwork -VirtualNetwork $vnet

# List subnets
Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet

# Delete subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
Remove-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "mySubnet2"
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### Network Security Groups

**Azure CLI:**
```bash
# Create NSG
az network nsg create --resource-group "myResourceGroup" --name "myNSG"

# List NSGs
az network nsg list --output table

# Show NSG
az network nsg show --resource-group "myResourceGroup" --name "myNSG"

# Create NSG rule
az network nsg rule create --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP" --protocol "TCP" --priority 1000 --destination-port-range "80" --access "Allow"

# List NSG rules
az network nsg rule list --resource-group "myResourceGroup" --nsg-name "myNSG" --output table

# Update NSG rule
az network nsg rule update --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP" --priority 1001

# Delete NSG rule
az network nsg rule delete --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP"

# Associate NSG with subnet
az network vnet subnet update --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet" --network-security-group "myNSG"

# Associate NSG with NIC
az network nic update --resource-group "myResourceGroup" --name "myNIC" --network-security-group "myNSG"
```

**PowerShell:**
```powershell
# Create NSG
New-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Location "East US" -Name "myNSG"

# List NSGs
Get-AzNetworkSecurityGroup

# Show NSG
Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"

# Create NSG rule
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"
Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowHTTP" -Description "Allow HTTP" -Access "Allow" -Protocol "TCP" -Direction "Inbound" -Priority 1000 -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange "80"
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# List NSG rules
Get-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg

# Update NSG rule
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"
Set-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowHTTP" -Priority 1001 -Access "Allow" -Protocol "TCP" -Direction "Inbound" -SourceAddressPrefix "*" -SourcePortRange

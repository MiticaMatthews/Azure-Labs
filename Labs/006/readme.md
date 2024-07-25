![net peering banner](https://github.com/user-attachments/assets/6e798d9f-7409-4817-8a71-a783bd51db04)

## What is Azure Network Peering?

## Requirements 
1. **Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/.
2. **Azure Cloud Shell:** You can launch and run the free interactive shell in the Azure portal. It has common Azure tools preinstalled and configured to use with your account.
3. **Azure CLI:** Alternatively, you can opt to use the terminal on your local machine. Ensure that Azure CLI is installed on your machine. Follow the official guidance on [How to Install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).

I will be using the terminal on my local machine for this task. 

## Create Virtual Networks 
1. Login to the Azure portal and authenticate your account via the terminal by running the following command:

    ```
    az login
    ```

You will not be able to run commands in Azure using the CLI without completing the above step. 

### Create Resource Group
Before we can proceed with creating our Azure Virutal Machine, we must create a resource group with the ```az group create``` command. When we create, provision, deploy etc. a resource, it must be assigned to a resource group. Resource groups are used to organise resources in Azure. Think of a resource group as a folder that helps you organise and manage your resources efficiently. 

2. Run the following command to create a resource group named **rg-VNpeering-001** in **westeurope**. 

   ```
   az group create --name rg-VNpeering-001 --location westeurope
   ```
   
### Create Virtual Networks and Subnets
3. Next, we will create three virtual networks using the `az network vnet create` command.

   Run the following command to create a virtual network named Engineering-VNet:

   ```
   az network vnet create \
   --name Engineering-VNet
   --resource-group rg-VNpeering-001 \
   --address-prefix 10.1.0.0/16 \
   --subnet-name Apps \
   --subnet-prefix 10.1.1.0/24 \
   --location westeurope 
   ```

   Run the following command to create a virtual network named Marketing-VNet:

   ```
   az network vnet create \
   --name Marketing-VNet
   --resource-group rg-VNpeering-001 \
   --address-prefix 10.2.0.0/16 \
   --subnet-name Digital \
   --subnet-prefix 10.2.1.0/24 \
   --location westeurope 
   ```

   Run the following command to create a virtual network named Finance-VNet:

   ```
   az network vnet create \
   --name Finance-VNet
   --resource-group rg-VNpeering-001 \
   --address-prefix 10.3.0.0/16 \
   --subnet-name Data \
   --subnet-prefix 10.3.1.0/24 \
   --location northeurope 
   ```

### Verify Successful Virtual Network Configuration

4. Run the following command to view the virtual networks in your resource group:

   ```
   az network vnet list --resource-group rg-VNpeering-001 --output table
   ```

   Example output:
   
   ```output
   
   ```

### Create Virtual Machines
5. Next, we will deploy an Ubuntu virtual machine (VM) in each virtual network. These VMs simulate the services in each virtual network. In the later steps of this tutorial, we'll use these VMs to test connectivity between the virtual networks.

   Run the following command to create a VM in the **Apps** subnet of **Engineering-VNet**:
   
   ```
    az vm create \
    --resource-group rg-VNpeering-001 \
    --name EngineeringVM01 \
    --image Ubuntu2204 \
    --vnet-name Engineering-VNet \
    --subnet Apps \
    --admin-username azureuser \
    --generate-ssh-keys \
    --no-wait
   ```

   Run the following command to create a VM in the **Digital** subnet of **Marketing-VNet**:
      
   ```
       az vm create \
    --resource-group rg-VNpeering-001 \
    --name MarketingVM01 \
    --image Ubuntu2204 \
    --vnet-name Marketing-VNet \
    --subnet Digital \
    --admin-username azureuser \
    --generate-ssh-keys \
    --no-wait
   ```

   Run the following command to create a VM in the **Data** subnet of **Finance-VNet**:
      
   ```
       az vm create \
    --resource-group rg-VNpeering-001 \
    --name FinanceVM01 \
    --image Ubuntu2204 \
    --vnet-name Finance-VNet \
    --subnet Data \
    --admin-username azureuser \
    --generate-ssh-keys
   ```

   **Note:** VMs can take a few minutes to create. Once the VM has been successfully created, the Azure CLI should output something similar to the following example:

    ```output
    
    ```

   **Note:** Take note of the `public IP address`, as we will need this address to access the VM from the internet in a later step.

### Get Virtual Machine Power State
6. To check the power state of **all** virtual machines in your resource group, run the following command:

   ```
    az vm get-instance-view --ids $(az vm list --resource-group rg-VM-03 --query "[].id" -- output tsv) --query "[*].{VMName:name, ProvisioningState:instanceView.statuses[0].displayStatus, PowerState:instanceView.statuses[1].displayStatus}" --output table
   ```

   Example output:

   ```output
   
   ```

   A provisioning state of "Succeeded" and a power state of "VM running" indicates the successful deployment of VMs. Once all three VMs are running, we can move on to the next step of connecting the VNets with virtual network peering.




## Delete Resources 

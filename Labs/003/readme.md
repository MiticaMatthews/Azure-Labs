![banner](https://github.com/user-attachments/assets/a588b853-4730-45a9-959b-2e15e017ecb3)

In this lab, I will walk you through the process of deploying an Ubuntu Virtual Machine in Microsoft Azure using the Azure Command-Line Interface (CLI). 

## Requirements 
1. **Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/.
2. **Azure Cloud Shell:** You can launch and run the free interactive shell in the Azure portal. It has common Azure tools preinstalled and configured to use with your account.
3. **Azure CLI:** Alternatively, you can opt to use the terminal on your local machine. Ensure that Azure CLI is installed on your machine. Follow the official guidance on [How to Install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).

I will be using the terminal on my local machine for this task. 
 
## Create an Ubuntu Virtual Machine Using Azure CLI
1. Sign in to the [Azure Portal](https://portal.azure.com/) and authenticate your account via your terminal by running the following command:

```az login```

You will not be able to run commands in Azure using the CLI without completing the above step. 

### Create an Azure Resource Group
Before we can proceed with creating our Azure Virutal Machine, we must create a resource group with the ```az group create``` command. When we create, provision, deploy etc. a resource, it must be assigned to a resource group. Resource groups are used to organise resources in Azure. Think of a resource group as a folder that helps you organise and manage your resources efficiently. 

2. Run the following command to create your resource group:

``` az group create --name rg-VM-01 --location ukwest```

***Note:*** *The above example creates a resource group named rg-VM-01 in the (Europe) UK West location. You can replace the variable values to a resource group name that you will easily remember, and an Azure datacenter location that is close to you.* 

### Create Ubuntu VM
Next, let's create an Ubuntu virtual machine. I will break down each part of the command to aid your understanding. 

3. Run the following command to create an Ubuntu VM:

```
az vm create \
  --resource-group rg-VM-01 \
  --name myUbuntuVM01 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys\
  --public-ip-sku Standard
```

```rg-VM-01```: This is the resource group we created in the previous step. 
```myUbuntuVM01```: Give your virtual machine a name that makes sense to you. 
```Ubuntu2204```: We have opted for Canonical Ubuntu 22.04 LTS, which is the latest (Long Term Support) version release. 
```azureuser```: This is your username. Make it something memorable. 
```--generate-ssh-keys```: This takes care of secure access.
```--public-ip-sku Standard```: We are using the "Standard" verion for the public IP. 

When your Ubuntu virtual machine has been successfully created, you should see something like this: 


### Test Ubuntu VM
To check whether the newly created virtual machine is reachable, we must run the ```ping``` command. Think of it as testing the connection to your new virtual computer in the cloud. 

4. Run the following command to test your VM:
```ping <public_ip_address>```

Remember to replace <public_ip_address> with the **public IP address** of your virtual machine. If you see responses, it means you are connected. Once you have confirmed that your virtual machine is reachable, you can press ctrl+C to exit. 

## Connect to Virtual Machine
5. Next, open a SSH connection to the created virtual machine with the following syntax:
``` ssh <username>@<public_ip_address>```.

Replace <username> with your chosen **username**, and <public_ip_address> with the **public IP address** of your virtual machine. 

In the example command from step 3, we chose ***azureuser*** as our username. If you used that username, your command would look like the following: ```azureuser@<public_ip_address>```. 

### Troubleshoot

## Manage Virtual Machine 

### Start VM

### Stop VM

### Restart VM

### Auto-Shutdown VM

### Delete VM

## Delete Resources





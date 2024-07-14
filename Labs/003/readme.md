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

Once you have logged in to the virtual machine, you can install and configure software applications. When you are finished, you can close the SSH session by simply running the ```exit``` command. 

### Troubleshoot

## Understand Virtual Machine Images
The Azure Marketplace has a list of virtual machine images that you can use when creating your VM. Earlier, we created a virtual machine using the Ubuntu image. You can run the following command to list of popular VM images available to use in table format: 

```az vm image list --output table```

The command returns the following output: 

```output
Architecture    Offer                         Publisher               Sku                                 Urn                                                                             UrnAlias                 Version
--------------  ----------------------------  ----------------------  ----------------------------------  ------------------------------------------------------------------------------  -----------------------  ---------
x64             CentOS                        OpenLogic               8_5-gen2                            OpenLogic:CentOS:8_5-gen2:latest                                                CentOS85Gen2             latest
x64             Debian11                      Debian                  11-backports-gen2                   Debian:debian-11:11-backports-gen2:latest                                       Debian-11                latest
x64             flatcar-container-linux-free  kinvolk                 stable-gen2                         kinvolk:flatcar-container-linux-free:stable-gen2:latest                         FlatcarLinuxFreeGen2     latest
x64             opensuse-leap-15-4            SUSE                    gen2                                SUSE:opensuse-leap-15-4:gen2:latest                                             OpenSuseLeap154Gen2      latest
x64             RHEL                          RedHat                  8-lvm-gen2                          RedHat:RHEL:8-lvm-gen2:latest                                                   RHELRaw8LVMGen2          latest
x64             sles-15-sp3                   SUSE                    gen2                                SUSE:sles-15-sp3:gen2:latest                                                    SLES                     latest
x64             0001-com-ubuntu-server-jammy  Canonical               22_04-lts-gen2                      Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest                    Ubuntu2204               latest
```

To deploy a virtual machine using a specific image, you need to use the value under the ***Urn*** or ***Alias*** columns. When specifying the `--image`, you can replace the version number with 'latest' which selects the latest version of the distribution. 

In the command example below, we are creating a windows virtual machine, and using the `--image` parameter to specify the latest version available. 

```
az vm create \
--resource-group rg-VM2-01 \
--name myWindowsVM \
--image MicrosoftWindowsServer:WindowsServer:2022-Datacenter:latest \
--admin-username azureuser \
--generate-ssh-keys \
 --public-ip-sku Standard \
```
## Understand Virtual Machine Sizes 


## Manage Virtual Machine 

### Start VM

### Stop VM

### Restart VM

### Auto-Shutdown VM

### Delete VM

## Delete Resources





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

* ***Note:*** *The above example creates a resource group named rg-VM-01 in the (Europe) UK West location. You can replace the variable values to a resource group name that you will easily remember, and an Azure datacenter location that is close to you.* 

### Create Ubuntu VM
Next, let's create an Ubuntu virtual machine. I will break down each part of the command to aid your understanding. 

3. Run the following command to create an Ubuntu VM:

```
az vm create \
--resource-group rg-VM-01 \
--name myUbuntuVM01 \
--image Ubuntu2204 \
--admin-username azureuser \
--generate-ssh-keys \
--public-ip-sku Standard
```

```rg-VM-01```: This is the resource group we created in the previous step. 

```myUbuntuVM01```: Give your virtual machine a name that makes sense to you. 

```Ubuntu2204```: We have opted for Canonical Ubuntu 22.04 LTS, which is the latest (Long Term Support) version release. 

```azureuser```: This is your username. Make it something memorable. 

```--generate-ssh-keys```: This takes care of secure access.

```--public-ip-sku Standard```: We are using the "Standard" verion for the public IP. 

It may take a few minutes to create the virtual machine. Once the Ubuntu VM has been successfully created, you should see the following Azure CLI output which contains information about the VM. Take note of the `publicIpAddress`, as you will use this to access the virtual machine: 

```output
{
  "fqdns": "",
  "id": "/subscriptions/f2c7d4w6-8m7b-0000-0000-000000000000/resourceGroups/rg-VM-01/providers/Microsoft.Compute/virtualMachines/myUbuntuVM01",
  "location": "ukwest",
  "macAddress": "00-3D-4A-88-6A-95",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "51.141.81.189",
  "resourceGroup": "rg-VM-01",
  "zones": ""
}
```

### Test Ubuntu VM
To check whether the newly created virtual machine is reachable, we must run the ```ping``` command. Think of it as testing the connection to your new virtual computer in the cloud. 

4. Run the following command to test your VM:
```ping public_ip_address```

Remember to replace <public_ip_address> with the **public IP address** of your virtual machine. If you see responses, it means you are connected. Once you have confirmed that your virtual machine is reachable, you can press `ctrl`+`c` to exit. 

## Connect to Virtual Machine
5. Next, open a SSH connection to the created virtual machine with the following syntax:
``` ssh <username>@<public_ip_address>```.

Replace <username> with your chosen **username**, and <public_ip_address> with the **public IP address** of your virtual machine. 

In the example command from step 3, we chose ***azureuser*** as our username. If you used that username, your command would look like the following: ```azureuser@<public_ip_address>```. 

* ***Important:*** *The first time you attempt to log onto a Linux virtual machine, you may get a warning about adding the server fingerprint to a list of known hosts. In this case, I am okay with proceeding, so I select "yes".*

<img width="861" alt="connect ssh" src="https://github.com/user-attachments/assets/2a15505c-6855-424e-8ac0-28f3692007cf">

Once you have logged in to the virtual machine, you can install and configure software applications. When you are finished, you can close the SSH session by simply running the ```exit``` command. 

### Install Nginx Web Server
Ensure you are logged in and connected to your virtual machine via `ssh`, then run the following commands: 

```
sudo apt-get update
sudo apt-get install nginx
```

Next: 
1. Paste the VMs public IP address in a browser of your choice; and/or
2. Run the following command: `curl -m 10 <public_ip_address>`

  * Note: The `curl` command is making a basic GET request to retrieve data (the default web page) from the server at the specified public IP address.

You should notice that this command **fails**. 

<img width="757" alt="fail nginx access" src="https://github.com/user-attachments/assets/5c770628-b3b5-45d1-9653-cf87ad9afcf3">

<img width="774" alt="fail access web server" src="https://github.com/user-attachments/assets/cf3f3c13-81af-4c58-a3d6-c1a8b5806bde">

* This is because Azure Network Security Groups (NSGs) have default rules that block all incoming traffic. To allow HTTP traffic to your web server, you need to configure the NSG to open port 80.

Exit the SSH session and run the following `az vm open-port` command: 

```
az vm open-port \
--resource-group rg-VM-01 \
--name myUbuntuVM01 \
--port 80
```

* Note: Since Azure CLI is not installed or configured on your VM itself, you won't be able to run `az` commands there.

And just like that, we can now view the default welcome page of our Nginx web server by: 

1. Choosing a web browser of your choice typing the public IP address of the VM as the web address; and/or
2. Running our curl command once again.

<img width="849" alt="successful nginx access" src="https://github.com/user-attachments/assets/0ca44e8c-3353-436e-a839-6e4d5b926ce8">

<img width="849" alt="successful web srver access" src="https://github.com/user-attachments/assets/dce4e619-526c-4acd-a730-be46a5f6e049">

Congratulations! You have successfully created an Ubuntu 22.04 LTS, installed an Nginx web server, and updated NSG rules to open port 80, all via Azure CLI. 

## Troubleshoot
When deploying virtual machines in Azure, you may encounter a couple of common issues. Below, I'll guide you through the process of dealing with them. 

### Error 1: 'Remote Host Identification Has Changed'
Sometimes, when connecting to a computer using SSH, you may see an error that looks something like this: 

```output
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
5d:0b:85:12:c7:ab:02:10:4a:ab:4g:f1:94:ab:e5:3c.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending key in /home/user/.ssh/known_hosts:2
RSA host key for <domain> has changed and you have requested strict checking.
Host key verification failed.
```

This error occurs when the public key of the host changes. Whenever you connect to a computer using SSH, the key you used to connect is stored in a file called `known_hosts`, located in your `.ssh` home directory. You can open the file with the text editor of your choice. I'm going to be using `vim`. 

```
vim ~/.ssh/known_hosts
```

Let's walk through your options to resolve this issue: 

1. **Option 1: Remove the 'Known_Hosts' File**

If the known_hosts file only contains one host, then deleting the file may be a solution. Note: A new **known_hosts** file will be created the next time you SSH into a computer. Before removing the file, you should back up the contents:

```
cp ~/.ssh/known_hosts ~/.ssh/known_hosts.old
rm ~/.ssh/known_hosts
```

2. **Option 2: Remove the Offending Host Key Individually**

Look at the original error message. You may notice the following line:

`RSA host key for <domain_name> has changed...`

This points out the offending host, so you can now remove that key by running: 

```
ssh-keygen -R hostname
```

Alternatively, you can edit the known_hosts file directly using a text editor of your choice (e.g. nano or vim), and remove the offending line which is indicated in the original error message: **Offending key in /home/user/.ssh/known_hosts:2.** In this example, the offending key is on line 2. Make sure that you back up the file before deleting anything in case you accidentally delete something you did not intend to, by running: 

```
cp ~/.ssh/known_hosts ~/.ssh/known_hosts.old
```

### Error 2: 'Passphrase' Prompt for SSH Keys 
If you are prompted for a passphrase when creating your virtual machine and you did not set one, it may be because an existing SSH key is being reused. 

The `--generated-ssh-keys` parameter will use existing keys if the `id_rsa` and `id_rsa.pub` files already exist in your `.ssh` home directory, and will not create new ones. Let's walk through your options to resolve this issue:

1. **Option 1: Remove the Existing SSH Key Pair**
   
If the computer/s that used the existing SSH key pair no longer exist, removing the existing key pair files and creating a new machine using the `--generate-ssh-keys` option may be a solution. You should back up the file contents before removing the key files:

```
cp ~/.ssh/id_rsa ~/.ssh/id_rsa_old
cp ~/.ssh/id_rsa.pub ~/.ssh/id_rsa_old.pub
rm ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```

The, create your virtual machine using the `--generate-ssh-keys` option: 

```
az vm create \
--resource-group rg-VM-01 \
--name myUbuntuVM01 \
--image Ubuntu2204 \
--admin-username azureuser \
--generate-ssh-keys \
--public-ip-sku Standard
```

2. **Option 2: Create New SSH Pair With a Different Filename**

You can generate a new key pair with a different filename to avoid conflicts with existing keys using the `ssh-keygen` command. To generate the new key pair, run:

```
ssh-keygen -t rsa -b 2048 -f ~/.ssh/new_id_rsa
```

```-t rsa```: Specifies the type of key to create with the value being "rsa". 

```-b 2048```: Specifies the number of bits in the key to create, using the default of 2048.

```-f ~/.ssh/new_id_rsa```: Specifies the filename "new_id_rsa" of the key file we are creating. 

* Note: Do not enter a passphrase when prompted.

Next, use the newly created public key to create your virtual machine, using the `--ssh-key-value` option: 

```
az vm create \
--resource-group rg-VM-01 \
--name myUbuntuVM01 \
--image Ubuntu2204 \
--admin-username azureuser \
--ssh-key-value ~/.ssh/new_id_rsa.pub \
--public-ip-sku Standard
```

3. **Option 3: Reset the SSH Key**

If you are unable to access the virtual machine due to forgetting a passphrase, you can reset the SSH key using the `az vm user update` command. Here's how you do it:

First, generate a new SSH key pair by running:

```
ssh-keygen -t rsa -b 2048 -f ~/.ssh/new_id_rsa
```

This will create a new SSH key in ~/.ssh/new_id_rsa

Then run the following command to reset the SSH key for your virtual machine:

```
az vm user update \
  --resource-group rg-VM-01 \
  --name myUbuntuVM01 \
  --username azureuser \
  --ssh-key-value ~/.ssh/new_id_rsa.pub
```

* Note: The `az vm user update` command will add the new public key to the `~/.ssh/authorized_keys` file for the specified user on the virtual machine, without removing any existing keys. As such, you should access your VM and remove the old public key for security and to ensure that *only* the new key is used for future connections.

Next, use your newly created key to SSH into your VM: 

```
ssh -i ~/.ssh/new_id_rsa azureuser@public_ip_address
```

Once connected to your virtual machine, open the `~/.ssh/authorized_keys` file on the VM using a text editor of your choice and remove the old key with the passphrase: 

```
vim ~/.ssh/authorized_keys
```

You can cross-reference old public key by running the following on your local machine:

```
cat ~/.ssh/id_rsa.pub
```

Note: Remember to save and exit the ~/.ssh/authorized_keys file you edited. 

Finally, back up and replace the old key files on your local machine. You can then rename your newly created key to ~/.ssh/id_rsa to ensure `ssh` finds it automatically. You can do this easily by running:

```
mv ~/.ssh/id_rsa ~/.ssh/id_rsa_old
mv ~/.ssh/id_rsa.pub ~/.ssh/id_rsa_old.pub
mv ~/.ssh/new_id_rsa ~/.ssh/id_rsa
mv ~/.ssh/new_id_rsa.pub ~/.ssh/id_rsa.pub
```

4. **Option 4: Replace SSH Key Pair**

If you have existing machines using an old key pair with a compromised passphrase, you can replace the key pair with a new one. 

First, generate a new key by running: 

```
ssh-keygen -t rsa -b 2048 -f ~/.ssh/new_id_rsa
```

This will create a new SSH key in ~/.ssh/new_id_rsa

Next, make a list of all machines that are accessible using the old / existing SSH key that we want to replace. Then, connect to each of those machines and remove the old key from all machines that have it using a text editor of your choice: 

```
ssh azureuser@public_ip_address
vim .ssh/authorized_keys
```

Editing the `~/.ssh/authorized_keys` file in your machine/s:

1. Ensure you remove the line containing your old SSH key. This will be simple if the file only contains one line. However, if it contains multiple lines, then look for the line that ends with the same cryptic letters as your old public key. You can view the old public key by running the following on your local machine:

```cat ~/.ssh/id_rsa.pub```: Assuming that your old private key is in 'id_rsa'. 
   
2. Add the new public key to the `authorized_keys` file. You can find and view the contents of your new public key file on your local machine by running:

```
cat ~/.ssh/new_id_rsa.pub
```

Copy the entire line of the `~/.ssh/new_id_rsa.pub` file to the `authorized_keys` file. Then save and exit the file by pressing `esc`+`:wq`,`Enter`. 

Test whether you can access your machine with your new key without closing your existing connection so that you are still connected in case something goes wrong. Run the following on your local machine: 

```
ssh -i ~/.ssh/new_id_rsa.pub azureuser@public_ip_address
```

* Note: Remember to replace the values of the username and VM public IP address variables.

If you are able to connect to your VM using your new keys, this means you have successfully replaced your SSH key on that machine. Now, repeat the process for the next machine, and so on. 

Once you have successfully replaced your old key, you can now rename your newly created key to ~/.ssh/id_rsa to ensure `ssh` finds it automatically. Make sure you back up your old keys in case you forgot about a system that still requires the old key for access. To do so, simply run:  

```
mv ~/.ssh/id_rsa ~/.ssh/id_rsa_old
mv ~/.ssh/id_rsa.pub ~/.ssh/id_rsa_old.pub
mv ~/.ssh/new_id_rsa ~/.ssh/id_rsa
mv ~/.ssh/new_id_rsa.pub ~/.ssh/id_rsa.pub
```

In the event you have forgotten a machine that is still using the old key, simply run the following to use it: 

```
ssh -i ~/.ssh/id_rsa_old
```

* Note: Don't forget to replace the old key in that machine with your new key. 

## Understand Virtual Machine Images
The Azure Marketplace has a list of virtual machine images that you can use when creating your VM. Earlier, we created a virtual machine using the Ubuntu image. You can run the following command to list popular VM images available to use in table format: 

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
--resource-group rg-VM-02 \
--name myWindowsVM \
--image MicrosoftWindowsServer:WindowsServer:2022-Datacenter:latest \
--admin-username azureuser \
--generate-ssh-keys \
--public-ip-sku Standard 
```
## Understand Virtual Machine Sizes 
When you create an Azure virtual machine, its size determines how much compute power it gets, including CPY, GPU and memory. It is therefore essential to choose a size that aligns with your expected workload. If your needs change, you can resize the virtual machine later. 

### Virtual Machine Sizes 
Below is a breakdown of different VM sizes and their use cases: 

| Type                      |    Description       |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [General purpose](../sizes-general.md)         | Balanced CPU-to-memory. Ideal for dev/test environments, small to medium applications, and data solutions.  |
| [Compute optimised](../sizes-compute.md)    | High CPU-to-memory. Suitable for medium traffic applications, network appliances, and batch processing.        |
| [Memory optimised](../sizes-memory.md)     | High memory-to-core ratio. Great for relational databases, medium to large caches, and in-memory analytics.                 |
| [Storage optimised](../sizes-storage.md)      | High disk throughput and IO. Perfect for Big Data, SQL, and NoSQL databases. |
| [GPU](../sizes-gpu.md)          | Specialised VMs for heavy graphic rendering and video editing.       |
| [High performance](../sizes-hpc.md) | Our most powerful CPU VMs with optional high-throughput network interfaces (RDMA). |

### Find Available Virtual Machine Sizes 
To see a list of VM sizes available in a specific region, run the `az vm list-sizes` command. Here's how to do it:

```
az vm list-sizes --location ukwest --output table 
```
Example partial output: 

```output
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
4                   8192          Standard_D2ds_v4           2                1047552           76800
8                   16384         Standard_D4ds_v4           4                1047552           153600
16                  32768         Standard_D8ds_v4           8                1047552           307200
32                  65536         Standard_D16ds_v4          16               1047552           614400
32                  131072        Standard_D32ds_v4          32               1047552           1228800
32                  196608        Standard_D48ds_v4          48               1047552           1843200
32                  262144        Standard_D64ds_v4          64               1047552           2457600
4                   8192          Standard_D2ds_v5           2                1047552           76800
8                   16384         Standard_D4ds_v5           4                1047552           153600
16                  32768         Standard_D8ds_v5           8                1047552           307200
32                  65536         Standard_D16ds_v5          16               1047552           614400
32                  131072        Standard_D32ds_v5          32               1047552           1228800
32                  196608        Standard_D48ds_v5          48               1047552           1843200
32                  262144        Standard_D64ds_v5          64               1047552           2457600
32                  393216        Standard_D96ds_v5          96               1047552           3686400
```

### Create a Virtual Machine with a Specific Size

```
az vm create \
--resource-group rg-VM-03 \
--name myUbuntuVM02 \
--image Ubuntu2204 \
--size Standard_D2ds_v4 \
--admin-username azureuser \
--generate-ssh-keys \
--public-ip-sku Standard
```

### Resize a Virtual Machine
Once your virtual machine has been deployed, you may want to resize it to adjust its resource allocation. To view the current size of a VM, run the `az vm show` command: 

```
az vm show --resource-group rg-VM-03 --name myUbuntuVM02 --query hardwareProfile.vmSize
```

Next, before resizing the virtual machine, we need to view the list of available VM sizes on the hardware cluster where the VM is hosted

```
az vm list-vm-resize-options --resource-group rg-VM-03 --name myUbuntuVM02 --query [].name --output table
```
* Note: The above command lists VM sizes for a specific VM `myUbuntuVM02` in the resource group `rg-VM-03` region. 

To list the available sizes for **all** VMs in the resource group, run: 

```
az vm list-vm-resize-options --ids $(az vm list --resource-group rg-VM-03 --query "[].id" --output tsv)
```

* Note: Remember to replace the resource group value with the name you chose for your resource group.

If the desired size is available, you can resize the virtual machine while it's powered on, but it will reboot during the operation. Use the `az vm resize` command:

```
az vm resize --resource-group rg-VM-03 --name myUbuntuVM02 --size Standard_D3_v2
```

If the desired size is not listed or available on the current Azure cluster, you will need to deallocate the virtual machine before resizing. This can be done by using the `az vm deallocate` command. 

* Note: when you power the VM back on, any data on the temporary disk may be lost, and the IP address could change unless you assigned a static IP to the virtual machine.

To deallocate the virtual machine, run: 
```
az vm deallocate --resource-group rg-VM-03 --name myUbuntuVM02 
```

Once the virtual machine is deallocated, you can proceed with resizing it:
```
az vm resize --resource-group rg-VM-03 --name myUbuntuVM02 --size Standard_D12_v2
```

Finally, after resizing it, you can restart the virtual machine again:
```
az vm start --resource-group rg-VM-03 --name myUbuntuVM02
```

## Virtual Machine Power States
An Azure virtual machine can go through various different states. These states represent the last known state of the VM. 

### Power States

| Power State | Description | Billing
|----|----|----|
| Creating | The virtual machine is allocating resources. | Not billed* |
| Starting | The virtual machine is being started. | Billed |
| Running | The virtual machine is up and running. | Billed |
| Stopping | The virtual machine is being stopped. | Billed |
| Stopped | The virtual machine is stopped. Virtual machines in the stopped state still incur compute charges.  | Billed |
| Deallocating | The virtual machine is being deallocated. | Not billed* |
| Deallocated | The virtual machine is removed from the hypervisor but still available in the control plane. Virtual machines in the Deallocated state do not incur compute charges. | Not billed* |

### Find the Power State
To check the power state of a specific virtual machine, run the `az vm get-instance-view` command with the relevant resource group and VM name: 

```
az vm get-instance-view \
--resource-group rg-VM-03 \
--name myUbuntuVM02 \
--query 'instanceView.statuses[1]' --output table 
```

Example output:

```
Code                Level    DisplayStatus
------------------  -------  ---------------
PowerState/running  Info     VM running
```

To get the power state of **all** VMs in a resource group, run: 

```
az vm get-instance-view --ids $(az vm list --resource-group rg-VM-03 --query "[].id" --output tsv) --query "[].{VMName:name, Code:instanceView.statuses[1].code, Level:instanceView.statuses[1].level, DisplayStatus:instanceView.statuses[1].displayStatus}" --output table
```

Example output: 

```
VMName        Code                Level    DisplayStatus
------------  ------------------  -------  ---------------
myUbuntuVM01  PowerState/running  Info     VM running
myUbuntuVM02  PowerState/running  Info     VM running
myUbuntuVM03  PowerState/running  Info     VM running
```

To get the power state of **all** VMs in subscription, run: 

```
az vm get-instance-view --ids $(az vm list --query "[].id" --output tsv) --query "[].{VMName:name, Code:instanceView.statuses[1].code, Level:instanceView.statuses[1].level, DisplayStatus:instanceView.statuses[1].displayStatus}" --output table
```

Example output: 

```
VMName        Code                    Level    DisplayStatus
------------  ----------------------  -------  ---------------
myUbuntuVM01  PowerState/deallocated  Info     VM deallocated
myUbuntuVM02  PowerState/deallocated  Info     VM deallocated
myUbuntuVM03  PowerState/deallocated  Info     VM deallocated
myUbuntuVM04  PowerState/running      Info     VM running
```

## Manage Virtual Machine 
Throughout the life cycle of a virtual machine, you may need to perform management tasks like starting, stopping or deleting the VM to improve performance and minimise costs. The Azure CLI simplifies VM management from the command line and allows you to automate these tasks with scripts. 

### Get the IP Address
The following command allows you to capture the public and private IP address of your VM. This can be useful if you forgot to capture the IP address earlier, or if the IP address has changed. 

```
az vm list-ip-addresses --name myUbuntuVM01 --output table
```

Example output: 

```output 
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
myUbuntuVM01      51.141.81.189        10.0.0.4
```
### Stop VM
Stop a virtual machine using the following command: 

```
az vm stop --resource-group rg-VM-01 --name myUbuntuVM01
```

Stop **all** VMs in a resource group: 

```
az vm stop --ids $(az vm list --resource-group rg-VM-01 --query "[].id" --output tsv)
```

Stop **all** VMs in subscription:

```
az vm stop --ids $(az vm list --query "[].id" -o tsv)
```

* Note: Remember you can verify the VM state and ensure it has in fact stopped by using the `az vm get-instance-view` command that we covered earlier.

**Stop:** Pauses VM operation but continues billing for compute resources and storage (OS disk). Useful for temporary stops when you plan to restart the VM soon and intend to retain all VM data and configurations. 

### Start VM
Start a specific stopped virtual machine in a select resource group with the following command: 
```
az vm start --resource-group rg-VM-01 --name myUbuntuVM01 
```

Start **all** VMs in a resource group: 

```
az vm start --ids $(az vm list --resource-group rg-VM-01 --query "[].id" --output tsv)
```

Start **all** VMs in subscription:

```
az vm start --ids $(az vm list --query "[].id" -o tsv)
```

### Restart VM
Restarting your virtual machine is just as simple. Run the following command: 

```
az vm restart --resource-group rg-VM-01 --name myUbuntuVM01
```

Retart **all** VMs in a resource group: 

```
az vm restart --ids $(az vm list --resource-group rg-VM-01 --query "[].id" --output tsv)
```

Restart **all** WMs in subscription:

```
az vm restart --ids $(az vm list --query "[].id" -o tsv)
```

### Auto-Shutdown VM
Schedule your virtual machine to automatically shutdown at a specific time to minimise costs: 

```
az vm auto-shutdown --resource-group rg-VM-01 --name myUbuntuVM01 --time 1930
```

Auto-shutdown **all** VMs in a resource group: 

```
az vm auto-shutdown --ids $(az vm list --resource-group rg-VM-01 --query "[].id" -o tsv) --time 1930 --output table
```

Auto-shutdown **all** VMs in subscription: 

```
az vm auto-shutdown --ids $(az vm list --query "[].id" -o tsv) --time 1930 --output table
```

**Auto-Shutdown:** Schedules shutdown to stop VM operation at specified times. Still billed for compute resources until shutdown. You continue to pay for storage (OS and data disks attached to the VM). Great for ensuring VMs are not left running unnecessarily and automating shutdowns to reduce compute costs without manual intervention. 

### Deallocate VM 
Deallocate (stop) your virtual machine and pause billing for compute resources:

```
az vm deallocate --resource-group rg-VM-01 --name myUbuntuVM01
```

Deallocate **all** VMs in a resource group: 

```
az vm deallocate --ids $(az vm list --resource-group rg-VM-01 --query "[].id" --output tsv)
```

Deallocate **all** VMs in subscription:

```
az vm deallocate --ids $(az vm list --query "[].id" -o tsv)
```

**Deallocate:** Stops VM completely. Halts billing for compute resources but is still charged for storage (OS and data disks attached to the VM). Suitable for more long-term cost management strategies where VMs may be inactive for extended periods, essentially pausing compute costs until needed again. 

### Delete VM
Delete your virtual machine using the following command: 

```
az vm delete --resource-group rg-VM-01 --name myUbuntuVM01 --yes --no-wait
```
* `--yes`: Confirms your intention to delete without prompting you again for confirmation.
* `--no-wait`: Returns terminal control immediately without having to wait for completion, once the operation is initiated.

Delete **all** VMs in a resource group: 

```
az vm delete --ids $(az vm list --resource-group rg-VM-01 --query "[].id" --output tsv)
```

Delete **all** VMs in subscription: 

```
az vm delete --ids $(az vm list --query "[].id" --output tsv)
```

## Delete Resources
It's worth noting that using the `az vm delete` command will only delete the virtual machine itself and not its associated network and disk resources.

When the virtual machine is no longer needed, you can delete the resource group, virtual machine, and all related resources to avoid unnecessary costs. To do this, run:

```
az group delete --name rg-VM-01 --yes --no-wait
```

Finally, you can verify the status of the resource group deletion by running: 

```
az group list --output table
```

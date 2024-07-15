![banner](https://github.com/user-attachments/assets/326faa93-410e-4af4-9a51-3e88cfa92897)

In a previous lab, I detailed the process of creating a virtual machine via the Azure portal. Below, I am going to walk you through a method of streamlining the process of deploying a virtual machine by using an ARM (Azure Resource Manager) Quickstart template in the portal. ARM templates allow users to define the desired configuration of an Azure virtual machine, making it easier to manage and reproduce their infrastructure while reducing errors, saving time, and ensuring consistency. 

## Requirements 
1. **Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/

2. **Quickstart Template:** Go to [Azure Quickstart Templates GitHub Repository](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-simple-linux). The link will direct you to a suitable template (called **'101-vm-simple-linux**) for deploying an Ubuntu virtual machine. 

## Create an Ubuntu Virtual Machine Using an ARM Template
1. Sign in to the [Azure Portal](https://portal.azure.com/)
2. Next, go to the Azure Quickstart Templates GitHub Repository to locate the simple [Linux VM Quickstart Template](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-simple-linux).
3. Select the **Deploy to Azure** button.
<img width="1136" alt="github arm template" src="https://github.com/user-attachments/assets/758728ba-84f2-405a-8092-c09633cedac9">

***Note:*** *This action should bring you to the below page where you can edit parameters to customise and deploy your virtual machine.* 
<img width="748" alt="optional deploy banner " src="https://github.com/user-attachments/assets/78548b80-902d-4a40-8d58-bb3b0380e965">

 4. Under **Project details**, ensure you select the subscription that you wish to charge the cost of resources against. Then, either select an existing resource group or create a new one.
<img width="744" alt="project details" src="https://github.com/user-attachments/assets/92923c35-ec5d-4b5c-a91c-48316a50025b">

5. Under **Instance details**, you will need to:
   * **Select Azure Region:** Select the geographical location where the virtual machine's resources will be physically hosted, such as **UK South, West US, etc.** Consider factors such as, region service availability and capacity, region proximity, and local compliance requirements when selecting a region.
   * **Name Virtual Machine:**. It should be a globally unique name across the Azure network.
   * **Name Administrator:** Enter unique username.
   * **Select Authentication Type:** Choose either Password or SSH Public Key. I have opted to use SSH keys, as it is best practise for secure access.
   * **SSH Public Key Source:** Leave the default option **Generate new key pair**.
   * **Key Pair Name:** Enter a key pair name.
<img width="775" alt="Instance details" src="https://github.com/user-attachments/assets/3e39ccdc-caec-4850-ac8d-19c589278351">


*Optionally, you can leave the following parameters as default or select the size you need for your virtual machine. Since this virtual machine is just a demo, I have adjusted the default **Vm Size** and opted for something small (Standard B1s, 1GB RAM), and changed the **Security Type** to 'Standard' to minimise costs.*
<img width="752" alt="instance details part 2" src="https://github.com/user-attachments/assets/2798e1b2-e819-4a65-bce9-e89e93563378">

6. Select the **Review + create** button at the bottom of the page.
7. Next, you should be able to see details about the virtual machine you are about to create. Review your settings and make any adjustments where necessary. After validation has passed, when you are ready, select **Create** and wait for your machine to be fully deployed.
8. A **Generate new key pair** window will open. Select **Download + create**. Your key file will download with a  ***.pem*** extension. Remember where the ***.pem*** file was downloaded as you will need the file path in the next step.
9. When the deployment is complete, select **Go to resource**.
In the resource group, you will see a list of different resources created:
* Virtual machine
* SSH key
* Operating system disk
* Network interface
* Public IP address for the VM
* Network security group SSH (port 22)
* Virtual network
<img width="986" alt="resources" src="https://github.com/user-attachments/assets/93f6387c-b0cb-4a8c-ab29-e83d60ba4731">

On the page for your new virtual machine, select the public IP address and copy it to the clipboard.
<img width="1075" alt="linux ARM vm" src="https://github.com/user-attachments/assets/09cae3f0-7d57-4d24-9d9d-fdd7eb0f57d0">

## Connect to Virtual Machine
Create an **SSH connection** with the virtual machine. 

1. Open your terminal of choice.
2. If you are using Mac or Linux, run the following bash command to set a read-only permission on the ***.pem*** file using ```chmod 400 path/to/keyfile.pem ```. If you are using a Windows machine, open a PowerShell prompt.
<img width="646" alt="permissions" src="https://github.com/user-attachments/assets/08e5afa4-e5b1-4e7b-aed5-819dc2453030">

3. Next, open a SSH connection to the created virtual machine with the following syntax:
``` ssh -i /path/to/keyfile.pem <user name>@<IP address>```.

***Important:*** *The first time you attempt to log onto a Linux virtual machine, you may get a warning about adding the server fingerprint to a list of known hosts. In this case, I am okay with proceeding, so I select "yes".*
<img width="861" alt="connect to vm" src="https://github.com/user-attachments/assets/9634aca8-30b6-404d-97c5-abb5a8901de0">


## Delete Resources
When the virtual machine is no longer needed, you can delete the resource group, virtual machine and all related resources to avoid unnecessary costs. To do this, you need to: 

1. Select the **Resource group** link on the Overview page for the virtual machine.
2. Select **Delete resource group** at the top of the page for the resource group.
3. A page will open with a warning that you are about to delete resources. Type the name of the resource group and select **Delete** to complete the deletion. 

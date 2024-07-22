![Deploy](https://github.com/user-attachments/assets/7d1216c8-1fd3-4b8f-bc38-6de91a14e5cd)

In this write-up, I'm going to walk you through the process of creating an Azure Virtual Network using Bastion, via the Azure Portal. We will associate the virtual network with two virtual machines, use Bastion to securely connect to the VMs from the internet using Secure Shell (SSH), and enable private communication between the VMs. 

But first, let's take a quick look at what an Azure Virtual Network is. 

<img width="876" alt="VN-resources" src="https://github.com/user-attachments/assets/e1142b3a-565b-494d-95da-6b5dc602f934">


## What is Azure Virtual Network? 
Azure Virtual Networks are the backbone of private networking in Azure, enabling secure and seamless communication between Azure resources like virtual machines, web apps and databases, and with external resources and on-premises systems. Unlike traditional wired networks, Azure Virtual Networks connect devices, servers and data centers through software and wireless technology, offering the ability to: 

1. **Isolation and segmentation:** Create isolated virtual networks with private IP address spaces that are not internet routable.
2. **Internet communications:** Enable public internet access to Azure resources by assigning public IP addresses to resources or using public load balancers.
3. **Communicate between Azure resources:** Securely connect Azure resources like Azure Virtual Machine Scale Sets, Azure Kubernetes Service (AKS), App Service Environments, Azure SQL databases, and storage accounts using virtual networks and service endpoints for optimal security and routing.
4. **Communicate with on-premises resources:** Link on-premises networks to Azure using VPNs or ExpressRoute for secure, high-bandwidth connections
5. **Route network traffic:** Control traffic flow between subnets and networks with custom route tables and Border Gateway Protocol (BGP)
6. **Filter network traffic:** Use Network Security Groups (NSGs) and Network Virtual Appliances, such as firewalls and wide area network (WAN) optimisers, to define security rules and optimise network functions.
7. **Connect virtual networks:** Link virtual networks using peering, enabling private communication and creating a global interconnected network.

Essentially, an Azure Virtual Network extends on-premises networks to the cloud, providing a secure and scalable environment for your cloud resources. 

## Requirements 
**Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/

## Create an Azure Virtual Network & an Azure Bastion Host
1. Sign in to the [Azure Portal](https://portal.azure.com/)

2. Search for and select *virtual networks* in the portal.

3. On the **virtual networks** page, select + **Create** and the **Create a virtual network** page should open.
<img width="1001" alt="virtual networks" src="https://github.com/user-attachments/assets/ae42d03e-0041-4e27-8f15-8d6a86d0cd04">

4. On the **Basics** tab, under **Project details**, select the subscription that you wish to charge the cost of resources against. Then, either select an existing resource group or create a new one.
<img width="1004" alt="project details" src="https://github.com/user-attachments/assets/44bc5aff-5aa4-4c1b-9b43-a5f5b4660f56">

5. Under **Instance details**, you will need to:
   * **Name Virtual Network:**. The name must be unique within a resource group, but can be duplicated within a subscription or Azure Region.
   * **Select Azure Region:** Select the geographical location where the virtual network’s compute resources will be physically hosted such as **UK South, West US, etc.** Consider factors such as region service availability, network latency, regional compliance requirements and proximity to other resources you plan to deploy. This choice may impact network performance and availability of Azure services.
<img width="1000" alt="vnet instance details" src="https://github.com/user-attachments/assets/dbe08939-d435-44c5-929c-9cc35b2b0b4d">

6. Select **Next** to proceed to the **Security** tab.

7. In the **Azure Bastion** section, select **Enable Bastion**.

* **Note:** Bastion uses your browser to connect to VMs in your virtual network over Secure Shell (SSH) or Remote Desktop Protocol (RDP) by using their private IP addresses. The VMs don't need public IP addresses, client software, or special configuration.

8. Under **Azure Bastion**, enter an Azure Bastion host name, and create an Azure Bastion public IP address, then select **OK**.
<img width="986" alt="bastion security tab" src="https://github.com/user-attachments/assets/129461a4-cb4f-44b7-8f43-7883132b92d3">

9. Select **Next** to proceed to the **IP Addresses** tab.

10. Navigate to the **default** subnet in the address space box, and select the edit symbol. A page called **Edit subnet** should open.

11. In **Edit subnet**, do the following:
    * **Subnet purpose:** Leave the subnet purpose as **Default**.
    * **Name:** Name your subnet, e.g. subnet-001.
    * **IPv4 address range:** Leave the default of **10.0.0.0/16**.
    * **Starting address:** Leave the default of **10.0.0.0**.
    * **Size:** Leave the default of **/24** (256 addresses).
      
![address-subnet-space](https://github.com/user-attachments/assets/a8836522-17ab-486e-8765-2ebc85a430b5)

12. Select **Save**.

13. Select **Review + create** at the bottom of the window. When validation passes, select **Create**.

## Create Virtual Machines
In the following steps, I will guide you through the process of creating two virtual machines named **vm-001** and **vm-002**, and associating them with the virtual network you set up earlier. 

1. In the portal, search for and select **Virtual machines**.

2. On the **Virtual machines** page, select + **Create++ and then select **Azure virtual machine**.
<img width="1090" alt="create VM-001" src="https://github.com/user-attachments/assets/718e10c4-f4ba-47b6-93bd-8d89b407d434">

3. On the **Basics** tab, under **Project details**, select the subscription that you wish to charge the cost of resources against. Then, either select an existing resource group or create a new one.
<img width="1004" alt="project details" src="https://github.com/user-attachments/assets/44bc5aff-5aa4-4c1b-9b43-a5f5b4660f56">

4. Under **Instance details**, you will need to:
   * **Name Virtual Machine:**. It should be a globally unique name across the Azure network.
   * **Select Azure Region:** Select the geographical location where the virtual machine's resources will be physically hosted, such as **UK South, West US, etc.** Consider factors such as, region service availability and capacity, region proximity, and local compliance requirements when selecting a region.
   * **Select Availability Option:** You can use the default option or select an option for your individual or business needs. Since this virtual machine is just a demo, high availability and fault tolerance is not necessary. In this case, the *'No infrastructure redundancy required'* option offers a striaghtforward and cost effective option that demonstrates functionality.
   * **Select OS Image:** Choose the operating system you need for your virtual machine. I chose *Ubuntu Server 20.04 LTS - x64 Gen2*.
<img width="895" alt="VM-001 instance details" src="https://github.com/user-attachments/assets/f47aab6e-dfb6-48d2-aa3f-cbfcabd76e66">

5. **Select Size:** Select the size you need for your virtual machine. Since this is just a demo, I have opted for something small (Standard B1s, 1GB RAM) to minimise costs.
<img width="890" alt="VM-001 size" src="https://github.com/user-attachments/assets/e0ee0d65-037c-43e1-8fb6-760806469839">

6. Under **Administrator account**, you will need to:
* **Select *SSH public key*** as your authentication type. I have opted to use SSH keys instead of passwords, as it is best practise for secure access.
* **Name Administrator:** Enter unique username.
* **SSH Public Key Source:** Leave the default option **Generate new key pair**.
* **Key Pair Name:** Leave autogenerated key pair name, or enter your own.
<img width="922" alt="VM-001 admin account" src="https://github.com/user-attachments/assets/3a524813-b36c-47ff-b5f2-fc94412562f9">

7. Under **Inbound port rules**, choose **Allow selected ports**, and then select **SSH (22)**.
<img width="918" alt="inbound port rules" src="https://github.com/user-attachments/assets/b48007b0-1d74-4a82-95b6-1c23921a4f82">

8. Select the **Networking** tab.

9. Under **Network interface**, do the following:
    * **Virtual network:** Select virtual network, e.g. **vnet-001**.
    * **Subnet:** Select **subnet-001 (10.0.0.0/24)**.
    * **Public IP:** Select of **None**.
    * **NIC network security group:** Select **Advanced**.
    * **Configure network security group:** Select **Create new**. Enter **vm-001-nsg** for the network security group name. Leave the rest at defaults and select **OK**.
<img width="926" alt="VM-001 networking tab" src="https://github.com/user-attachments/assets/63976325-2a58-4985-a034-4fa781725f12">


10. Leave the rest of the settings at the defaults and select **Review + create**.

11. You should be able to see details about the virtual machine you are about to create. Review your settings and make any adjustments where necessary. When you are ready, select **Create**. Your virtual machine should be ready in seconds.

12. A **Generate new key pair** window will open. Select **Download private key and create resource**. Your key file will download with a  ***.pem*** extension. Remember where the ***.pem*** file was downloaded as you will need the file path in the next step.

13. Wait for the first virtual machine **vm-001** to deploy then repeat the previous steps to create a second virtual machine **vm-002** with the following settings:

|Setting |Value |
|----|----|
|Virtual machine name	|Enter **vm-002**. |
Virtual network	|Select **vnet-001**. |
Subnet	|Select **subnet-001 (10.0.0.0/24)**. |
Public IP	|Select **None**. |
NIC network security group	|Select **Advanced**. |
Configure network security group	|Select **vm-001-nsg**. |

* **Note:** Above, we will use the same **Network Security Group (NSG)** for multiple VMs since they share the same security requirements which will help to simplify management and ensure consistent security rules across those VMs.
  
* **Also Note:** Virtual machines in a virtual network with an Azure Bastion host don't need public IP addresses. Bastion provides the public IP, allowing the VMs to use private IPs for communication within the network. You can remove the public IPs from any VMs in Bastion-hosted virtual networks.

## Connect to Virtual Machines
SSH (Secure Shell) into the VMs in your virtual network using Bastion. 

1. In the portal, search for and select **Virtual machines**. You should see your two newly created VMs, **vm-001** and **vm-002**. 
<img width="1277" alt="VMs" src="https://github.com/user-attachments/assets/ff8c7d28-d25c-4fd7-98f7-a0b3d2739ba0">

2. On the **Virtual machines** page, select **vm-001**.

3. In the **Overview** information for **vm-001**, select **Connect**.

4. On the **Connect to virtual machine** page, select the **Bastion** tab.

5. Select **Use Bastion**.

6. Enter the following:
   * **Authentication Type:** Select **SSH Private Key from Local File**.
   * **Username:** Enter **azureuser**.
   * **Local File:** Locate and select the downloaded **vm-001** key file.
   * **Advanced:** Ensure the *'Open in new browser tab' box is checked.* 

8. Select **Connect** at the botton of the window.
   * **Note:** Once you press connect, a browser-based terminal should open with an established connection to your VM using the provided username and SSH key.   

## Start Communication Between Virtual Machines
1. Run the following command in the browser terminal for **vm-001** to test the connection:
   * `ping -c 5 vm-002`.

     * **Note:** You should see something similar to the following message:
     
       <img width="929" alt="VM-001 ping command" src="https://github.com/user-attachments/assets/22354357-6637-4277-90b4-e746587dafca">

2. Close the Bastion connection to **vm-001** using the **exit** command. 

3. Repeat the steps in [Connect to Virtual Machines](https://github.com/MiticaMatthews/Azure-Labs/edit/main/Labs/004/readme.md#connect-to-virtual-machines) to connect to **vm-002**. 

4. For **vm-002**, run the following command in the browser terminal to test the connection:
   * `ping -c 5 vm-001`

     * **Note:** You should see something similar to the following message:
    
       <img width="976" alt="VM-002 ping command" src="https://github.com/user-attachments/assets/329ca63f-07d1-4ca1-9b23-16d2f7f09829">

5. Close the Bastion connection to **vm-002** using the **exit** command.

## Delete Resources

When the resources are no longer needed, you can delete the resource group and all its resources by doing the following: 

1. In the Azure portal, search for and select **Resource groups**.
2. On the **Resource groups** page, select the **rg-VN-001** resource group.
3. On the **rg-VN-001** page, navigate to the top of the page and select **Delete resource group**.
4. A page will open with a warning that you are about to delete resources. Type the name of the resource group (**rg-VN-001** in this example) and then select **Delete** to complete the deletion. 

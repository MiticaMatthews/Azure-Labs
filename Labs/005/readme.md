![rnt banner](https://github.com/user-attachments/assets/b4f39e86-612b-480e-9cbf-cd6c52e9e90f)

In this write-up, I will walk you through the process of routing network traffic with a route table using the Azure portal. This lab will show you how to create **User-Defined Routes (UDRs)** to direct traffic between subnets through a **Network Virtual Appliance (NVA)**. As such, any outbound traffic will first be routed to the NVA before reaching the Private subnet.

By the end of this write-up, you will have learned to: 
1. Create a virtual network and subnets.
2. Create an NVA that routes traffic.
3. Deploy virtual machines (VMs) into different subnets.
4. Create a route table.
5. Create a route.
6. Associate a route table to a subnet.
7. Route traffic from one subnet to another through an NVA.

## What is Azure Routing
Routing is the backbone of IP connectivity, essential for linking various services and enabling secure access to private networks. Azure provides two main types of routes: **System routes**, which are automatically created and assigned to subnets and cannot be removed, and **User-Defined Routes (UDRs)**, which offer custom routing options to override default routes and tailor traffic flow to meet specific needs. This tutorial will focus on **UDRs**, but understanding both default and custom routes is crucial for effectively managing and securing network traffic in Azure.

## Requirements 
**Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/.

## Create a Virtual Network & an Azure Bastion Host
1. Sign in to the [Azure Portal](https://portal.azure.com/).

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

## Create Subnets
At the end of this section, you will have set up three subnets within your virtual network (vnet-001): a public subnet (subnet-001, created in previous steps), a Demilitarized Zone (DMZ) subnet, and a Private subnet. The Public subnet (subnet-001) will host and allow access to services and applications over the internet. The DMZ subnet is where you deploy the Network Virtual Appliance (NVA), which acts as a firewall or router to manage and secure traffic between networks. The Private subnet is where you deploy virtual machines that need to be secure, typically containing internal resources and sensitive business data. Traffic between the public and private subnets is routed through the NVA, ensuring security by filtering and managing traffic flow. This setup helps maintain a secure boundary between external and internal networks, protecting your data and resources. 

1. Navigate to the **Virtual networks** page in the Azure portal.
2. Select your previously created virtual network, **vnet-001**.
3. In **vnet-001**, select **Subnets** under the **Settings** section.
4. Select + **Subnet**.
5. In **Add Subnets**, enter the following information for your Private subnet:
   * **Name:** Enter a name for your Private subnet, e.g. **subnet-private**.
   * **Subnet Address Range:** Enter **10.0.2.0/24.**
6. Select **Save**.
<img width="843" alt="add subnet-private" src="https://github.com/user-attachments/assets/04a41acb-81cb-4cbe-8485-3cee04bbb0e2">

7. Select + **Subnet**.
8. In **Add Subnets**, enter the following information for your DMZ subnet:
   * **Name:** Enter a name for your Private subnet, e.g. **subnet-dmz**.
   * **Subnet Address Range:** Enter **10.0.3.0/24.**
9. Select **Save**.
<img width="847" alt="add subnet-dmz" src="https://github.com/user-attachments/assets/f1f6a09f-1198-44e9-a38b-141eba23bf71">

You should see the following **Subnets** in your **vnet-001** virtual network: 

<img width="955" alt="subnet list" src="https://github.com/user-attachments/assets/193ab70e-87df-4a8e-88eb-ef5af2847cb5">

**Note:** When deploying a Bastion host, Azure automatically creates a dedicated Bastion subnet to securely facilitate remote access to your virtual machines without needing public IP addresses. We will use Bastion to connect to the VMs later. 

## Create an Network Virtual Appliance (NVA) Virtual Machine
Network Virtual Appliances (NVAs) are virtual machines that perform specific network functions, such as routing and firewall optimisation. In this section, we will create a NVA using an **Ubuntu 22.04** virtual machine. 

1. In the portal, search for and select **Virtual machines**.

2. On the **Virtual machines** page, select + **Create** and then select **Azure virtual machine**.
<img width="1090" alt="create VM-001" src="https://github.com/user-attachments/assets/718e10c4-f4ba-47b6-93bd-8d89b407d434">

3. On the **Basics** tab, under **Project details**, select your subscription. Then, select the resource group (**rg-VN-001**) that you used to create your virtual network **vnet-001**.
<img width="1004" alt="project details" src="https://github.com/user-attachments/assets/44bc5aff-5aa4-4c1b-9b43-a5f5b4660f56">

4. Under **Instance details**, you will need to:
   * **Name Virtual Machine:** Enter **vm-nva**.
   * **Select Azure Region:** Select the geographical location where the virtual machine's resources will be physically hosted, such as **UK South, West US, etc.** Use the same region selected for your resource group.
   * **Select Availability Option:** You can use the default option or select an option for your individual or business needs. Since this virtual machine is just a demo, high availability and fault tolerance is not necessary. In this case, the *'No infrastructure redundancy required'* option offers a striaghtforward and cost effective option that demonstrates functionality.
   * **Select OS Image:** Select **Ubuntu Server 22.04 LTS - x64 Gen2**.
     <img width="869" alt="vm-nva instance details" src="https://github.com/user-attachments/assets/b5fa0fe7-07f3-4a02-b751-0076b1310488">


5. **Select Size:** Select the size you need for your virtual machine. Since this is just a demo, I have opted for something small (Standard B1s, 1GB RAM) to minimise costs.
<img width="890" alt="VM-001 size" src="https://github.com/user-attachments/assets/e0ee0d65-037c-43e1-8fb6-760806469839">

6. Under **Administrator account**, you will need to:
* **Select *SSH public key*** as your authentication type. I have opted to use SSH keys instead of passwords, as it is best practise for secure access.
* **Name Administrator:** Enter unique username.
* **SSH Public Key Source:** Leave the default option **Generate new key pair**.
* **Key Pair Name:** Leave autogenerated key pair name, or enter your own.
  <img width="913" alt="vm-nva admin account" src="https://github.com/user-attachments/assets/439fca5b-5c7e-4c32-a595-2ea20ea56b1e">

7. Under **Inbound port rules**, select **None** since we will be accessing the VM via Bastion.
   <img width="905" alt="inbound port rules nva" src="https://github.com/user-attachments/assets/56aef237-d8f0-4b16-a820-32976925f741">


8. Select **Next: Disks** then **Next: Networking**.

9. Under **Network interface**, do the following:
    * **Virtual network:** Select virtual network, e.g. **vnet-001**.
    * **Subnet:** Select **subnet-dmz (10.0.3.0/24)**.
    * **Public IP:** Select of **None**.
    * **NIC network security group:** Select **Advanced**.
    * **Configure network security group:** Select **Create new**. Enter **vm-nsg-nva** for the network security group name. Leave the rest at defaults and select **OK**.
      <img width="902" alt="vm-nva network int" src="https://github.com/user-attachments/assets/7bec9d36-2dec-4212-8473-5efc76159995">

10. Leave the rest of the settings at the defaults and select **Review + create**.

11. You should be able to see details about the virtual machine you are about to create. Review your settings and make any adjustments where necessary. When you are ready, select **Create**. Your virtual machine should be ready in seconds.
12. A **Generate new key pair** window will open. Select **Download private key and create resource**. Your key file will download with a  ***.pem*** extension. Remember where the ***.pem*** file was downloaded as you will need the file path later.

## Create Public & Private Virtual Machines 
In this section, we will create two virtual machines in the **vnet-001** virtual network. One virtual machine will be placed in the **subnet-001** subnet, and the other in the **subnet-private** subnet. Make sure you use the same **Ubuntu 22.04** virtual machine image for both virtual machines. 

### Create Public Virtual Machine
The public virtual machine is used to simulate a machine in the public internet. The public and private virtual machine are used to test the routing of network traffic through the NVA virtual machine.

1. In the portal, search for and select **Virtual machines**.

2. On the **Virtual machines** page, select + **Create** and then select **Azure virtual machine**.
3. In **Create a virtual machine**, enter or select the following information in the **Basics** tab:

|Setting	|Value |
|----|----|
|Project details |
|Subscription	|Select your subscription. |
|Resource group	|Select **rg-VN-001**. |
|Instance details | 
|Virtual machine name	|Enter **vm-public**. |
|Region	|Select **(Europe) UK South**. |
|Availability options	|Select **No infrastructure redundancy required**. |
|Security type	|Select **Standard**. |
|Image	|Select **Ubuntu Server 22.04 LTS - x64 Gen2**. |
|VM architecture	|Leave the default of **x64**. |
|Size	|Select **Standard_B1s - 1 vcpu, 1 GiB memory ($8.61/month) (free services eligible)**. |
|Administrator account |
|Authentication type	|Select **SSH public key**. |
|Username	|Enter a username e.g. **azureuser**. |
|Key pair name	|Use pre-generated key pair name or enter key pair name. |
|Inbound port rules |	
|Public inbound ports	|Select **None**. |

4. Select **Next: Disks** then **Next: Networking**.
5. In the **Networking** tab, enter or select the following information:

    |Setting	|Value |
    |----|----|
    |Network interface |
    |Virtual network	|Select **vnet-001**. |
    |Subnet	|Select **subnet-001** (**10.0.0.0/24**). |
    |Public IP	|Select **None**. |
    |NIC network security group	|Select **None**. |

6. Leave the rest of the options at the defaults and select **Review + create**.
7. Select **Create**.
8. A **Generate new key pair** window will open. Select **Download private key and create resource**. Your key file will download with a  ***.pem*** extension. Remember where the ***.pem*** file was downloaded as you will need the file path later.

### Create Private Virtual Machine 
1. In the portal, search for and select **Virtual machines**.

2. On the **Virtual machines** page, select + **Create** and then select **Azure virtual machine**.
3. In **Create a virtual machine**, enter or select the following information in the **Basics** tab:

|Setting	|Value |
|----|----|
|Project details |
|Subscription	|Select your subscription. |
|Resource group	|Select **rg-VN-001**. |
|Instance details | 
|Virtual machine name	|Enter **vm-private**. |
|Region	|Select **(Europe) UK South**. |
|Availability options	|Select **No infrastructure redundancy required**. |
|Security type	|Select **Standard**. |
|Image	|Select **Ubuntu Server 22.04 LTS - x64 Gen2**. |
|VM architecture	|Leave the default of **x64**. |
|Size	|Select **Standard_B1s - 1 vcpu, 1 GiB memory ($8.61/month) (free services eligible)**. |
|Administrator account |
|Authentication type	|Select **SSH public key**. |
|Username	|Enter a username e.g. **azureuser**. |
|Key pair name	|Use pre-generated key pair name or enter key pair name. |
|Inbound port rules |	
|Public inbound ports	|Select **None**. |

4. Select **Next: Disks** then **Next: Networking**.
5. In the **Networking** tab, enter or select the following information:

    |Setting	|Value |
    |----|----|
    |Network interface |
    |Virtual network	|Select **vnet-001**. |
    |Subnet	|Select **subnet-private** (**10.0.2.0/24**). |
    |Public IP	|Select **None**. |
    |NIC network security group	|Select **None**. |

6. Leave the rest of the options at the defaults and select **Review + create**.
7. Select **Create**.
8. A **Generate new key pair** window will open. Select **Download private key and create resource**. Your key file will download with a  ***.pem*** extension. Remember where the ***.pem*** file was downloaded as you will need the file path later.

## Enable IP Forwarding
To route traffic through the NVA, turn on IP forwarding both in Azure and in the operating system of vm-nva. When IP forwarding is enabled, vm-nva will forward any incoming traffic destined for another IP address to its correct destination, rather than dropping it.

### Enable IP Forwarding in Azure 
In this section, we will turn on IP forwarding for the network interface of the **vm-nva** virtual machine. 

1. In the portal, search for and select **Virtual machines**.
2. In **Virtual machines**, select **vm-nva**.
3. In **vm-nva**, select **Networking settings** under the **Networking** section.
4. Select the name of the interface next to **'Network Interface:'**. The name begins with **vm-nva** and has a random number assigned to the interface. The name of the interface in this example is **vm-nva430**.

   <img width="1054" alt="vm-nva network settings" src="https://github.com/user-attachments/assets/575bbab8-de4e-448e-9ebb-ec895208d7da">

5. In the network interface overview page, select **IP configurations**.
6. In **IP configurations**, select the box next to **Enable IP forwarding**.
7. Select **Apply**. 
 
    <img width="963" alt="vm-nva enable ip forwarding" src="https://github.com/user-attachments/assets/c1b801e0-b538-4d18-b1dc-3933728a67f1">

### Enable IP Forwarding in the Operating System
In this section, we will enable IP forwarding in the operating system of the **vm-nva** virtual machine to allow it to forward network traffic, and use the **Azure Bastion** service to securely connect to the **vm-nva** virtual machine.

1. In the portal, search for and select **Virtual machines**.
2. In **Virtual machines**, select **vm-nva**.
3. Select **Bastion** in the **Connect** section.
4. Select **SSH Private Key from Local File** for your **Authentication Type**.
5. Enter your username, e.g. **azureuser**.
6. Locate the SSH key file for the **vm-nva** virtual machine in your local directory.
7. Select **Connect**.
   
   * **Note:** Once you press connect, a browser-based terminal should open with an established connection to your VM using the provided username and SSH key.

     <img width="754" alt="vm-nva enable bastion" src="https://github.com/user-attachments/assets/42b00946-8071-452c-a69b-f45f5e1898a2">

8. Run the following commands in your browser terminal to enable IP forwarding:
   * Activate IP forwarding immediately:
     
     `sudo sysctl -w net.ipv4.ip_forward=1`

   * Ensure the change is persistent across reboots:
     
     `echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf`

   * Apply changes made to the configuration file immediately:
     
     `sudo sysctl -p` 

## Create a Route Table 
In this section, we will create a route table to define the route of the traffic through the **vm-nva** virtual machine. The route table is associated to the **subnet-001** subnet where the **vm-public** virtual machine is deployed.

1. In the portal, search for and select **Route tables**.
2. Select + **Create**.
3. In **Create a Route table**, enter or select the following information:
   |Setting	|Value |
   |----|----|
   |Project details |
   |Subscription	|Select your subscription. |
   |Resource group	|Select **rg-VN-001**. |
   |Instance details | 
   |Region	|Select **(Europe) UK South**. |
   |Name	|Enter **route-table-public**. |
   |Propagate gateway routes	|Leave the default of **Yes**. |

4. Select **Review + create**.
5. Select **Create**. 

## Create a Route
In this section, we will create a route in the route table that you created in the previous steps. 

1. In the portal, search for and select **Route tables**.
2. Select **route-table-public**.
3. In **Settings** select **Routes**.
4. Select + **Add** in **Routes**.
5. Enter or select the following information in **Add route**:

   |Setting	|Value |
   |----|----|
   |Route name|	Enter **to-private-subnet**. |
   |Destination type|	Select **IP Addresses**. |
   |Destination IP addresses/CIDR ranges|	Enter **10.0.2.0/24**. |
   |Next hop type|	Select **Virtual appliance**. |
   |Next hop address|	Enter **10.0.3.4**. |

   **Note:** `10.0.3.4` is the IP address of the **vm-nva** virtual machine you created in the earlier steps.

   <img width="828" alt="route" src="https://github.com/user-attachments/assets/ab7723ed-0d44-4e5a-bd57-7feb11764c93">

6. Select **Add**.
7. Select **Subnets** in **Settings**.
8. Select + **Associate**.
9. Enter or select the following information in **Associate** subnet:

  |Setting	|Value |
  |----|----|
  |Virtual network	|Select **vnet-001 (rg-VN-001)**. |
  |Subnet	|Select **subnet-001**. |
 
  <img width="580" alt="Associate subnet" src="https://github.com/user-attachments/assets/2282ffdf-56e8-4dd7-8337-54449ea63514">

10. Select **OK**. 

## Test Routing of Network Traffic 
Next, we will test routing. 

### Test Network Traffic from public-VM to private-VM. 
1. In the portal, search for and select **Virtual machines**.
2. In **Virtual machines**, select **vm-public**.
3. Select **Bastion** in the **Connect** section.
4. Select **SSH Private Key from Local File** for your **Authentication Type**.
5. Enter your username, e.g. **azureuser**.
6. Locate the SSH key file for the **vm-public** virtual machine in your local directory.
7. Select **Connect**.
   * **Note:** Once you press connect, a browser-based terminal should open with an established connection to your VM using the provided username and SSH key.
8. In the terminal, run the following command to trace the routing of network traffic from vm-public to vm-private:
   `tracepath vm-private`

   The response is similar to the following example:
   ```
   azureuser@vm-public:~$ tracepath vm-private
   1?: [LOCALHOST]                      pmtu 1500
   1:  vm-nva.internal.cloudapp.net                          0.806ms
   1:  vm-nva.internal.cloudapp.net                          1.008ms
   2:  vm-private.internal.cloudapp.net                      1.980ms reached
   Resume: pmtu 1500 hops 2 back 1 
   ```

    **Note:** In the tracepath output, you’ll notice two hops when ICMP traffic travels from **vm-public** to **vm-private**. The first hop is **vm-nva**, and the second hop is **vm-private**. This indicates that Azure routed the traffic from **subnet-001** through the NVA to reach **subnet-private**, as specified by the **to-private-subnet** route in **route-table-public**.

9. Close the Bastion connection. 

### Test Network Traffic from private-VM to public-VM.
1. In the portal, search for and select **Virtual machines**.
2. In **Virtual machines**, select **vm-private**.
3. Select **Bastion** in the **Connect** section.
4. Select **SSH Private Key from Local File** for your **Authentication Type**.
5. Enter your username, e.g. **azureuser**.
6. Locate the SSH key file for the **vm-private** virtual machine in your local directory.
7. Select **Connect**.
   * **Note:** Once you press connect, a browser-based terminal should open with an established connection to your VM using the provided username and SSH key.
8. In the terminal, run the following command to trace the routing of network traffic from vm-public to vm-private:
   `tracepath vm-public`

   The response is similar to the following example:
   ```
   azureuser@vm-private:~$ tracepath vm-public
   1?: [LOCALHOST]                      pmtu 1500
   1:  vm-public.internal.cloudapp.net                       1.890ms reached
   1:  vm-public.internal.cloudapp.net                       2.508ms reached
   Resume: pmtu 1500 hops 1 back 2 
   ```

     **Note:** In the tracepath output above, there’s a single hop to **vm-public**, which shows that traffic went directly from **subnet-private** to **subnet-001**. By default, Azure routes traffic directly between subnets without additional routing rules.
  
9. Close the Bastion connection. 

## Delete Resources 
When the resources that you created are no longer needed, you can delete the resource group and all its resources by doing the following: 

1. In the Azure portal, search for and select **Resource groups**.
2. On the **Resource groups** page, select the **rg-VN-001** resource group.
3. On the **rg-VN-001** page, navigate to the top of the page and select **Delete resource group**.
4. A page will open with a warning that you are about to delete resources. Type the name of the resource group (**rg-VN-001** in this example) and then select **Delete** to complete the deletion. 

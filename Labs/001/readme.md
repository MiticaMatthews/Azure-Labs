![banner](https://github.com/user-attachments/assets/ef6e3b2a-e9c7-47d1-a39e-ec6d169e302b)

## What is Azure Virtual Machine?

## Requirements 
**Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/

## Create an Ubuntu Virtual Machine 
1. Sign in to the [Azure Portal](https://portal.azure.com/)
2. Navigate to the Virtual machines resource on the home page, or alternatively, enter *virtual machines* in the search bar.
<img width="1064" alt="virtual machines" src="https://github.com/user-attachments/assets/3ba2ebe5-fe7b-4e21-8ec5-ba29a9e7fed9">

3. In the **Virtual machines** page, select **Create** and then **Virtual machine**. The **Create a virtual machine** page should open.
<img width="1063" alt="create vm" src="https://github.com/user-attachments/assets/8393150b-3ede-4ff5-8292-8ca640281874">

4. In the **Basics** tab, under **Project details**, ensure you select the subscription that you wish to charge the cost of resources against. Then, either select an existing resource group or create a new one.
<img width="795" alt="project details" src="https://github.com/user-attachments/assets/0a9287cb-7d42-4791-a329-d248d676d17b">

5. Under **Instance details**, you will need to:
   * **Name Virtual Machine:**. It should be a globally unique name across the Azure network.
   * **Select Azure Region:** Select the geographical location where the virtual machine's resources will be physically hosted, such as **UK South, West US, etc.** Consider factors such as, region service availability and capacity, region proximity, and local compliance requirements when selecting a region.
   * **Select Availability Option:** You can use the default option or select an option for your individual or business needs. Since this virtual machine is just a demo, high availability and fault tolerance is not necessary. In this case, the *'No infrastructure redundancy required'* option offers a striaghtforward and cost effective option that demonstrates functionality.
   * **Select OS Image:** Choose the operating system you need for your virtual machine. I chose *Ubuntu Server 20.04 LTS - x64 Gen2*.
<img width="823" alt="instance details" src="https://github.com/user-attachments/assets/d577bac4-21fe-49c3-9296-8f2679b31a46">






   
## Connect to Virtual Machine

## Delete Resources

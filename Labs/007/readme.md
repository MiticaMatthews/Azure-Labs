![Azure P2S banner](https://github.com/user-attachments/assets/00e9b7cd-5589-474a-996a-f671d03ca59e)

In this write-up, I will walk you through the process of configuring a Point-to-Site (P2S) VPN connection for a macOS machine using the Azure Portal, and Bash CLI (for certificate generation and management). We will use certificates to authenticate the connection between your device and the Azure Virtual Network.

By the end, you will have learned how to:

1. Create a virtual network.
2. Create a gateway subnet within the virtual network.
3. Set up a virtual network gateway.
4. Generate self-signed certificates.
5. Download and configure the VPN client on your macOS machine.

## What is an Azure Point-to-Site (P2S) VPN?
A Point-to-Site (P2S) VPN allows users to securely connect their computer (the "point") to an Azure Virtual Network (the "site"). This enables private access to resources in the network from anywhere in the world.

## Requirements 
**Azure Account:** If you don't already have an Azure subscription, create a free account using the following link: https://azure.microsoft.com/en-gb/free/.

## Create a Virtual Network 
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

6. Select the **Next** tab to proceed to the **Security** tab. For this lab, we will leave the default values for all services on this page.

   **Note:** It's worth noting that *Azure DDoS Basic Protection* is integrated into the Azure platform at no additional cost. *Azure DDoS Standard Protection* is a premium paid service that comes with advanced capabilities to protect users against network attacks such as logging, alerting and telemetry. 

7. Select the **Next** tab to proceed to the **IP Addresses** tab.

8. Navigate to the **default** subnet in the address space box, and select the edit symbol. A page called **Edit subnet** should open.

9. In **Edit subnet**, do the following:
    * **Subnet purpose:** Leave the subnet purpose as **Default**.
    * **Name:** Name your subnet, e.g. subnet-001.
    * **IPv4 address range:** Leave the default of **10.0.0.0/16**.
    * **Starting address:** Leave the default of **10.0.0.0**.
    * **Size:** Leave the default of **/24** (256 addresses).
      
<img width="1458" alt="edit subnet" src="https://github.com/user-attachments/assets/26cd4c36-fde4-4a93-a872-893aa08741a8">


10. Select **Save**.

11. Select **Review + create** at the bottom of the window. When validation passes, select **Create**.

## Create a Gateway Subnet
To successfully configure a VPN (whether point-to-site (P2S) or site-to-site (S2S)), a gateway subnet is required. 

1. Navigate to the newly created virtual network.

2. Select **Subnets** on the left panel under **Settings**.

3. At the top of the page, select  **+ Gateway subnet** to open the **Add subnet** pane.

4. Leave the values provided as default. Do not add any network security group to the gateway subnet. Select **Save** at the bottom of the page to save the subnet.

## Create the VPN Gateway
A VPN (Virtual Private Network) gateway is a key component of creating secure connections between on-premises networks and cloud virtual networks, like Azure virtual networks. It encrypts data and safely sends it across the public Internet.

1. Navigate to the homepage and search for **Virtual Network Gateway** in the portal, and select **Create**.

2. On the **Basics** tab, fill in the values for **Project details** and **Instance details**.

![create vnet gateway](https://github.com/user-attachments/assets/66f1baf3-17b2-40e6-a89a-c8d2c917a934)

  * **Subscription:** Select the subscription that you wish to charge the cost of resources against.
  * **Resource Group:** This setting should be autofilled when you select your virtual network on this page. If not, select the resource group associated with your virtual network, e.g. `rg-vnet-001`.
  * **Name:** Name your gateway. Note: naming your gateway is *not* the same as naming your *gateway subnet*.
  * **Region:** Ensure that your region is the same as the virtual network.
  * **Gateway Type:** Select **VPN**. VPN gateways use the virtual network gateway type VPN.
  * **SKU:** Select the gateway SKU that supports the features you need. See: [Gateway SKUs](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsku).
  * **Generation:** Select the relevant generation. Azure recommends using a **Generation2 SKU**.
  * **Virtual Network:** Select the virtual network you wish to add to this gateway. Ensure you have selected the correct subscription and regions in the previous settings if you cannot see the virtual network you intend to use for this gateway.
  * **Gateway Subnet:** You should see the address range for the **GatewaySubnet** created in earlier steps for your virtual network. A gateway subnet is necessary to set up a VPN gateway. If you don't see a gateway subnet or the option to create one on this page, go back to your virtual network and create it before returning here to configure the VPN gateway.
    
3. Fill in the values for the **Public IP address** that will be associated to the VPN gateway.
   * **Public IP Address:** Select **Create new**.
   * **Public IP Address Name:** Enter a name for your public IP address instance.
   * **Assignment:** The assignment option is typically autoselected. For the Standard SKU, assignment is always Static.
   * **Availability Zone:** Select **Zone-redundant**.  
   * **Enable active-active mode:** Select **Disabled**. Only enable this setting if you're creating an active-active gateway configuration.
   * **Configure BGP:** Select **Disabled**, unless your configuration specifically requires this setting.

4. Select **Review + create** to run validation.

5. After validation passes, select **Create** to deploy the VPN gateway.

## Generate Certificates
Virtual network gateways can take up to 45 minutes to create. In the meantime, let's generate the self signed certificates required to connect to the VPN. It's worth noting that you can use the enterprise provided certificates as well. 

1. Install the required packages. In this lab, we will be using homebrew to install these packages.
   ```
   brew install strongswan
   brew install openssl
   ```
   
   * **Note: **If you don't already have homebrew installed on your machine, see: [Homebrew](https://brew.sh/) for guidance on how to install the homebrew package manager for macOS. If you already have homebrew installed on your machine, ensure that it is up-to-date before installing the above packages.

2. Next, generate the CA certificate.

```
ipsec pki --gen --outform pem > caKey.pem
ipsec pki --self --in caKey.pem --dn "CN=VPN CA" --ca --outform pem > caCert.pem
```

This will generate the `caCert.pem` in your current working directory. Next, we need to opten the certificate in Base64 encoded format so that we can copy the certificate and paste it to the virtual network gateway. 

```
openssl x509 -in caCert.pem -outform der | base64
```

You should now see the certificate details. Copy the output and paste it in any text editor of your choice for later use: 

3. Once the virtual network gateway has successfully deployed, navigate to the resource and on the left panel, select **Point-to-site configuration** under the settings tab. Select **Configure now** and then add the certificate to the gateway.

  * **Address Pool:** Enter the IP address range of your choice, e.g. `172.18.0.0/24`.
  * **Tunnel Type:** Select `IKEv2` for macOS.
  * **Authentication Type:** Set to **Azure certificate**.
  * **Root Certificates:** In the **Name** section, enter your chosen name. Then, under **Public certificate data**, copy and paste the certificate you previously saved in a text file or editor.

    <img width="1234" alt="Point-to-site configuration" src="https://github.com/user-attachments/assets/83b87d3f-c2dc-4a2e-ac41-ce6a57f25be9">

4. Select **Save**.

5. Next, we need to generate the certificate that we will use for the VPN client on our machine. To do this, run the following:

```
export PASSWORD="test123"
export USERNAME="user001"

ipsec pki --gen --outform pem > "${USERNAME}Key.pem"
ipsec pki --pub --in "${USERNAME}Key.pem" | ipsec pki --issue --cacert caCert.pem --cakey caKey.pem --dn "CN=${USERNAME}" --san "${USERNAME}" --flag clientAuth --outform pem > "${USERNAME}Cert.pem"
```

  * **Note:** You can use any password and/or username of your choice. Ensure it is unique and memorable as you will need your password of choice later.
  * **Note:** Once you have executed the above commands, two files will be created `clientCert.pem` and `clientKey.pem`. You can check this by running the following command in your terminal to view all folders and files in your current working directory:

    ```
    ls
    ```

6. Next, we need to generate a p12 bundle containing the user certificate. This bundle will be used in the next steps when working with the client configuration files. Run the following command:

```
openssl pkcs12 -in "${USERNAME}Cert.pem" -inkey "${USERNAME}Key.pem" -certfile caCert.pem -export -out "${USERNAME}.p12" -password pass:${PASSWORD} -legacy
```

  * **Note:** This step will create a `client.p12` file in your current working directory. It's also worth noting that if someone else wishes to use the VPN tunnel that you've set up, then you will need to give them this p12 bundle generated alone, along with the password that you set in the previous step.
    
  * **Additional Note for macOS Users:** Make sure to include the `-legacy` flag when generating the .p12 certificate with OpenSSL to maintain compatibility with systems using LibreSSL, such as Apple Keychain. Without this flag, the certificate may not be readable, even with the correct password.

## Configure the VPN Client
The next step is to configure VPN client on your client machine. If you will be using a p12 bundle generated by someone else, then you will need to request the associated zip file **(VPN client)**. 

**Otherwise, follow the below steps:**

1. Navigate to the **Virtual Network Gateway** page in the portal, and select **point-to-site configuration** on the left panel under **Settings** tab.

2. At the top of the **Point-to-site configuration** page, select **Download VPN client** to download a zip file. It may take a few minutes for the client configuration package to generate. You may not see any indications until the packet generates. Once complete, your browser should indicate that a client configuration zip file is available and it will have the same name as your gateway.

3. Unzip the file on your machine. You should see the following contents in the zip file.
   <img width="779" alt="VPN client" src="https://github.com/user-attachments/assets/67901ea0-e834-44a7-9be3-22178fe841cc">


4. Before we configure the VPN client, we need to add the certificate to our machine's root. To do this, navigate to and double-click on the **p12** file and you should be prompted for the password.

   * **Note:** This file should be in your local user directory: search `/users/<username>` in the search bar on your desktop. The password is the one set in previous steps, e.g. `test123`. Select **OK**. You should now be able to see the certificate added to the root certificates in Apple Keychain.

5. Next, navigate to **System Settings > Network** on your machine and select **Add VPN Configuration**, and then **IKEv2**.

6. Locate your downloaded gateway zip file and select the **Generic** folder, and then open the **VpnSettings.xml** file. Next, copy the **VpnServer** tag value and paste this value in the **Server Address** and **Remote ID** fields of the profile.
   
    <img width="696" alt="VPN configuration" src="https://github.com/user-attachments/assets/a72559d2-c52e-407a-9c72-498112f88a13">

7. Under the **User Authentication** settings, select **Certificate**.

8. Next, choose the client certificate that you wish to use for authentication. This is the certificate that we imported earlier.

9. Enter the name of the selected certificate under **Local ID**.

10. Select **Connect**. to start the P2S connection.

    * **Note:** You may be prompted to enter your machine login password.

      <img width="702" alt="login password prompt" src="https://github.com/user-attachments/assets/6cd0876d-3fb3-40d5-9d2b-5d5cee09b3d6">
      
       <img width="701" alt="vpn connection success" src="https://github.com/user-attachments/assets/07ad862d-054e-4a90-b196-3917a407aa28">

Congratulations! You have successfully connected to the VPN. You can now access resources in the virtual network just as if you were on-site.

## Delete Resources 
When the resources that you created are no longer needed, you can delete the resource group and all its resources by doing the following: 

1. In the Azure portal, search for and select **Resource groups**.
2. On the **Resource groups** page, select the **rg-VN-001** resource group.
3. On the **rg-VN-001** page, navigate to the top of the page and select **Delete resource group**.
4. A page will open with a warning that you are about to delete resources. Type the name of the resource group (**rg-VN-001** in this example) and then select **Delete** to complete the deletion. 

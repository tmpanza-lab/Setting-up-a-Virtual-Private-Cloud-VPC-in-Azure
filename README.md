# Setting up a Virtual Private Cloud (VPC) in Azure

## Objective

The objective of this lab exercise is to set up a secure and functional Virtual Private Cloud (VPC) in Microsoft Azure. This involves creating an isolated network environment to host resources like virtual machines (VMs), while implementing security measures, controlled access, and network connectivity to simulate a real-world cloud infrastructure.

### Skills Learned

- Gained hands-on experience with Azure's networking and security features.
- Foundational skill in designing secure, scalable cloud infrastructure.
- Implementing an isolated and controlled internet access for workloads.
- Security and Access Control
- Virtual Machine (VM) Deployment and Management

### Tools Used

Azure Portal
Azure CLI

## Steps

*Ref 1: Network Diagram*
![image](https://github.com/user-attachments/assets/1b251143-6101-4eed-996a-a63ce795035e)

Step 1: Create a Resource Group

A Resource Group is a container that holds related resources for your project.
• Azure Portal: Go to Resource groups > Create > Choose a name and region.
![image](https://github.com/user-attachments/assets/187c2120-4a08-482d-9d71-e6be64d34219)
• Azure CLI: Run: Copy code
az group create --name MyResourceGroup --location eastus
![image](https://github.com/user-attachments/assets/ae3bb735-7d4a-4bf7-a7ac-6a0d2c373f0f)
![image](https://github.com/user-attachments/assets/0bdd8050-7e51-4eca-b140-ca021a163f47)

Step 2: Set Up a Virtual Network (VNet)

1.	Create the VNet:

o Portal: Navigate to Virtual networks > Create.
![image](https://github.com/user-attachments/assets/fb48a103-d976-40f4-ad90-5edd1c8da1a5)

o Set the name, region, and address space (e.g., 10.0.0.0/16).
2. Create Subnets:
o Create one public subnet (e.g., 10.0.1.0/24) for internet-accessible resources.
o Create a private subnet (e.g., 10.0.2.0/24) for resources that should be isolated. 
![image](https://github.com/user-attachments/assets/48627767-0101-4e5d-9ad2-3c41dceaff3c)


o CLI: Copy code
az network vnet create \
--resource-group MyResourceGroup \
--name MyVNet \
--address-prefix 10.0.0.0/16 \
--subnet-name PublicSubnet \
--subnet-prefix 10.0.1.0/24 az network vnet subnet create \
--resource-group MyResourceGroup \
--vnet-name MyVNet \
--name PrivateSubnet \
--address-prefix 10.0.2.0/24
![image](https://github.com/user-attachments/assets/fdefb831-99e7-49bc-924c-f503d07354c7)
![image](https://github.com/user-attachments/assets/6b7c7c7b-9d26-4c71-bcb2-9a003fa3dc86)

Step 3: Set Up Security Groups and Network ACLs

Security Groups (NSGs) and Network ACLs (network security rules) control traffic.

1.	Create NSGs:

o Portal: Go to Network security groups > Create.

![image](https://github.com/user-attachments/assets/27ba5875-46e1-434e-919b-0e697f708735)

o Name each NSG, associate it with your VNet, and set rules.
o Create an NSG for the public subnet to allow specific inbound access (e.g., HTTP, HTTPS, SSH).

![image](https://github.com/user-attachments/assets/498e5806-842a-4504-ab29-3311aa9eac37)
![image](https://github.com/user-attachments/assets/e177de00-4c08-46b5-ae33-ab3d9b2e9319)

o Create an NSG for the private subnet, allowing only internal communication or specific services (e.g., database ports).
![image](https://github.com/user-attachments/assets/537d9a9a-3116-4ea3-9839-be5eaf924fe0)

o CLI example to create an NSG for the public subnet: 
az network nsg create 
--resource-group MyResourceGroup \
--name PublicNSG az network nsg rule create \
--resource-group MyResourceGroup \
--nsg-name PublicNSG \
--name AllowSSH \ --protocol Tcp \ --priority 1000 \
--source-address-prefixes '*' \
--source-port-ranges '*' \
--destination-address-prefixes '*' \
--destination-port-ranges 22 \
--access Allow
![image](https://github.com/user-attachments/assets/a5eab1fc-0b4f-452c-9996-d792b442b660)

2. Associate NSGs with Subnets:

o Portal: Go to Subnets within your VNet, select each subnet, and associate the appropriate NSG.
![image](https://github.com/user-attachments/assets/25ec39a8-6e30-40bf-a113-2ad71de11967)
o CLI: Copy code
az network vnet subnet update \
--vnet-name MyVNet \
--name PublicSubnet \
--resource-group MyResourceGroup \
--network-security-group PublicNSG
![image](https://github.com/user-attachments/assets/a7f3635b-ed55-4a45-be4c-dd248507d5b7)
![image](https://github.com/user-attachments/assets/ada719f6-6a51-41a9-98f4-a6230b8f4831)

Step 4: Deploy Virtual Machines (VMs) in the VNet 

1. Create a VM in the Public Subnet:

o Portal: Go to Virtual machines > Create.
o Choose the public subnet, assign a public IP for SSH/RDP access.
![image](https://github.com/user-attachments/assets/f8b61454-6fdf-48ae-b6f3-9fb6af74f7cb)
o CLI:
Copy code
az vm create \
--resource-group MyResourceGroup \
--name PublicVM \
--image UbuntuLTS \
--vnet-name MyVNet \ --subnet PublicSubnet \ --public-ip-address "" \ --admin-username azureuser \
--generate-ssh-keys

![image](https://github.com/user-attachments/assets/3fa8041b-c680-4e83-8250-7f43ed07b197)

2. Create a VM in the Private Subnet:

o Ensure this VM does not have a public IP (it will only be accessible through
internal mechanisms).
o Copy code	CLI example:
az vm create \
--resource-group MyResourceGroup \
--name PrivateVM \ --image UbuntuLTS \ --vnet-name MyVNet \ --subnet PrivateSubnet \
--admin-username azureuser \
--generate-ssh-keys

![image](https://github.com/user-attachments/assets/0678d3c7-c004-40a1-afed-352d02809063)

Step 5: Set Up NAT Gateway for Internet Access in Private Subnet 

1. Create a NAT Gateway:

o Portal: Go to NAT gateways > Create.
o Link the NAT gateway to the VNet and associate it with the private subnet.

![image](https://github.com/user-attachments/assets/1243a42e-cc51-432c-84e2-bbb948406bb8)

o CLI:
Copy code
az network nat gateway create \
--resource-group MyResourceGroup \
--name MyNATGateway \
--location eastus \
--public-ip-addresses MyNATPublicIP \
--idle-timeout 10
az network vnet subnet update \
--vnet-name MyVNet \
--name PrivateSubnet \
--nat-gateway MyNATGateway

![image](https://github.com/user-attachments/assets/3d3f1f50-ffb1-4343-86f7-a0071ddbea83)

Step 6: Configure a Bastion Host for Secure Access to Private VMs

The Bastion Host provides secure access to VMs without exposing them to the internet.
1.	Create the Bastion Host:

![image](https://github.com/user-attachments/assets/07906e3a-bde8-4038-b9e3-43fcfdd9316a)

o Portal: Go to Bastions > Create.
o Associate it with the public subnet of your VNet.
![image](https://github.com/user-attachments/assets/b0a07805-fd7f-482c-bce1-3328353c0229)

o CLI:
Copy code
az network bastion create \
--resource-group MyResourceGroup \
--name MyBastion \
--vnet-name MyVNet \
--subnet BastionSubnet \
--public-ip-address MyBastionPublicIP
![image](https://github.com/user-attachments/assets/31178f14-3e40-4166-8874-fd2c7b8a6f63)

2. Connect via Bastion:
o Use the Bastion Host to SSH/RDP into the private VM securely from the Azure portal or using Azure CLI commands.
![image](https://github.com/user-attachments/assets/753626fd-fa59-42ba-bc27-7d23bfbcbc06)

Step 7: Test Connectivity and Security

1.	Public VM: Test SSH/HTTP connections.

2. Private VM: Verify access through the Bastion Host and confirm the NAT gateway is
working (e.g., can access the internet but has no direct external access).

2.	Network ACL Rules: Adjust as needed to block or allow specific traffic.

## This setup should give you a secure, isolated environment with internet access only as configured.













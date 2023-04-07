---
weight: 1
url: test
title: Comparatif terraform / Azure cli

---

# Resource group 

**terraform**


_variables.tf_
```
variable "resource_group_location" {
  default     = "francecentral"
}

variable "resource_group_name_network" {
  default = "norsys_network"
}
```

_main.tf_
```
resource "azurerm_resource_group" "rg_network" {
  location = var.resource_group_location
  name     = var.resource_group_name_network
}
```


**az cli**
```
location=francecentral
az group create --name norsys_network --location $location
```

# Virtual Network

**terraform**
```
# Create a virtual network
resource "azurerm_virtual_network" "vnet" {
  name                = "norsysVnet"
  address_space       = ["10.0.0.0/20"]
  location            = var.resource_group_location
  resource_group_name = azurerm_resource_group.rg_network.name
}
#subnet    gateway VPN
resource "azurerm_subnet" "SubNetGateway" {
  name                 = "GatewaySubnet"
  resource_group_name  = azurerm_resource_group.rg_network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.15.0/27"]
}
# subnet 1 machine
resource "azurerm_subnet" "SubNet1" {
  name                 = "norsysSubNet1"
  resource_group_name  = azurerm_resource_group.rg_network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}
# subnet 2 kubernetes
resource "azurerm_subnet" "SubNet2" {
  name                 = "norsysSubNet2"
  resource_group_name  = azurerm_resource_group.rg_network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.2.0/24"]
}
```

**az cli** 
```
# virtual Network 

az network vnet create \
  --name $vnetName \
  --resource-group $rgNetwork \
  --dns-servers xxx.10 xxx.11
  --address-prefixes 10.59.0.0/20

# subnet

# gateway subnet pour vpn 

az network vnet subnet create \
   --resource-group $rgNetwork \
   --name GatewaySubnet \
   --vnet-name $vnetName \
   --address-prefixes 10.59.15.0/27

# subnet reseau 
 
az network vnet subnet create \
   --resource-group $rgNetwork \
   --name norsysSubnet1 \
   --vnet-name $vnetName \
   --address-prefixes 10.59.1.0/24

az network vnet subnet create \
   --resource-group $rgNetwork \
   --name norsysSubnet2 \
   --vnet-name $vnetName \
   --address-prefixes 10.59.2.0/24
```

**creation VM + SSD **
```
   az vm create --name vmlinux1 \
     --resource-group norsys_infra \
     --location francecentral \
     --public-ip-address "" \
     --size Standard_B1ls \
     --subnet /subscriptions/xx-df0b-4e99-97f9-xxxxxx/resourceGroups/norsys_network/providers/Microsoft.Network/virtualNetworks/norsysVnet/subnets/norsysSubnet1 \
     --storage-sku Standard_LRS \
     --admin-username azureid \
     --ssh-key-values ~/.ssh/infra.pub \
     --image Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest 
     #--private-ip-address 10.59.1.4 
  
az vm disk attach -g norsys_infra \
      --vm-name vmlinux1 \
      --size-gb 128 \
      --sku StandardSSD_LRS \
      --name disk_vmlinux1 --new

ipslave1=$(az vm list-ip-addresses --resource-group norsys_infra --name vmlinux1 --query [0].virtualMachine.network.privateIpAddresses[0] -o tsv)
 
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo apt update 

      # proc pour add disque aprés ssh machine  
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo mkfs.xfs /dev/sdc1
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo partprobe /dev/sdc1
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo mkdir /data
ssh -o "StrictHostKeyChecking no" azureid@$ipslave1 sudo mount /dev/sdc1 /data



``` 
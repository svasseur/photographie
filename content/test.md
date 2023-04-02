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
  --dns-servers 172.20.16.10 172.20.16.11
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


# SAP HANA ARM Installation
This ARM template is used to install SAP HANA on a single VM running SUSE SLES 12 SP 2 or Red Hat Enterprise Linux . It uses the Azure SKU for SAP. **We will be adding additional SKUs and Linux flavors in future Versions.** The template takes advantage of [Custom Script Extensions](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) for the installation and configuration of the machine.

[Deploy to Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fstagea66a6dd3ca954d68bdd.blob.core.windows.net%2Fazuresaphana22rgt-stageartifacts%2Fazuredeploy.json)


## Machine Info
The template current deploys HANA on a one of the machines listed in the table below with the noted disk configuration.  The deployment takes advantage of Managed Disks, for more information on Managed Disks or the sizes of the noted disks can be found on [this](https://docs.microsoft.com/en-us/azure/storage/storage-managed-disks-overview#pricing-and-billing) page.

Machine Size | RAM | Data and Log Disks | /hana/shared | /root | /usr/sap | hana/backup
------------ | --- | ------------------ | ------------ | ----- | -------- | -----------
E16 | 128 GB | 2 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S15
E32 | 128 GB | 2 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S20
E64 | 432 GB | 2 x P20 | 1 x S20 | 1 x P6 | 1 x S6 | 1 x S30
GS5 | 448 GB | 2 x P20 | 1 x S20 | 1 x P6 | 1 x S6 | 1 x S30
M64s | 1TB | 2 x P30 | 1 x S30 | 1 x P6 | 1 x S6 | 2 x S30
M64ms | 1.7TB | 3 x P30 | 1 x S30 | 1 x P6 | 1 x S6 | 2 x S40
M128S | 2TB | 3 x P30 | 1 x S30 | 1 x P6 | 1 x S6 | 2 x S40
M128ms | 3.8TB | 5 x P30 | 1 x S30 | 1 x P6 | 1 x S6 | 5 x S30

## Installation Media
Installation media for SAP HANA should be downloaded and place in the SapBits folder. This location will be automatically be uploaded to Azure Storage upon deployment.  Specifically you need to download SAP package 51052325, which should consist of four files:
```
51052325_part1.exe
51052325_part2.rar
51052325_part3.rar
51052325_part4.rar
```

Addtionally, if you wish to install a HANA Jumpbox with HANA Studio enabled, create a SAP_HANA_STUDIO folder under your SapBits folder and place the following packages:
```

IMC_STUDIO2_212_2-80000323.SAR
sapcar.exe
serverjre-9.0.1_windows-x64_bin.tar.gz

```

The Server Java Runtime Environment bits can be downloaded [here](http://www.oracle.com/technetwork/java/javase/downloads/server-jre9-downloads-3848530.html).


## Deploy the Solution
### Deploy from the Portal

To deploy from the portal using a graphic interface you can use the [Deploy to Azure](insertlink) link to bring up the template in your subscription and fill out the parameters.

### Deploy from Powershell

```powershell
New-AzureRmResourceGroup -Name HANADeploymentRG -Location "Central US"
New-AzureRmResourceGroupDeployment -Name HANADeployment -ResourceGroupName HANADeploymentRG `
  -TemplateUri https://raw.githubusercontent.com/claudhg9/saptest/master-subnet/azuredeploy.json `
  -VMName HANAtestVM -HANAJumpbox yes -CustomURI https://yourBlobName.blob.core.windows.net/yourContainerName -VMPassword AweS0me@PW
```

### Deploy from CLI
```azurecli
az login

az group create --name HANADeploymentRG --location "Central US"
az group deployment create \
    --name HANADeployment \
    --resource-group HANADeploymentRG \
    --template-uri "https://raw.githubusercontent.com/claudhg9/saptest/master-subnet/azuredeploy.json" \
    --parameters VMName=HANAtestVM HANAJumpbox=yes CustomURI=https://yourBlobName.blob.core.windows.net/yourContainerName VMPassword=AweS0me@PW
```


## Parameters

```
Parameter name | Required | Description | Default Value | Allowed Values
-------------- | -------- | ----------- | ------------- | --------------
VMName |Yes |Name of the HANA Virtual Machine. | None | No restrictions
HANAJumpbox |Yes |Defines whether to create a Windows Server with HANA Studio installed. | None | No Restrictions
VMSize |No |Defines the size of the Azure VM for the HANA server. | Standard_GS5 | Standard_GS5, Standard_M64s, Standard_M64ms, Standard_M128s, Standard_M128ms, Standard_E16s_v3, Standard_E32s_v3, Standard_E64s_v3 | No restrictions
NetworkName |No |Name of the Azure VNET to be provisioned | ra-hana-vnet | No restrictions
addressPrefixes |No |Address prefix for the Azure VNET to be provisioned | 10.0.0.0/16 | No restrictions
HANASubnetName |No | Name of the subnet where the HANA server will be provisioned | SAPDataSubnet | No restrictions
HANASubnetPrefix |No |Subnet prefix of the subnet where the HANA server will be provisioned | 10.0.5.0/24 | No restrictions
ManagementSubnetName |No | Name of the subnet where the HANA jumpbox will be provisioned | SAPMgmtSubnet | No restrictions
ManagementSubnetPrefix |No |Subnet prefix of the subnet where the HANA jumpbox will be provisioned | 10.0.6.0/24 | No restrictions
CustomURI | Yes | URI where the SAP bits are stored | None | No restrictions
VMUserName | No | Username for both the HANA server and the HANA jumpbox | testuser | No restrictions
VMPassword | Yes | Password for the user defined above | None | No restrictions
OperatingSystem | No | Linux distribution to use for the HANA server | SLES for SAP 12 SP2 | SLES for SAP 12 SP2, RHEL 7.2 for SAP HANA
HANASID | No | HANA System ID | H10 | No restrictions
HANANumber | No | SAP HANA Instance Number | 00 | No restrictions
ExistingNetworkResourceGroup | No | This gives you the option to deploy the VMs to an existing VNET in a different Resource Group. The value provided should match the name of the existing Resource Group. To deploy the VNET in the same Resource Group the value should be set to "no" | no | No restrictions
IPAllocationMethod | no | LEts you choose between Static and Dynamic IP Allocation | Dynamic | Dynamic, Static
StaticIP | No | Allows you to choose the specific IP to be assgined to the HANA server. If the allocation method is Dynamic this parameter will be ignored | 10.0.5.6 | No restrictions

```
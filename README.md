<<<<<<< HEAD
# Azure-Custom
# VM-Series for Microsoft Azure

This is a repository for Azure Resoure Manager (ARM) templates to deploy VM-Series Next-Generation firewall from Palo Alto Networks in to the Azure public cloud.

[VM-Series in Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/paloaltonetworks.vmseries-ngfw?tab=Overview):

- Bring Your Own License - [BYOL](https://azure.microsoft.com/en-us/marketplace/partners/paloaltonetworks/vmseries-ngfwbyol/)
- Pay-As-You-Go (PAYG) Hourly [Bundle 1](https://azure.microsoft.com/en-us/marketplace/partners/paloaltonetworks/vmseries-ngfwbundle1/) and [Bundle 2](https://azure.microsoft.com/en-us/marketplace/partners/paloaltonetworks/vmseries-ngfwbundle2/)

**Documentation**

- [Technical documentation](https://www.paloaltonetworks.com/documentation/80/virtualization/virtualization/set-up-the-vm-series-firewall-on-azure)
- [VM-Series Datasheet PDF](https://www.paloaltonetworks.com/content/dam/pan/en_US/assets/pdf/datasheets/vm-series/vm-series-for-microsoft-azure.pdf)
- [Deploying ARM Templates](https://azure.microsoft.com/en-us/documentation/articles/resource-group-template-deploy/#deploy-with-azure-cli)

**NOTE:**
- Deploying ARM templates requires some expertise and customization of the ARM JSON template. Please review the basic structure of ARM templates.
- Most of the templates in this repository typically use the BYOL version of VM-Series. If you want to use a different SKU then you can edit the azureDeploy.json template to set the `"imageSku"` variable to use `"byol"`, `"bundle1"` or `"bundle2"`:
```javascript
"imagePublisher": "paloaltonetworks",
"imageSku":       "byol",
"imageOffer" :    "vmseries1",
"imageVersion":   "latest"
```
- By default, if `"imageVersion"` is not specified then the latest PAN-OS version available in Azure Marketplace is used (equivalent to writing `"imageVersion": "latest"`). To use a specific PAN-OS version available in the Azure Marketplace, set it as `"imageVersion": "8.0.0"` or `"imageVersion": "7.1.0"`. For an example on setting the PAN-OS version see the following template: https://github.com/PaloAltoNetworks/azure/tree/master/vmseries-avset

- Before you use the custom ARM templates here, you must first deploy the related VM from the Azure Marketplace into the intended/destination Azure location. This enables programmatic access (i.e. template-based deployment) to deploy the VM from Azure Marketplace. You can then delete the Marketplace-based deployment if you don't need it.
- For example, if you plan to use a custom ARM template to deploy a BYOL VM of VM-Series into Australia-East, then first deploy the BYOL VM from Marketplace into Australia. This is needed only the first time. You can then delete this VM and its related resources. Now your ARM templates, from GitHub or via CLI, will work.
- The older Marketplace listing [VM-Series (BYOL) Solution Template](https://azure.microsoft.com/en-us/marketplace/partners/paloaltonetworks/vmseriesbyol-template2template2-3nic-3subnetbyol/) is deprecated; please do not use this template. Use the above listings in the Marketplace.
=======
# VM-Series with NAT VM, and VM's for Web and DB subnet

This ARM template deploys a VM-Series next generation firewall VM in an Azure resource group alongwith the following resources similar to a typical two tier architecture. It also adds the relevant User-Defined Route (UDR) tables to send all traffic through the VM-Series firewall.

[<img src="http://azuredeploy.net/deploybutton.png"/>](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdmaynard1%2FAzure-Custom%2Fmaster%2FazureDeploy.json)

Azure-Custom/azureDeploy.json

[<img src="https://camo.githubusercontent.com/536ab4f9bc823c2e0ce72fb610aafda57d8c6c12/687474703a2f2f61726d76697a2e696f2f76697375616c697a65627574746f6e2e706e67" data-canonical-src="http://armviz.io/visualizebutton.png" style="max-width:100%;">](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fdmaynard1%2FAzure-Custom%2Fmaster%2FazureDeploy.json)

**Virtual Machines:**

- VM-Series Next generation firewall - (D3 VM size) - Bring Your Own License (BYOL)
- Web VM - an Ubuntu VM (A1) that can be setup as a web server
- DB VM - an Ubuntu VM (A1) that can be setup with a database

<b>NOTE</b>: The NAT VM is no longer required as Azure supports [assigning a public IP](https://azure.microsoft.com/en-us/updates/ga-multiple-ips-per-nic) to any NIC of VM-Series. After deployment you can delete it and assign a public IP to the eth1(Untrust) interface of VM-Series.

**Virtual Network (VNET):**

- VNET : 192.168.0.0/16
- Mgmt Subnet: 192.168.0.0/24 - For the firewall's management interface (eth0)
- Untrust Subnet: 192.168.1.0/24 - For eth1 of firewall
- Trust Subnet: 192.168.2.0/24 - For eth2 of firewall
- Web Subnet: 192.168.3.0/24
- DB Subnet: 192.168.4.0/24

**Note:** The template also configures the Azure User-Defined Routing (UDR) table to force all packets through the VM-Series firewall. This provides visibility to all traffic from the VM-Series dashboard and allows you to enforce advanced security policies to protect your Azure deployment.

You can also download the templates and customize them as needed. To deploy them from Azure CLI use the following commands:
```
azure login
azure config mode arm
azure group create -v -n <i>ResourceGroupName</i>  -l <i>AzureLocationName</i>  -d  <i>DeploymentLabel</i>  \
    -f azureDeploy.json  -e azureDeploy.parameters.json
For example:
azure group create  -n myResGp1  -l westus  -d myResGp1Dep1  \
    -f azureDeploy.json  -e azureDeploy.parameters.json
```

See documentation on how to configure the VM-Series firewall after deployment. Here is a basic outline:
- If you want to use hourly Pay-As-You-Go (PAYG) options then change the template's sku variable to bundle1 or bundle2 instead of byol.
- Connect to the firewall using the public IP or DNS assigned to **eth0** of the firewall
- Enable **eth1** and **eth2** as DHCP interfaces with a default virtual router
- Create static routes for each of the firewall's dataplane interfaces to point to the .1 of its subnet
- Create a NAT policy inside the firewall to decide where the traffic should go after deployment, for example you can send it, via DNAT, to the webserver (192.168.3.5) in case of Internet facing deployments.
- Create security policies: Inter-zone from Untrust/Trust and Intra-zone (web subnet to/from DB subnet)
- Once these are configured:
    - connect over SSH to the NAT VM's public IP/DNS which sends it to the firewall, which using the firewall's DNAT is sent to the webserver
    - Install an Apache webserver on this VM using  `sudo apt-get install apache2`
>>>>>>> b8d9595e72eec8349bbfd02fe2b505fd036ee0a2

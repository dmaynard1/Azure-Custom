# VM-Series with 4 interfaces (mgmt,untrust,trust,dmz)

This ARM template deploys a VM-Series next generation firewall VM in an Azure resource group alongwith the following resources similar to a typical two tier architecture.

[<img src="http://azuredeploy.net/deploybutton.png"/>](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdmaynard1%2FAzure-Custom%2Fmaster%2Fvmseries-ARM4interface%2Ftemplate.json)

Azure-Custom/azureDeploy.json

**Virtual Machines:**

- VM-Series Next generation firewall - (D3 VM size) - Bring Your Own License (BYOL)

**Virtual Network (VNET):**

- VNET : 192.168.0.0/16
- Mgmt Subnet: 192.168.0.0/24 - For the firewall's management interface (eth0)
- Untrust Subnet: 192.168.1.0/24 - For eth1 of firewall
- Trust Subnet: 192.168.2.0/24 - For eth2 of firewall
- DMZ Subnet: 192.168.3.0/24 - For eth3 of firewall

See documentation on how to configure the VM-Series firewall after deployment. Here is a basic outline:
- If you want to use hourly Pay-As-You-Go (PAYG) options then change the template's sku variable to bundle1 or bundle2 instead of byol.
- Connect to the firewall using the public IP or DNS assigned to **eth0** of the firewall

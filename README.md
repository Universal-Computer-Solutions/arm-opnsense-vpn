# OPNsense VPN Appliance — Azure Deployment

ARM template for deploying an OPNsense VPN appliance VM in Azure from a generalised image stored in an Azure Compute Gallery.

## What This Deploys

- OPNsense VM from a specified Compute Gallery image
- WAN NIC with Standard Static public IP and NSG (IKE/NAT-T/ESP/SSH/HTTPS inbound)
- LAN NIC with IP forwarding enabled
- Root password set via post-deployment custom script extension
- Optional: Route table for remote site subnets, associated with selected VNet subnets

## Prerequisites

- An OPNsense generalised image version in an Azure Compute Gallery
- The MSP tenant app registration consented in the target tenant (for cross-tenant image access)
- An existing VNet with at minimum a WAN subnet and LAN subnet
- Contributor access to the target resource group

## Deploy

Click the button below to launch the deployment wizard in the Azure portal:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FUniversal-Computer-Solutions%2Farm-opnsense-vpn%2Fmain%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2FUniversal-Computer-Solutions%2Farm-opnsense-vpn%2Fmain%2FcreateUiDefinition.json)

> **Note:** Replace both `REPLACE_WITH_RAW_URL_TO_*` placeholders in the README with the actual raw GitHub URLs to the two files after pushing to your repo. Raw URLs follow the format:
> `https://raw.githubusercontent.com/<org>/<repo>/main/<filename>`

## Parameters

| Parameter | Description |
|---|---|
| `vmName` | Name of the VM. All related resources are named from this. |
| `imageResourceId` | Full resource ID of the OPNsense image version in the MSP Compute Gallery. |
| `vmSize` | Azure VM size. Defaults to `Standard_B2als_v2`. |
| `rootPassword` | Password to set for the root account on the OPNsense VM. |
| `vnetResourceGroup` | Resource group containing the target VNet. |
| `vnetName` | Name of the target VNet. |
| `wanSubnetName` | Name of the WAN subnet. |
| `lanSubnetName` | Name of the LAN subnet. |
| `osDiskSku` | OS disk storage SKU. Defaults to `StandardSSD_LRS`. |
| `remoteSubnets` | Array of on-premises/remote site CIDRs to route via OPNsense LAN NIC. Optional. |
| `routeTableSubnetNames` | Array of VNet subnet names to associate the route table with. Optional. |

## Post-Deployment Steps

1. **Restrict NSG rules** — update the SSH and HTTPS inbound rules on `nsg-nic-<vmName>-wan` to allow only your MSP management IP rather than `*`
2. **Set static private IPs** — set both NIC private IPs to static in Azure to prevent reassignment on VM restart
3. **Configure OPNsense** — log in via SSH or the web UI and configure:
   - WAN/LAN interface assignments
   - VPN tunnel(s)
   - Firewall rules
   - `/usr/local/etc/monit-webhook.env` with customer HaloPSA URL and client ID
4. **Verify routing** — confirm route table is associated with the correct subnets and test connectivity from a VM on each associated subnet

## Updating the Image

When a new OPNsense image version is published to the Compute Gallery, update the `imageResourceId` parameter at next deployment. Existing VMs are not affected.

## Repository Structure

```
├── azuredeploy.json          # ARM template
├── createUiDefinition.json   # Portal wizard UI definition
└── README.md                 # This file
```

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

1. **Set static private IPs** — set both NIC private IPs to static in Azure to prevent reassignment on VM restart
2. **Associate route table** — if remote site subnets were specified, go to each Azure subnet that needs to route traffic via OPNsense, open the subnet, and associate the created route table (`rt-<vmName>-remotesites`). Do this manually rather than via ARM to avoid removing existing subnet properties such as NAT gateways
3. **Run the configure script** — SSH in or use Azure Serial Console and run:
   ```bash
   sh /usr/local/bin/opnsense-configure.sh
   ```
   This configures the LAN gateway, static routes, firewall aliases, IPsec VPN connections and PSKs
4. **Note the PSKs** — the script outputs a PSK for each VPN connection at the end; provide these to the administrators of each remote site
5. **Reboot** — reboot the VM to ensure all settings are cleanly applied from saved config:
   ```bash
   reboot
   ```
6. **Configure remote site VPN gateways** — use the PSKs from step 4 to configure the VPN on each remote site device
7. **Update monitoring** — edit `/usr/local/etc/monit-webhook.env` with the customer HaloPSA URL and client ID
8. **Test connectivity** — verify VPN tunnels come up and test traffic both directions from Azure and from each remote site

## Updating the Image

When a new OPNsense image version is published to the Compute Gallery, update the `imageResourceId` parameter at next deployment. Existing VMs are not affected.

## Repository Structure

```
├── azuredeploy.json          # ARM template
├── createUiDefinition.json   # Portal wizard UI definition
└── README.md                 # This file
```
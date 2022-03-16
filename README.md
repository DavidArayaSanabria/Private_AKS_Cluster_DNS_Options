# Azure Kubernetes Service or AKS

Azure Kubernetes Service (AKS) simplifies deploying a managed Kubernetes cluster in Azure by offloading the operational overhead to Azure. As a hosted Kubernetes service, Azure handles critical tasks, like health monitoring and maintenance. Since Kubernetes masters are managed by Azure, you only manage and maintain the agent nodes. Thus, AKS is free; you only pay for the agent nodes within your clusters, not for the masters.

# Private AKS Clusters

In a private cluster, the control plane or API server has internal IP addresses that are defined in the RFC1918 - Address Allocation for Private Internet document. By using a private cluster, you can ensure network traffic between your API server and your node pools remains on the private network only.


## Target audience

- Infrastructure Architect
- Security Team
- Kubernetes Administrator
- Cloud Solution Architect

# Create a Private AKS Cluster

The [Template.bicep](https://github.com/DavidArayaSanabria/Private_AKS_Cluster_DNS_Options/blob/b717471cb688931b694ac42e3f9754154b1d3d46/Deployment%20Templates/Template.bicep) Azure Bicep template will help you automatically deploy an AKS cluster

You need to modify: enablePrivateCluster: bool

# Private AKS DNS options:

## System assigned Private DNS Zone:


A private DNS zone is created automatically for you in the node resource group.
 
The name of the API server gets an A record in that DNS zone and the DNS zone is associated with the VNET that contains the cluster and off course the resources in the same VNET will know how to resolve the API server.

The cluster VNET contacts the API server using a private link deployed automatically

So if you want to manage the cluster you need to have a client with kubectl in the cluster's VNET

If your client is not on the VNET you can use the az aks invoke command. (AAD integration does not work)

![alt image](https://github.com/DavidArayaSanabria/Private_AKS_Cluster_DNS_Options/blob/de51056f93b8754a109eac9a93342a5a5ab508d2/Images/System%20Option.png)

## No Private DNS Zone:

The cluster is deployed with a public DNS instead of a Private DNS Zone.

You still need the translation to the private IP of the API server but instead of using a Private DNS zone you use a Public FQDN.
Microsoft creates a name that can be resolved by public DNS servers to the private IP of the API, this is transparent for the customers.

This is the simplest option if you do not want to managed private DNS.

Modify your .bicep as below to deploy a private AKS witha public FQDN

```json
apiServerAccessprofile: {
  enbalePrivateCluster: true
  privateDNSZone: 'none'
  enablePrivateClusterPublicFQDN: true
}
```

## Bring Your Own DNS:

With the custom DNS option, you cannot use any name you like. The Private DNS Zone has to be like: privatelink.<region>.azmk8s.io. For instance, if you deploy your AKS cluster in West Europe, the Private DNS Zone’s name should be privatelink.westeurope.azmk8s.io. There is an option to use a subdomain as well.

When you use the custom DNS option, you also need to use a user-assigned Managed Identity for the AKS control plane. To make the registration of the A record in the Private DNS Zone work, in addition to linking the Private DNS Zone to the virtual network, the managed identity needs the following roles (at least):
- Private DNS Zone Contributor role on the Private DNS Zone
- Network Contributor role on the virtual network used by AKS


![alt image](https://github.com/DavidArayaSanabria/Private_AKS_Cluster_DNS_Options/blob/b717471cb688931b694ac42e3f9754154b1d3d46/Images/BYO%20DNS.png)



## Azure services and related products

- Private Endpoint
- Private Link
- Kubernetes
- Private DNS zone

## Related references
- https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes
- https://docs.microsoft.com/en-us/azure/aks/private-clusters
- https://blog.baeke.info/2021/07/01/dns-options-for-private-azure-kubernetes-service/
- https://docs.microsoft.com/en-us/azure/aks/coredns-custom
- https://docs.microsoft.com/en-us/azure/templates/microsoft.containerservice/managedclusters?tabs=bicep
- https://docs.microsoft.com/en-us/azure/firewall/protect-azure-kubernetes-service
- https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-managed-identities-work-vm#user-assigned-managed-identity


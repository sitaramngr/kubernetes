frontend - ASP.NET Core Blazor (web UI)  - Hosted on VM
backend - ASP.NET Core Web API (rest API backend)  - Hosted on VM
Database - postgresql (Persistance storage) - Azure DB for postgresql

have the application code and Packer templates for both the frontend and backend. 
Then, we have GitHub Actions to implement our CI/CD process and Terraform to provision our 
Azure infrastructure and reference the Packer-built VM images for our Azure VMs.

VNET,Subject - Scoped to region
Azure has two resiliency modes: one based on fault domains or regional 
and another based on Availability Zones or zonal. VMs can be provisioned in either of these two modes.

Load Balancer - 
FrontEnd LB- Frontend IP(Public), backend address pool(frontend subnet), 

Azure Load Balancer organizes how it routes incoming traffic using rules. 
Each rule has a protocol, a frontend component, and a backend component.

Frontend LB -> 5000(http) -> ingress(frontend Nodepool) -> Egress(frontend Nodepool)-> 80(http) ->ingress(backupend LB)
backend LB -> 5000(http) -> ingress(backend Nodepool) -> Egress(backend Nodepool) > 5432(http) > ingress(database)

Secrets such as database credentials or service access keys need to be stored securelyin azure key vault.
Secrets can be accessed - VM are grated RBAC , assign user manged identity to VM , assign role assignment to managed identity

Pcker - 
Azure plugin - a plugin for Packer that encapsulates the integration with its services:
azure-arm builder - that will generate Azure VM images by creating a new VM from a base image, executing the provisioners, 
taking a snapshot of the Azure managed disk, and creating an Azure managed image from it. 

To configure the operating system, we must install software dependencies (such as .NET 6.0), copy and deploy our application code’s 
deployment package to the correct location in the local filesystem, 
configure a Linux service that runs on boot, and set up a local user 
and group with necessary access for the service to run as.


Terraform

azurerm_resource_group
azurerm_virtual_network
azurerm_subnet
azurerm_public_ip
azurerm_lb
azurerm_lb_backend_address_pool
azurerm_network_interface_backend_address_pool_association
azurerm_lb_probe
azurerm_lb_rule
azurerm_application_security_group
azurerm_network_security_group
azurerm_key_vault
azurerm_user_assigned_identity
azurerm_role_assignment  (role_definition_name = "Key Vault Secrets User")

data "azurerm_image"
azurerm_network_interface
azurerm_linux_virtual_machine



Kubernetes
azurerm_container_registry
azurerm_role_assignment
azurerm_kubernetes_cluster
azurerm_role_assignment
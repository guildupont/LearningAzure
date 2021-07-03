# LearningAzure
A set of scripts and templates to learn Azure and help preparing for Azure certifications. 

This repository contains two ARM templates to deploy VMs or VM scale sets running a webserver returning instance details. It also contains a Dockerfile to return similar metadata for container-based solutions such as Azure Kubernetes Services and Azure Container Instances.

## Objective
Setting up a basic webserver on a virtual machine will sometimes make it a lot easier to experiment Azure Solutions. Think of all load balancing and networking solutions for example. This repository contains two Azure Resource Manager templates which use the Linux Custom Script extension to set up a webserver when the VM starts. You can deploy the ARM template in any region/location to set up the webserver and carry on other experiments once the templates have been deployed. A Dockerfile is also available for similar experiments on AKS and ACI.

## Usage
Start by creating a resource group in a given location. Note, by default resources will be created in the location of the parent resource group.
```bash
az group create --name gdtest-vm-rg --location eastus
```

Update the downloaded templates, by replacing default passwords and name prefixes for example.

Option 1: Create a VM with a webserver
```bash
az deployment group create --resource-group gdtest-vm-rg --template-file vm/CreateWebserverVM.json
```
To display the public IP created for the VM, use 
```bash
az vm list --output jsonc --show-details --query "[*].publicIps"
```

Option 2: Create a VM scale set with a webserver
```bash
az deployment group create --resource-group gdtest-vm-rg --template-file vmss/CreateWebserverVMSS.json
```
Identify the public IP address associated to the load balancer by using a similar az CLI command.

Option 3: Create a Docker container and push it into a newly created Azure Container Registry
```bash
docker build -t webserver-container:v1 docker/
az acr create --name gdtestcontainerreg --resource-group gdtest-vm-rg --sku basic
az acr login --name gdtestcontainerreg 
docker tag webserver-container:latest gdtestcontainerreg.azurecr.io/webserver-container:latest
docker push gdtestcontainerreg.azurecr.io/create-webserver-container:latest
```
Once the container is in the registry, you can deploy it in a kubernetes cluster (using kubectl create deployment and kubectl expose deployment) or in a container instance.



## Technical considerations
### VM and VM scalesets templates
This package contains two ARM templates,
  - One for single VMs
  - One for VM scalesets
And one Dockerfile,
  - To be used for Azure solutions such as App Service, Container Instance, Kubernetes, etc

The VM ARM template starts by deploying NSGs, a virtual network, a public IP, and a network interface, before deploying a small B1s virtual machine and the Custom Script extension. 

The VM scale set template is slightly more complex since it uses an Azure Load Balancer (load balancing level 4) for inbound load balancing but also for outbound traffic NATing. ARM templates also vary slightly between virtual machines and VM scale sets: For example, network interfaces, extensions, etc, are not declared the same way. Note that you can delete the Load Balancer or any other component once the template has been deployed to replace it by an Azure Application Gateway, Front Door, or any other solution you may want to test.

### Webserver
The ARM extension starts by using the APT packet manager to install the Apache 2 webserver and its dependencies. Once this is done, the next lines of the script override the index.php index page of the webserver with the following content
- A first line containing the VM or VM scale set name, location, timestamp, and OS host name. The OS hostname is retrieved using the uname command. Retrieving the OS hostname is one of the ways to know exactly on which virtual machine of a scaleset the workload is being executed.
- A call to the Azure Metadata API (169.254.169.254) to retrieve and display all VM instance details.

The Dockerfile uses the same principles but starts from a Apache/PHP base container image before overriding the index.php page.

### Production use
It is not recommended to use these templates in production. Some of the Azure ARM best practices [https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/best-practices] are voluntarilly ignored for simplicity's sake. The PHP pages with exec commands are also not the most elegant solution around, however they allow to reach the business objectives in only a couple of easy-to-understand lines of code.







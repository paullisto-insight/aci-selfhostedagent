# Azure Container Instance - Azure Self-Hosted DevOps Agent

**This is the initial Prototype of the ACI Azure DevOps Self-Hosted Agent. Further testing is required  before it is approved for Production Project Deployments**

This repo will provide a dockerfile to build a container image, and push it a already provisioned Azure Container Registry.

Please use best practices with service principals no Admin Access enabled on the ACR.

The container image is meant to be deployed into a Private Network. 

The ACI SelfHosted DevOps Agent should be deployed into a Private Azure  Vnet  Delegated  Subnet.
It should not accessible on a public endpoint for security reasons.

## Azure Container Instance Packages

The following Capabilities are available on the Container Image

- Powershell Core 7
- .Net ASPNet Core  6.0 and DotNet  SDK 6.0
- AzCopy
- AzureCli
- Azure Powershell Modules 
- Azure SQL Powershell modules
- HashiCorp Packer
- HashiCorp Terraform
- KubeCtl
- Helm
- Ostio
- Bicep CLI
- GitHub CLI
- AzureDevops Agent

(Github HostedRunner Agent Feature to be added at future date.)

**All Packages are installed from Trusted Source locations. For new Capabilities to be added to the Agent, they must come from a verifiable source.**

Please Note: Out  of  the box functionality will install range of Powershell Modules from Azure, SQL, PowerBi etc.

Azure Powershell Modules have been dilberately installed using the same method as the Microsoft Maintained Cloud Shell container Image to get the best experience and extra functionality.  Remove  the comments in the dockerfile to install the PSCloudShellUtility Module to unlock extra commands including Enter-AzVM, Invoke-AzVMCommand|

For More information on the  AzureCloudShell Project Please refer to <a  href="https://github.com/Azure/CloudShell">Azure Cloud Shell on Github</a>

# To Get Start Started:

1.	Create your Azure Container Registry
2.	Fill in the Missing details on the DevopsPipleine  .\pipelines\BuildDevOpsAgent.yaml
3.	The pipeline will push container image to the ACR once it has finished building.
4.  Use the ARM Templates as a reference to deploy a container instance using the latest image from ACR into an Azure Virtual Network.
5.  Ensure to specify the environment variables necessary to run and registar the Devops Agent.


# The Finished Result:

The Azure Container Instance deployed and configured as an Azure DevOps Agent.
![Azure Container Instance Provisioned](/images/ContainerImageBuilt.png)

The Azure Devops Agent Registered.
![Azure Devops Agent Registered](/images/ACIRegisteredInAgentPools.PNG)




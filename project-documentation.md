<h1> Disclaimer </h1>

Many thanks to the guidance offered in the laboratory: https://www.azuredevopslabs.com/labs/vstsextend/kubernetes/#overview.

The documentation is customized with my implementation and how I solved the issues that have appeared along the way.

<h1> Overview </h1>

Azure Kubernetes Service (AKS) is the fastest way to deploy Kubernetes on Azure. AKS handles the complexities of your hosted Kubernetes environment, allowing you to focus on building and managing containerized applications. It automates operations like provisioning, scaling, and upgrading resources, ensuring minimal downtime. Azure DevOps streamlines Docker image creation for efficient and reliable deployments.

Unlike traditional cloud resource management, AKS empowers you to create and manage resources within the Azure Kubernetes cluster using deployment and service manifest files. This provides greater flexibility and control over your application infrastructure.


<h1> Main topics </h1>

**- Docker:** Docker is a platform that isolates applications into containers, providing a lightweight, portable way to run software on any Linux system.<br />
**- Images:** An image is a blueprint containing the instructions needed to create a container, serving as a read-only template.<br />
**- Containers:** Containers offer isolated environments for running applications, ensuring they function independently and consistently.<br />
**- Kubernetes:** Kubernetes is an open-source system that automates the management of containerized applications across multiple hosts, simplifying deployment, maintenance, and scaling.<br />
**- Pods:** Pods are the fundamental units of Kubernetes, typically containing a single container. They represent the smallest executable unit of work.<br />
**- Services:** Services define how other pods can access the services provided by your application.<br />  
**- Deployments:** Deployment controllers manage updates to pods based on declarative specifications.<br />  
**- Kubernetes Manifest Files:** Kubernetes manifests describe the desired state of your application, defining deployments, services, and pods in JSON or YAML format. The file extensions .yaml, .yml, and .json are commonly used.<br />

<h1> Laboratory setup </h1>

This lab showcases the deployment of a Dockerized ASP.NET Core web application, MyHealthClinic (MHC), to an Azure Kubernetes Service (AKS) cluster using Azure DevOps.

The deployment utilizes a manifest file named mhc-aks.yaml. This file defines the desired configuration for your application, including:<br />
**- Deployments:** These specify how containerized applications like MHC should be launched and managed.<br />
**- Services:** These define how network traffic is routed to your application components, including the load balancer for external access and the Redis cache in the backend.<br />
**- Load Balancer:** This acts as the entry point for external traffic, directing requests to healthy MHC instances within the cluster.<br />

Within the cluster, the MHC application runs inside a pod named mhc-front, alongside the load balancer component.

![image](https://github.com/user-attachments/assets/345f9085-8d0b-4055-abed-843b4df2a6b1)


<h2> Basic steps </h2>

1. Create an Azure Container Registry (ACR), AKS and Azure SQL server
2. Provision the Azure DevOps Team Project with a .NET Core application using the Azure DevOps Demo Generator tool.
4. Configure application and database deployment, using Continuous Deployment (CD) in the Azure DevOps.
5. Initiate the build to automatically deploy the application.

<h2> Prerequisites </h2>

Before to start, you will need the followings:

**1. Microsoft Azure Account:** You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a free trial.<br />
**2. You will need an Azure DevOps account:** Azure DevOps Demo Generator helps you create team projects on your Azure DevOps Organization with sample content that include source code, work items, iterations, service endpoints, build and release definitions based on the template you choose during the configuration.
Example: Use Azure DevOps Demo Generator | Microsoft Learn

<h2> My setup </h2>

![image](https://github.com/user-attachments/assets/c67146fb-cb70-44a1-b075-d4b5f72772ee)


The confirmation of completion

![image](https://github.com/user-attachments/assets/d8caf9cc-befb-4921-8dc6-8db4365e3821)

![image](https://github.com/user-attachments/assets/e87e10c3-908c-410a-abb3-116be08052e0)


<h1> Environment setup </h1>

The following azure resources need to be configured for this lab:

![image](https://github.com/user-attachments/assets/74f215d1-be57-4309-9fc1-b1855339d6c3)


<h2> Steps </h2>

1. Launch the Azure Cloud Shell from the Azure portal and choose Bash.
2. Deploy Kubernetes to Azure, using CLI:

2.1) Get the latest available Kubernetes version in your preferred region into a bash variable. <br />

version=$(az aks get-versions -l eastus --query 'orchestrators[-1].orchestratorVersion' -o tsv)

2.2) Create a Resource Group

az group create --name akshandsonlab --location eastus

![image](https://github.com/user-attachments/assets/67bcaee2-50d0-451e-9154-22d58a6b8960)

2.3) Create AKS using the latest version available.

az aks create --resource-group akshandsonlab --name aks-deploy-training --enable-addons monitoring --kubernetes-version 1.28.0 --generate-ssh-keys --location eastus

![image](https://github.com/user-attachments/assets/e690b3de-26dd-4332-9737-bee426f2af0b)

3. Deploy Azure Container Registry (ACR): Run the below command to create your own private container registry using Azure Container Registry (ACR).

az acr create --resource-group akshandsonlab --name acrdeploytraining --sku Standard --location eastus

![image](https://github.com/user-attachments/assets/127f5efb-5483-4b05-8ef8-e1af6c5aa47f)

**Note:** When I am using Azure Container Registry (ACR) with Azure Kubernetes Service (AKS), and an authentication mechanism needs to be established. 

I set up the AKS to ACR integration using a simple command with the Azure CLI. 

This integration assigns the AcrPull role to the managed identity associated with the AKS Cluster.

az aks update -n aks-deploy-training -g akshandsonlab --attach-acr acrdeploytraining

![image](https://github.com/user-attachments/assets/262605e0-76f3-413d-bda2-6d8bc9a52062)

4. Create an Azure SQL server.

az sql server create -l eastus -g akshandsonlab -n sql-deploy-training -u sqladmin -p P2ssw0rd1234

![image](https://github.com/user-attachments/assets/f9835e4a-65e0-4eba-879d-24af851f5a57)

5. Create a database.

az sql server create -l eastus -g akshandsonlab -n sql-deploy-training -u sqladmin -p P2ssw0rd1234

![image](https://github.com/user-attachments/assets/2c7e914f-cf7b-4ff2-b435-720b50f48999)

**Note:** I resolved the below error using another region, because my subscription doesn’t support creating SQL server in the East US location.

![image](https://github.com/user-attachments/assets/3557bb20-d886-4beb-abe8-6a1b2401dc4e)

az sql server create -l westus -g akshandsonlab -n sql-deploy-training -u sqladmin -p P2ssw0rd1234

All the resources that were created in the akshandsonlab resource group.

![image](https://github.com/user-attachments/assets/3837fd0f-1dfc-4e2a-b2cf-161bd654c57a)

Click on “Set server Firewall” and enable “Allow Azure services …” option.

![image](https://github.com/user-attachments/assets/d9ba3a5f-768e-423f-a3df-9e346d4196e8)

**Important information:**

**Server name:** sql-aks-training.database.windows.net
**Login server name:** acrdeploytraining.azurecr.io


<h1> Build pipeline configuration </h1>

I manually mapped Azure resources such as AKS and Azure Container Registry to the build and release definitions.

<h2> Steps </h2>

1. Navigate to Pipelines, select and edit MyHealth.AKS.Build pipeline and click Edit.

![image](https://github.com/user-attachments/assets/0d3e95d4-1856-4ba6-adbe-dc13ee15c6d9)

2. Authorize the Azure subscription In Run services task.

![image](https://github.com/user-attachments/assets/7977ac16-4eb0-402f-937d-57581a9a63d7)

**Note:**

- This creates an Azure Resource Manager Service Endpoint, which defines and secures a connection to a Microsoft Azure subscription, using Service Principal Authentication (SPA). This endpoint will be used to connect Azure DevOps and Azure. <br />

- If the subscription is not listed or to specify an existing service principal, follow the Service Principal creation instructions. 

3. After completing the successful authentication, select appropriate values from the dropdown - Azure subscription and Azure Container Registry for the rest of the services (build, push, and lock).

![image](https://github.com/user-attachments/assets/3f8a5f37-6aaf-4667-9360-bff0e0312eb7)

![image](https://github.com/user-attachments/assets/b6f72824-68e2-47cb-a0ce-45a8d66b18d8)

Note: Explanations of the above services and important files used in the deployment:

![image](https://github.com/user-attachments/assets/82fdf9d8-4a20-4d97-ad80-d2859500009a)

The manifest file will look like as in the following screenshot:

![image](https://github.com/user-attachments/assets/c933edb5-e3d5-4934-8d04-046978d29cd9)

4. Click on the Variables tab and update ACR and SQLserver values for Pipeline Variables with the details saved earlier while configuring the environment. Then save the changes.

![image](https://github.com/user-attachments/assets/491224a5-14f0-400d-8dac-5d0301dc2a8a)


<h1> Build pipeline yaml file configuration </h1>

I also have a YAML build pipeline. It is the same configuration of the above pipeline, but using a YAML file.

<h2> Steps </h2>

1. Navigate to Pipelines –> Pipelines, select MyHealth.AKS.Build - YAML pipeline and click Edit.

![image](https://github.com/user-attachments/assets/0d131048-9748-4821-9e22-dcb3601d9910)

2. Authorize in Run Services task the Azure subscription.

![image](https://github.com/user-attachments/assets/a894aec8-d656-4362-9a4c-e722aeebaf65)

**Note:** This creates an Azure Resource Manager Service Endpoint, which defines and secures a connection to a Microsoft Azure subscription, using Service Principal Authentication (SPA). This endpoint will be used to connect Azure DevOps and Azure.

3. Following the successful authentication, select appropriate values from the dropdown for Azure subscription and Azure Container Registry. Repeat this for the Build services, Push services and Lock services tasks in the pipeline.

![image](https://github.com/user-attachments/assets/764e345a-be08-444a-a7df-5853e0545fa4)


4. Click on the Variables tab, and Update ACR and SQLserver values for Pipeline Variables with the details noted earlier while configuring the environment.

![image](https://github.com/user-attachments/assets/d52cc0d5-3d91-400b-bd5c-8a520b8451f8)

![image](https://github.com/user-attachments/assets/12cc54b1-f0e5-4e83-8b74-e518f4b40396)

<h1> Release pipeline configuration </h1>

1. Navigate to Pipelines ->  Releases -> Select MyHealth.AKS.Release pipeline and click Edit.

![image](https://github.com/user-attachments/assets/38af3408-fe16-4131-9136-9eba8f080dce)

2. Select Dev stage and click View stage tasks to view the pipeline tasks.

![image](https://github.com/user-attachments/assets/be90c268-0fc6-4c77-9911-660ec4bca9d1)

3. In the Dev environment, under the DB deployment phase, select Azure Resource Manager from the drop down for Azure Service Connection Type, andupdate the Azure Subscription value from the dropdown.

![image](https://github.com/user-attachments/assets/9510f3da-34a2-41e7-83c2-34064b5eaeff)

4. In the AKS deployment phase, select Create Deployments & Services in AKS task.

![image](https://github.com/user-attachments/assets/f179603a-0e7c-4bd2-b8f6-8946fd9720db)

Update the Azure Subscription, Resource Group and Kubernetes cluster from the dropdown. Expand the Secrets section and update the parameters for Azure subscription and Azure container registry from the dropdown. Repeat the same steps for “Update image in AKS” step.

![image](https://github.com/user-attachments/assets/ae80f702-3134-4e38-8448-5ac834223181)

**Note:** Explanations of the steps used in the pipeline.

![image](https://github.com/user-attachments/assets/584f73a7-ac86-4f4d-9ab5-90972f0f0607)

-  A secret called mysecretkey is created in AKS cluster through Azure DevOps by using command kubectl create secret in the background. This secret will be used for authorization while pulling myhealth.web image from the Azure Container Registry. <br />

5. Select the Variables section under the release definition, update ACR and SQLserver values for Pipeline Variables with the details noted earlier while configuring the environment. Select the Save button.

Note: The Database Name is set to mhcdb and the Server Admin Login is sqladmin and Password is P2ssw0rd1234. If you have entered different details while creating Azure SQL server, update the values accordingly.

![image](https://github.com/user-attachments/assets/d852f5f1-a97b-43b2-a68c-ccc63233d4ee)

<h1> Trigger a build and deploy the application </h1>

In this step, we will  trigger a build manually and upon completion, an automatic deployment of the application will be triggered. 

The application is designed to be deployed in the pod with the load balancer in the front-end and Redis cache in the back-end.

<h2> Steps </h2>

1. Select MyHealth.AKS.build pipeline. Click on Run pipeline.

![image](https://github.com/user-attachments/assets/d45896cf-b06c-491a-9ce1-9bb7c2cab8b8)

2. Once the build process starts, select the build job to see the build in progress.

![image](https://github.com/user-attachments/assets/f501319d-dab6-47c8-9726-e886ca618b9a)

Note: I resolved the below error by renaming the project’s repository to aks-project (something that is accepted by the official Microsoft documentation).

![image](https://github.com/user-attachments/assets/fa2998ce-3451-409c-bfd4-fa235562d719)


Resolved build: 

![image](https://github.com/user-attachments/assets/5c039dfa-5356-443a-bb99-3528d6c3570c)

![image](https://github.com/user-attachments/assets/aff9816a-81c9-4c1c-b2b5-1ae2bbf1a316)

Note: 

- The build will generate and push the docker image to ACR. After the build is completed, you will see the build summary. <br />

- To view the generated images navigate to the Azure Portal, select the Azure Container Registry and navigate to the Repositories.

![image](https://github.com/user-attachments/assets/8cdc1d5e-6093-4dde-bf05-4052c2e22865)


3. Switch back to the Azure DevOps portal. Select the Releases tab in the Pipelines section and double-click on the latest release. Select In progress link to see the live logs and release summary.

![image](https://github.com/user-attachments/assets/34a8cfca-cf91-40ac-a24e-6904701b62e7)

![image](https://github.com/user-attachments/assets/6e6def2b-9c49-4fe7-81a1-d24e05d36d39)

4. Once the release is complete, launch the Azure Cloud Shell and run the below commands to see the pods running in AKS: <br />

a. Type az aks get-credentials --resource-group akshandsonlab --name aks-deploy-training in the command prompt to get the access credentials for the Kubernetes cluster. 

![image](https://github.com/user-attachments/assets/5f7709df-6797-453d-93c6-ee4bd507984b)

b. kubectl get pods

After running this command I have an issue, the frontend pod wa s not ready.

![image](https://github.com/user-attachments/assets/6e279a0f-b121-431d-a015-1f0fd6112e6f)

I checked kubectl logs showing mhc-front can’t connect SQL (sql server name in src/MyHealth.Web/appsettings.json has not been replaced). 

And the solution for this error was to make sure that the token patterns are set to _…_ in the “replace tokens” steps.

![image](https://github.com/user-attachments/assets/f6648bae-fb16-4582-8526-d7ae2cbba672)

![image](https://github.com/user-attachments/assets/05266526-46a6-41bd-a3ca-7e9ecbf51065)

I needed to start another release to make sure the changes are propagated in AKS.

![image](https://github.com/user-attachments/assets/35d76c98-a272-46fb-81e5-ae26d86556d8)

After that change and re-running the command both pods were ready:

![image](https://github.com/user-attachments/assets/cfb58451-2d95-4cfb-b5b0-12756c608112)

**Note:** The deployed web application is running in the displayed pods.

5. To access the application, run the below command. If you see that External-IP is pending, wait for sometime until an IP is assigned.

kubectl get service mhc-front --watch

![image](https://github.com/user-attachments/assets/b49c59f2-f551-4225-adec-d1b8611f4d3f)

Copy the External-IP and paste it in the browser and press the Enter button to launch the application.

![image](https://github.com/user-attachments/assets/689c7da8-60cb-4472-8a89-9b5d1943893d)

<h1> Kubernetes resources view in Azure portal </h1>

The Azure portal now offers a convenient Kubernetes resource viewer (preview), providing direct access to Kubernetes objects within your Azure Kubernetes Service (AKS) cluster. This eliminates the need to switch between the Azure portal and the kubectl command-line tool, simplifying the management of your Kubernetes resources. <br />

The resource viewer currently supports various resource types, including deployments, pods, and replica sets. It's important to note that this new feature replaces the AKS dashboard add-on, which is scheduled for deprecation.

![image](https://github.com/user-attachments/assets/c4af7e88-ad78-48d2-9743-5a4ac886ff68)

<h1> Conclusion </h1>

Azure Kubernetes Service (AKS) simplifies the management of Kubernetes clusters by handling many of the underlying operational tasks. In combination with Azure DevOps and Azure Container Services (AKS), you can create robust DevOps pipelines for Dockerized applications by utilizing Docker capabilities available on Azure DevOps Hosted Agents.

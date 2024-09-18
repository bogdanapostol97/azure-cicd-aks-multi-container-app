<h1> Technical documentation: Deployment pipeline for MyHealth.Web Application </h1>

This document explains the steps involved in the deployment pipeline for the MyHealth.Web application. This pipeline is triggered whenever there are changes pushed to the master branch of your code repository. 

It utilizes Azure DevOps and Docker Compose to automate the deployment process.

<h2> Pipeline stages </h2>

The pipeline can be broken down into several key stages:

**1. Trigger:** The pipeline is triggered automatically whenever there are code changes pushed to the master branch. <br />
**2. Environment:** The pipeline runs on a virtual machine with the Ubuntu-latest operating system. <br />
**3. Token replacement:** This stage replaces specific values (tokens) within configuration files with appropriate values for deployment. <br />

**Step 1:** Replaces tokens in appsettings.json located within the src/MyHealth.Web directory. <br />
**Step 2:** Replaces tokens prefixed and suffixed with double underscores (__) in the mhc-aks.yaml file. <br />

4. Building and Pushing Docker Images:

    **Step 1:** Runs Docker Compose services (defined in docker-compose.ci.build.yml) in detached mode (background process). This step likely builds and runs any dependencies or services required for building the application image. <br />
    **Step 2:** Builds Docker images for your application services defined in the docker-compose.yml file. It also tags the images with the current build ID for identification. <br />
    **Step 3:** Pushes the built Docker images to the Azure Container Registry (ACR) specified in the pipeline configuration. <br />

6. Locking Images (Optional):
This step (currently commented out) allows locking the image versions in the docker-compose.yml file. This can be useful for ensuring a specific version is deployed to staging or production environments. The locked configuration file is saved to a staging directory. <br />

7. Copying Deployment Artifacts:
This stage copies the mhc-aks.yaml file and any .dacpac files (likely database deployment scripts) to the artifact staging directory. <br />

8. Publishing Deployment Artifact:
This final stage publishes a compressed package (artifact) named "deploy" containing the copied files from the previous step. This artifact can be used to deploy the application to a specific environment (e.g., staging or production). <br />

<h2> Additional notes </h2>
This is a basic example, and additional steps might be required for a complete deployment process (e.g., deploying the application to a specific environment, running integration tests). <br />

The specific values for Azure resources (subscription ID, container registry details) are likely stored as secrets within Azure DevOps and referenced securely within the pipeline. <br />

This documentation can be further enhanced with links to relevant resources for deeper understanding of concepts like Docker Compose, Azure Container Registry, and DevOps pipelines. <br />

By understanding this documentation, you should be able to follow the automated deployment process for your MyHealth.Web application and make informed modifications to the pipeline configuration file. <br />

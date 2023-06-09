# Digital Desk Infrastructure Terraform

This project aims to create infrastructure using various AWS services for the Digital Desk application. The following AWS services are utilized:

## Services Overview:

* _S3 bucket_: The S3 bucket is used for storing and uploading objects. In this project, the DocumentS3Bucket is used for uploading files related to the project.

* _Lambda Function_: Lambda functions are utilized to trigger the AMQ (Advanced Message Queuing) service, which executes AMQ for establishing connections.

* _RDS_: The RDS MSSQL service is employed for database storage. It provides a reliable and scalable solution for storing and managing the application's data.

* _EKS_: EKS (Elastic Kubernetes Service) is used for cluster creation. The application is deployed using EKS, which provides a managed Kubernetes environment for seamless scaling and management.

* _AMQ_: AMQ serves as the Messenger Broker service setup. It facilitates the communication and exchange of messages between different components of the application.

* _CloudWatch_: CloudWatch is utilized for storing logs generated by the APIs. It provides monitoring and observability capabilities, allowing efficient tracking and analysis of application logs.

* Secret Manager: Secret Manager is used for securely storing sensitive information such as usernames and passwords required for AMQ.

## Terraform Scripts and Azure Pipeline:

Terraform scripts are created for provisioning and managing the AWS infrastructure components mentioned above. These scripts will be stored in an Azure repository. Azure Pipeline will then utilize these scripts for deployment, ensuring seamless and automated infrastructure provisioning on AWS.

## Build Pipeline

### Azure Build Pipeline Configuration

To deploy the infrastructure, we already have an Azure Repo set up. Now, let's configure the Azure Build pipeline to generate artifacts that will be used in the Azure release.

The following tasks are used in the Build pipeline:

* **Copy Files**: This task copies the necessary files from the Azure repo. Configure the task to select the required files and specify the destination folder.
  
* **Publish Artifact**: After copying the files, this task publishes the artifact that will be used in the release. Specify the artifact name and the location where it should be published.
  
### Hosted Agent Selection 
_Ondemand-window ADO_
In deployment process, we have opted to use the "Ondemand-window ADO" hosted agent. We have chosen this specific hosted agent for two primary reasons: dacpac deployment and internal private connectivity requirements.
_Dacpac Deployment_: The "Ondemand-window ADO" hosted agent provides support for dacpac deployment. Dacpac files are used for deploying and updating SQL Server databases. By using this hosted agent, we can seamlessly execute dacpac deployment tasks as part of our deployment pipeline.
_Internal Private Connectivity_: Our deployment process requires internal private connectivity to specific resources or networks. The "Ondemand-window ADO" hosted agent has been configured to facilitate this connectivity, ensuring secure and reliable communication between the agent and the required internal resources.

## Release Pipeline
### Azure Release Pipelines Configuration

In Azure release pipelines, we have artifacts, stages, tasks, and agent jobs, all working together to facilitate the deployment process.

* __Artifacts__: Artifacts are generated during the Build pipeline. These artifacts contain the necessary files and components required for the deployment process.
  
* __Stages__: The deployment process is divided into multiple stages. Currently, we have three stages: SB, DEV, and Shared (for the shared account). Each stage represents a specific environment or phase of the deployment.
  
* __Agent Job__: Agent jobs are responsible for executing multiple tasks within a stage. Each stage can have multiple agent jobs that run in parallel or sequentially, depending on the configuration. Agent jobs combine the necessary tasks to create the complete deployment process.
  
* __Tasks__: Within each stage, various tasks are executed to accomplish specific actions. In our case, we utilize different Terraform tasks to deploy the Terraform scripts stored in the repository. These Terraform tasks include "Terraform Install" and "Terraform CLI Commands."
  
To deploy our Terraform scripts, we have defined the following tasks in our Azure DevOps pipeline:
* _Terraform Install_: This task is responsible for installing the latest version of Terraform before executing any Terraform CLI commands. It ensures that the required Terraform configuration is available and up to date for the deployment process.
* _Terraform CLI Commands_: In this task, we utilize various Terraform CLI commands to execute the Terraform scripts correctly. The following commands are commonly used:
•_INIT_: The terraform init command is used to initialize the Terraform working directory. It downloads the necessary provider plugins and sets up the backend configuration.
• _PLAN_: The terraform plan command is used to create an execution plan for the deployment. It compares the desired state defined in the Terraform scripts with the current state of the infrastructure and generates a plan outlining the changes that will be made.
•_APPLY_: The terraform apply command is used to apply the changes defined in the Terraform scripts to the infrastructure. It creates or modifies resources according to the desired state.

## Database Deployment
For the deployment of the database, we are utilizing dacpac, and we have created separate build and release pipelines to manage the process effectively.

### DB Build Pipeline
In the DB build pipeline, multiple tasks are employed to facilitate the build process. The following tasks are included:

* __Build Solution__: This task builds the solution for the database project, generating the necessary artifacts.

* __Publish Symbols Path__: The task publishes the symbols path, enabling better debugging and error analysis if needed.

* __Publish Artifact__: This task publishes the artifacts generated from the build process, making them available for the release pipeline.

* __Copy Files__: The task copies the required files to a designated location for further use.

### DB Release Pipeline
In the release pipeline, the database deployment is divided into two sections: DB Database and DB Migration. Each section consists of different tasks tailored for their respective purposes.

##### _DB Database_
In the DB Database deployment section, the following tasks are executed:

* __Azure App Configuration__: This task is used to store data variables required for the deployment process.

* __Deploy using dacpac__: This task performs the deployment of the dacpac database. It leverages the dacpac file to deploy the database schema and objects.

##### _DB Migration_
In the DB Migration section, the following task is performed after the dacpac deployment:

* __Data Migration__: This task handles the migration of data within the database. It ensures that the necessary data is migrated or transformed according to the requirements.
These tasks collectively manage the deployment process for the database, including schema deployment (using dacpac) and subsequent data migration.

## API Deployment
For the deployment of APIs, separate build and release pipelines have been created for each API. In total, there are 15 APIs in this project, and each API has its own build and release pipelines.

### Build Pipeline
In the Build pipeline, multiple tasks are utilized to create the build for API deployment. The tasks involved are as follows:

* __dotnet restore__: This task is used to restore the dotnet files, ensuring that all the required dependencies are available.

* __dotnet publish__: The task publishes the dotnet files after the restoration process. It prepares the API for deployment by generating the necessary artifacts.
* __Build__: This task builds the Docker image for the API. It compiles the necessary source code and configurations to create the image.

* __Push Image__: The task pushes the Docker image to the ECR (Elastic Container Registry) repository. It makes the image available for deployment on the Kubernetes cluster.

* __Copy Files__: This task copies the required files to a designated location for further use.

* __Publish Artifact__: The task publishes the artifacts generated from the build process, making them available for the release pipeline.

### Release Pipeline
In the Release pipeline for API deployment, multiple Kubernetes tasks are used to manage the deployment process. The tasks involved are as follows:

* __Azure App Configuration__: This task fetches the variables required for the deployment from Azure App Configuration.

* __*Replace tokens in /*.json__: The task replaces the variables in the JSON files with the corresponding values fetched from Azure App Configuration.

* __kubectl apply ingress__: This task applies the ingress configuration for routing external traffic to the appropriate API endpoints.

* __kubectl apply namespace__: The task applies the namespace configuration for isolating and organizing the API within the Kubernetes cluster.

* __kubectl apply service__: This task applies the service configuration, exposing the API internally within the cluster.

* __kubectl apply deployment__: The task applies the deployment configuration, ensuring that the API is running as per the desired state.

* __Naming Convention__:This task applies convention or standard followed in Azure pipelines for organizing and naming tasks, job groups, stages, variables, or other pipeline components.
  
*  __Task Group__: In the task group, all the tasks required for multiple APIs are combined. This allows for easy attachment of the task group to each API, ensuring that the tasks are executed correctly.

## UI Deployment
For the deployment of UIs, separate build and release pipelines have been created. Let's explore the details of the build pipeline.

### Build Pipeline
In the Build pipeline, several tasks are employed to create the build for UI deployment. The following tasks are involved:

* __Use Node 12.20.0__: This task sets the Node.js version to 12.20.0, ensuring that the build process uses the specified Node.js environment.

* __npm install__: The task executes the npm install command, which installs the necessary npm packages and dependencies required for the UI.

* __Copy Files__: This task copies the required files to a designated location for further use.

* __Publish Artifact__: The task publishes the artifacts generated from the build process, making them available for the release pipeline.

### Release Pipeline
In the Release pipeline for UI deployment, multiple Kubernetes tasks,Naming Convention are used to manage the deployment process. The tasks involved are as follows:

* __Azure App Configuration__: This task fetches the variables required for the deployment from Azure App Configuration.

* __*Replace tokens in /*.json__: The task replaces the variables in the JSON files with the corresponding values fetched from Azure App Configuration.

* __kubectl apply ingress__: This task applies the ingress configuration for routing external traffic to the appropriate API endpoints.

* __kubectl apply namespace__: The task applies the namespace configuration for isolating and organizing the API within the Kubernetes cluster.

* __kubectl apply service__: This task applies the service configuration, exposing the API internally within the cluster.

* __kubectl apply deployment__: The task applies the deployment configuration, ensuring that the API is running as per the desired state.
* __Naming Convention__:This task applies convention or standard followed in Azure pipelines for organizing and naming tasks, job groups, stages, variables, or other pipeline components.
*  __Task Group__: In the task group, all the tasks required for multiple APIs are combined. This allows for easy attachment of the task group to each API, ensuring that the tasks are executed correctly.



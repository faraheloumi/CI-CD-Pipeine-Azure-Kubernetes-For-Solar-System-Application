# CI/CD Pipeline on Gitlab for Solar system application deployed with Azure Kubernetes

## 📖 **Table of Contents**

- [📌 Project Overview](#-project-overview)
- [🌱 About the Original Project](#-about-the-original-project)
- [🔑 Key Objectives](#-key-objectives)  
- [⚙️ Technologies Used](#️-technologies-used)  
- [🏛️ Architecture](#-architecture)  
- [☸️ Kubernetes Architecture and Deployment](#-kubernetes-architecture-and-deployment)  
- [📊 Results: CI/CD pipeline](#-results-cicd-pipeline)
- [🔧 Setup and Usage](#-setup-and-usage)  
- [🔮 Future Considerations](#-future-considerations) 
- [👨‍💻 Project By](#project-by) 

---
## 📌 **Project Overview**

This repository builds upon the Solar System project, initially developed with HTML for the frontend, Express.js for the backend, and MongoDB as the database. It enhances the project by automating and optimizing the development and deployment processes, as outlined in the future considerations. The project incorporates a **GitLab CI/CD pipeline** with integrated **development environments** and **testing workflows**.

This pipeline enhances the original project by automating complex workflows, improving reliability, and reducing deployment time.  

This is a project by Farah Elloumi.

### 🌱 **About the Original Project**

The original **Solar System** project focuses on developing a platform that provides information about the solar system and a description of each solar system corresponding to its respective number. The web application is integrated into a user-friendly application with a backend (NodeJs) and a persistent database (MongoDB) to track the solars databases.

## 🔑 **Key Objectives**

- 🛡️ **Security and Vulnerability Scanning**: Ensure code, secrets, and container images are secure by identifying and mitigating vulnerabilities during the CI/CD pipeline.  
- ✅ **Automated Testing**: Automatically test all changes to ensure functionality and stability.  
- 🐳 **Dockerization**: Build, test, and package services into portable Docker images.  
- ☸️ **Kubernetes Deployment**: Deploy services seamlessly across multiple environments (**Dev**) using Kubernetes clusters. 
- 🔄 **Integration Testing**: Validate the functionality and reliability of all services after deployment.  
- 🔗 **Post-Deployment Automation**: Ensure automated deployment of the application by updating the deployment URL dynamically in the web app to ensure seamless deployment.
 

## 🏛️ Architecture
The Gitlab CI/CD pipeline follows a **stage-by-stage process** to automate development and deployment. Below is the visual workflow:

<div align="center">
    <img src="./images/CICD pipeline.png" alt="CI/CD pipeline workflow">
</div>

The GitLab CI/CD pipeline automates the **integration, testing, and deployment** processes for the application, ensuring reliable and consistent delivery across different environments.

The pipeline is divided into two main phases: **Continuous Integration** and **Continuous Deployment & Delivery**.

### ⚙️ **Workflow Stages**

1. **Continuous Integration**  
   - **Automated Build and Test**:   
     - Run **Unit Tests** using `Jest` to verify the backend functionality (NodeJS app) and validate the connection to MongoDB. 
     - If all tests pass:  
       - Build the **Docker image**.  
       - Test the Docker image functionality.
     - **Failure Handling**: If any step fails, the developer must fix the code and push changes again until all checks pass.  
   - **Push to DockerHub**:  
     - Store validated and tested Docker images in **DockerHub** for use in deployment.

2. **Continuous Deployment & Delivery**  
   - **Dev Deployment**:  
     - Deploy the different services to the **Development Environment**, on the **Azure Kubernetes Cluster**, for initial validation.  
     - Used for coding, testing, and debugging during development.  
     - Run **integration tests** to ensure the deployment works as expected.  

## ⚙️ **Technologies Used**

| **Component**         | **Technology**                     |
|------------------------|------------------------------------|
| **CI/CD**             | GitLab CI/CD                       |
| **Testing**           | Jest (Node.js Backend)             |
| **Containerization**  | Docker             |
| **Orchestration**     | Kubernetes                         |
| **Cloud Deployment**  | Azure Kubernetes Service (AKS)     |


## ☸️ Kubernetes Architecture and Deployment

### Overview
The Kubernetes architecture for the Solar System project enables scalable, secure, and automated deployment of application services in the development environment. Using **Kubernetes Ingress**, services are exposed to external users with **load balancing** and **HTTPS encryption**.

### Goals
1. **Service Management**: Efficiently manage backend service using Kubernetes resources such as:
   - **Deployments**: Ensure high availability and scalability of application pods.  
   - **Services**: Enable internal communication and expose pods to external traffic.  

2. **Dynamic Traffic Routing**: Use **Ingress** to expose services to the internet with **SSL/TLS termination** provided by **Let's Encrypt**, ensuring secure and efficient traffic handling.  
Ingress is suitable for exposing **multiple services under a single domain** or for setting up more complex routing rules.

3. **Stability and Recovery**:
   - Rollback functionality ensures stability by enabling a return to a previous deployment in case of issues.  
   - Minimizes downtime and preserves reliability for end users.  

### Kubernetes Components
The deployment uses the following components:

1. **Deployments**:
   - Manage single pod for **backend** service based on the docker images pushed to DockerHub.
   - Ensure high availability with configurable replicas (`REPLICAS` in deploy.yaml).

2. **Services**:
   - Expose backend (`/`) service internally using ClusterIP.
   - Facilitate internal communication between pods.

3. **Ingress**:
   - Use **Azure Kubernetes Ingress Controller** (`webapprouting.kubernetes.azure.com`) for routing traffic to services.
   - Provide secure HTTPS access to services with **TLS certificates** from **Let's Encrypt** (`clusterIssuer.yaml`).
   - Suitable for routing requests under the URL :  
     ```
     solar-system.${INGRESS_IP}.nip.io
     ```
   - This URL ensures that requests are routed appropriately based on the namespace environment `${NAMESPACE}` .
   - The **Ingress IP** is the public IP address assigned to the **Azure Kubernetes Ingress Controller**. It serves as the entry point for all external traffic directed to the Kubernetes cluster. It can be obtained by:
    ```
    INGRESS_IP=$(kubectl get service -n app-routing-system nginx -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
    ```

4. **Secrets**:
   - Store sensitive credentials (e.g., MongoDB connection details) securely using Kubernetes Secrets.


### Kubernetes Cluster and CI/CD Integration

#### Cluster Creation
- The Kubernetes cluster was created using the Azure CLI command:
```bash
az aks create --resource-group solarsystem_grp --name solar-systemCluster --tier free --generate-ssh-keys --node-vm-size Standard_B2s --node-count 2 --enable-app-routing
```
- Key Flags Explained:

  - `--generate-ssh-keys`: Automatically generates SSH keys for secure access to the Kubernetes nodes, simplifying authentication.
  - `--enable-app-routing`: Configures an ingress controller (Azure Kubernetes Ingress Controller) for traffic routing, enabling secure and efficient access to application services.

This cluster supports **development** namespace

#### Kubeconfig File
To allow interaction with the Kubernetes cluster, the kubeconfig file was retrieved using:
```
  az aks get-credentials --resource-group solarsystem_grp --name solar-systemCluster
```
- The kubeconfig file provides cluster authentication details and is integrated into GitLab CI/CD as a variable (`DEV_KUBE_CONFIG`) to enable automated deployments to the **development** namespace.

## 📊 Results: CI/CD pipeline

<div align="center">
    <img src="./images/Resultat pipeline.png" alt="CI/CD pipeline feature">
</div>

## 🔧 Setup and Usage

### Prerequisites
- **GitLab Account**: Free plan is sufficient.  
- **Cloud Provider**: A cloud provider offering managed Kubernetes services (**Azure Kubernetes Service** is used here, leveraging the $100 credit for students enrolled in a university).  

### Steps to Run

#### 1. **Clone the Repository**  
   ```bash
   git clone https://github.com/faraheloumi/CI-CD-Pipeline-Azure-Kubernetes-For-Solar-System-Application.git
   cd CI-CD-Pipeline-Azure-Kubernetes-For-Solar-System-Application
   ```
#### 2. **Set Up a GitLab Project**
- Create a new project in GitLab.  
- Add the GitLab CI/CD variables:  
   - Go to **Settings → CI/CD → Variables → Add variables**:
   - Uncheck the options in the **Flags** field

| **Key**           | **Value**                 | **Visibility**  |
|-------------------|---------------------------|------------|
| `DOCKER_USERNAME` | `<your_dockerhub_username>` | Visible    |
| `DOCKER_PASSWORD` | `<your_dockerhub_password>` | Masked     |
| `MONGO_PASSWORD`  | `<your_mongodb_password>`  | Masked     |
| `NAMESPACE`       | `development`             | Visible    |
| `REPLICAS`        | `2`                       | Visible    |


#### 3. **Create an Azure Kubernetes Cluster**
- Log in to Azure CLI:
  ```bash
  az login
  ```
- Create the AKS Cluster:
 ```bash
  az aks create --resource-group solarsystem_grp --name solar-systemCluster --tier free --generate-ssh-keys --node-vm-size Standard_B2s --node-count 2 --enable-app-routing
 ```

 #### 4. **Retrieve the Kubeconfig File**
 - Fetch the credentials for your AKS cluster:
 ```bash
  az aks get-credentials --resource-group solarsystem_grp --name solar-systemCluster
 ```
 - Copy the content of the Kubeconfig file and add it to GitLab CI/CD variables:
    - **Key**: DEV_KUBE_CONFIG
    - **Value**: The content of the Kubeconfig file
    - **Type**: File
    - **Visibility**: Visible
    - **Flags**: uncheck them

#### 5. **Prepare Kubernetes Namespaces**
- Install `kubectl` on your local machine following [this guide.](https://kubernetes.io/docs/tasks/tools/#kubectl)
- Create namespaces for your environments in your local terminal:
```bash
    kubectl create namespace development
 ```

### 6. **Push the Code to GitLab**
Push the files in the cloned directory to the main branch:
```bash
git push origin main
```

### 7. **Trigger the Pipeline**  
This will automatically trigger the CI/CD pipeline on the `main` branch.
- Go to **Build → Pipelines**.  
- Click on the running pipeline to monitor the jobs.  


## 🔮 Future Considerations

1. Extend the current setup to include **staging and production environments** in addition to **the development environment**. This will enable thorough testing and validation of changes in staging before rolling them out to production, ensuring higher stability and reliability.

2. Adopt a **feature branch workflow** for development. All changes will first be pushed to dedicated feature branches. Once the features are reviewed, tested, and approved, they **will be merged into the main branch** to maintain a clean and stable codebase. This approach will improve collaboration, version control, and overall code quality.

3. **Integration of Continuous Security Practices**:  
   Security enhancements can be integrated into the CI/CD pipeline to ensure robust protection of the application and user data:
   - Adding runtime security checks to detect anomalies in deployed services.
   - Utilizing Kubernetes features like Pod Security Policies and Network Policies to limit attack surfaces and enforce security best practices.

## Project by
<a href="https://github.com/faraheloumi/CI-CD-Pipeline-Azure-Kubernetes-For-Solar-System-Application/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=faraheloumi/CI-CD-Pipeline-Azure-Kubernetes-For-Solar-System-Application" />
</a>
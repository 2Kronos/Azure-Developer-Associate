## Definitions 

## **Examine Azure App Service**

### **Overview**
Azure App Service is a fully managed platform designed to simplify the deployment and scaling of web apps, mobile back ends, and RESTful APIs. It abstracts infrastructure management, allowing developers to focus on writing code. Applications can be built using preferred stacks (.NET, Java, Node.js, Python, PHP) and deployed to Windows or Linux environments, with support for custom containers.

### **Built-in auto scale support**
The service provides integrated scaling capabilities:
- You can basically scale the resources of the underlying machine that is hosting your web app either up or down 
- **Scale up/down**: Adjust the resources (CPU cores, RAM) of the underlying machine hosting your app.
- **Scale out/in**: Increase or decrease the number of machine instances running your app based on usage.

### **Container support**
- Deploy and run containerized web apps on Windows and Linux.
- Pull images from Azure Container Registry or Docker Hub.
- Supports multi-container apps, Windows containers, and Docker Compose orchestration.

### **Continuous integration/deployment support**
- Out-of-the-box CI/CD integration with Azure DevOps, GitHub, Bitbucket, FTP, or local Git repositories.
- Automatic code sync and deployment when changes are pushed to a connected repository.
- Supports containerized app deployment via Azure Container Registry or Docker Hub.

### **Deployment slots**
- Available in the Standard App Service pricing tier or higher.
- Deployment slots are live apps with separate hostnames.
- Allows swapping of app content and configurations between slots (e.g., staging and production) for controlled deployments.

### **App Service on Linux**
- Hosts web apps natively on Linux for supported application stacks or via custom Linux containers (Web App for Containers).
- Built-in images support .NET Core, Java (Tomcat, JBoss EAP, Java SE), Node.js, Python, and PHP.
- Retrieve current supported runtimes using: `az webapp list-runtimes --os-type linux`

**Limitations of App Service on Linux:**
- Not supported on the Shared pricing tier.
- The Azure portal only displays features currently enabled for Linux apps.
- Apps using built-in images store code/content on an Azure Storage volume, which has higher and more variable latency than the container filesystem. Read-heavy apps may benefit from custom containers.
- Custom containers place files in the container filesystem, improving performance for content-heavy operations.

### **App Service Environment**
- A feature providing a fully isolated and dedicated environment for running App Service apps.
- Offers enhanced security and high-scale capabilities.
- Compute resources are dedicated to a single customer, unlike the multi-tenant App Service offering.
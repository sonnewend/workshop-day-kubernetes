# Workshop Day Kubernetes e AKS

Mentoria Arquiteto Cloud - Hands-on Lab

## Cloud-Native Applications

Deploy a Web application and API Microservice to a Kubernetes platform hosted on Azure Kubernetes Services (AKS)

   ![Screenshot of the App Modernization](/AllFiles/Images/PartsUnlimited.png)

## Requirements Hands-on lab

1. An active Microsoft Azure subscription in preference be Pay-as-you-go or MSDN.

   - You must have enough cores available in your subscription to create the build agent and Azure Kubernetes Service cluster in Before the Hands-on Lab. This lab requires a minimum of eight cores which may demand more cluster resources than your current quota will allow. In such cases, you can request a CPU core quota increase to deploy the AKS cluster.

  - **Important:** To complete this lab, you must have sufficient rights within your Azure AD tenant to register resource providers in your Azure Subscription.

2. An active [GitHub Account](https://github.com) in Free or Pay plans.

3. Local machine or a virtual machine configured with:

   - A browser, such as Microsoft Edge or Google Chrome, for consistency with the lab implementation tests.

4. You will install other tools throughout the exercises.

> **Very important**: Make sure to type all the commands as they appear in the guide. Do not try to copy and paste to your command windows or other documents when instructed to enter the information shown in this document, except where explicitly stated in this document. There can be issues with Copy and Paste that result in errors, instructions execution, or file content creation.

## Hands-on project 

Fabrikam Medical Conferences (FabMedical) provides conference website services tailored to the medical community. They are refactoring their application to run as a Docker application. They want to implement a proof of concept that will help them get familiar with the development process, lifecycle of deployment, and critical aspects of the hosting environment. They will be deploying their applications to Azure Kubernetes Service and want to learn how to deploy containers in a dynamically load-balanced manner, discover containers, and scale them on demand.

In this hands-on lab, you will assist with completing this POC with a subset of the application codebase. You will create a build agent based on Linux and an Azure Kubernetes Service cluster for running deployed applications. You will be helping them to complete the Docker setup for their application, test locally, push to an image repository, deploy to the cluster, and test load-balancing and scale.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully to understand the whole of the solution as you are working on the various components.

   ![Screenshot of the Solution topology](/AllFiles/Images/Solution-Topology.png)

Each tenant will have the following containers:

- **Conference Web site**: The single page application (SPA) application that uses configuration settings to handle custom styles for the tenant.

- **Admin Web site**: The SPA application that conference owners use to manage conference configuration details, attendee registrations, campaigns, and communications with attendees.

- **Registration service**: The API that handles all registration activities creating new conference registrations with the appropriate package selections and associated cost.

- **Email service**: The API that handles email notifications to conference attendees during registration or when the conference owners engage the attendees through their admin site.

- **Config service**: The API that handles conference configuration settings such as dates, locations, pricing tables, early-bird specials, countdowns, and related.

- **Content service**: The API that handles content for the conference, such as speakers, sessions, workshops, and sponsors.

## Prereqs Hands-on lab (15 minutes)

You should follow all the steps provided in this section _before_ taking part in the hands-on lab ahead of time as some of these steps take time.

### Set up Azure Cloud Shell

1. Open a Cloud shell by selecting the cloud shell icon in the menu bar.

2. The cloud shell opens in the browser window. Choose **Bash** if prompted or use the left-hand dropdown on the shell menu bar to choose **Bash** from the dropdown (as shown). If prompted, select **Confirm**.

3. Make sure to set your default subscription correctly. To view your current subscription type:

   ```bash
   az account show
   ```
4. If the subscription displayed in the previous step is not the subscription you plan on using for this workshop, you will need to set the current subscription to your desired subscription. To set your default subscription to something other than the current selection, type the following, replacing {id} with the desired subscription id value:

   ```bash
   az account set --subscription {id}
   ```

   > **Note**: If you do not know the id of your desired subscription, you can list the subscriptions available to you along with their ids via the following command:

   ```bash
   az account list
   ```

### Download Starter Files repository

> **Note**: You will need access to your Azure subscription via Azure Cloud Shell to proceed further in the workshop.  If you don't have a cloud shell available, refer back to [Task 1: Set up Azure Cloud Shell](#task-1-set-up-azure-cloud-shell) for set up instructions.

1. Check out the starter files from the MCW Cloud-native applications GitHub repository and detach them from the existing remote repository via the following commands:

   ```bash
   cd ~
   git clone \
      --depth 1 \
      --branch main \
      https://github.com/maiaacademy/MCW-Cloud-native-applications.git \
      MCW-Cloud-native-applications
   cd MCW-Cloud-native-applications
   ```

   > **Note**: If you do not have enough free space, you may need to remove extra files from your Cloud Shell environment.  Try running `azcopy jobs clean` to remove any `azcopy` jobs and data you do not need.


### Create a GitHub repository

FabMedical has provided starter files for you. They have taken a copy of the websites for their customer Contoso Neuro and refactored it from a single node.js site into a website with a content API that serves up the speakers and sessions. This refactored code is a starting point to validate the containerization of their websites. Use this to help them complete a POC that validates the development workflow for running the website and API as Docker containers and managing them within the Azure Kubernetes Service environment.

1. Open a web browser and navigate to <https://www.github.com>. Log in using your GitHub account credentials.

2. In the upper-right corner, expand the user drop-down menu and select **Your repositories**.

3. Next to the search criteria, locate and select the **New** button.

4. On the **Create a new repository** screen, name the repository **Fabmedical** and select the **Create repository** button.

5. On the **Quick setup** screen, copy the **HTTPS** GitHub URL for your new repository, paste this into notepad for future use.

### Set up Azure Cloud Shell environment 

1. A GitHub personal access token (PAT) with appropriate permissions is required to set up and complete this lab - [Follow this link](https://github.com/settings/tokens/new?scopes=repo&description=GitHub%20Secrets%20CLI) to quickly set up a GitHub personal access token with the required permissions. Save the obtained PAT as it will be needed by future steps.

> **Note**: Make sure to select the `workflow` and `admin:org read:org` permission scopes in addition to the `repo` scopes already selected when visiting the aforementioned link.

2. Set the following environment variables in an Azure Cloud Shell terminal.

   ```bash
   export MCW_SUFFIX=<SUFFIX>                            # Needs to be a unique three letter string
   export MCW_GITHUB_USERNAME=<GITHUB USERNAME>          # Your Github account username
   export MCW_GITHUB_TOKEN=<GITHUB PAT>                  # A personal access token for your Github account
   export MCW_AZURE_SUBSCRIPTION=<AZURE SUBSCRIPTION ID> # The target Azure Subscription ID
   export MCW_GITHUB_URL - Defaults=<GITHUB URL PATH> # Default 'https://github.com/$MCW_GITHUB_USERNAME/Fabmedical'
   export MCW_PRIMARY_LOCATION=<AZURE PRIMARY REGION> # Defaults to `northeurope`
   export MCW_PRIMARY_LOCATION_NAME<AZURE PRIMARY REGION NAME> # Defaults to `North Europe`
   export MCW_SECONDARY_LOCATION=<AZURE SECONDAY REGION> # Defaults to `westeurope`
   export MCW_SECONDARY_LOCATION_NAME=<AZURE SECONDAY REGION NAME> # Defaults to `West Europe`
   ```

   > **Note**: The following environment variables can also be set if their defaults are not appropriate for the lab setting or environment.

   > **Note**: If you run into the error below, you may have to either use a different regional pair or increase your regional core quotas in your current regions. This lab's cloud resources require at least eight available cores in your regional core quota. An up to date list of Azure Region Pairs can be found at [this link](https://docs.microsoft.com/en-us/azure/best-practices-availability-paired-regions#azure-regional-pairs "Azure Region Pairs").

      ```bash
      {"error":{"code":"InvalidTemplateDeployment","message":"The template deployment 'azuredeploy' is not valid according to the validation procedure. The tracking id is '3d4adbc2-647b-4741-8d98-fe20495e0541'. See inner errorsfor details.","details":[{"code":"QuotaExceeded","message":"Provisioning of resource(s) for container service fabmedical-??? in resource group fabmedical-??? failed. Message: {\n  \"code\": \"QuotaExceeded\",\n  \"message\": \"Provisioning of resource(s) for container service fabmedical-??? in resource group fabmedical-??? failed. Message: Operation could not be completed as it results in exceeding approved Total Regional Cores quota. Additional details - Deployment Model: Resource Manager, Location: eastus, Current Limit: 10, Current Usage: 8, Additional Required: 4, (Minimum) New Limit Required: 12. Submit a request for Quota increase at https://aka.ms/ProdportalCRP/#blade/Microsoft_Azure_Capacity/UsageAndQuota.ReactView/Parameters/%7B%22subscriptionId%22:%228c924580-ce70-48d0-a031-1b21726acc1a%22,%22command%22:%22openQuotaApprovalBlade%22,%22quotas%22:[%7B%22location%22:%22eastus%22,%22providerId%22:%22Microsoft.Compute%22,%22resourceName%22:%22cores%22,%22quotaRequest%22:%7B%22properties%22:%7B%22limit%22:12,%22unit%22:%22Count%22,%22name%22:%7B%22value%22:%22cores%22%7D%7D%7D%7D]%7D by specifying parameters listed in the ‘Details’ section for deployment to succeed. Please read more about quota limits at https://docs.microsoft.com/en-us/azure/azure-supportability/regional-quota-requests. Details: \"\n }. Details: "}]}}
      ```

3. Configure your Git email and name.

   ```bash
   git config --global user.email "your@email.com"
   git config --global user.name "Your Name"
   ```

4. Run the `create_azure_resources.sh` script in the `MCW-Cloud-native-applications` repository that was cloned in a previous step. This will provision all of the Azure cloud resources necessary to execute the workshop.

   ```bash
   cd ~/MCW-Cloud-native-applications/Hands-on\ lab/lab-files/developer/scripts
   bash create_azure_resources.sh
   ```

5. Upon successful execution of the `create_azure_resources.sh` script, a command for establishing an SSH session to the build agent VM should be present in the output.

   ```bash
   Command to create an active session to the build agent VM:

       ssh -i ~/.ssh/fabmedical adminfabmedical@<PUBLIC IP OF VM>
   ```

6. Use the SSH command output in the previous step to establish an SSH session to the build agent VM.  You should be presented with a prompt similar to the following:

   `adminfabmedical@fabmedical-SUFFIX:~$`


### Complete the build agent setup

1. From an Azure Cloud Shell terminal, use the SSH command output from the previous task and start an active SSH session to the build agent VM.

2. Clone the FabMedical GitHub repository created in the previous task.

   ```bash
   git clone https://github.com/<GITHUB_USERNAME>/Fabmedical
   ```

3. Set the following environment variables in the active SSH session to the build agent VM. Use the same GitHub access token and Azure subscription ID used in a previous task.

   ```bash
   export MCW_SUFFIX=<SUFFIX>                            # Needs to be a unique three letter string
   export MCW_GITHUB_USERNAME=<GITHUB USERNAME>          # Your Github account username
   export MCW_GITHUB_TOKEN=<GITHUB PAT>                  # A personal access token for your Github account
   export MCW_AZURE_SUBSCRIPTION=<AZURE SUBSCRIPTION ID> # The target Azure Subscription ID 
   ```

4. Run the `create_build_environment.sh` script to set up the build agent VM environment. This script installs necessary dependencies on the build agent VM and applies the configuration settings to the VM's environment necessary for proper execution of the workshop.

   ```bash
   cd ~/Fabmedical/scripts
   bash create_build_environment.sh
   ```

   > **Note**: Ignore any errors you encounter regarding the Docker client. That will be resolved after joining a new SSH session in the following steps.

5. After the script completes execution, type `exit` to exit the SSH session. We will need to join a new SSH session to ensure the docker environment on the build agent VM has completed set up.

6. Reestablishing an SSH session in the previous steps: 

   `adminfabmedical@fabmedical-SUFFIX:~$`

6. After reestablishing an SSH session to the build agent VM and run the `create_and_seed_database.sh` script to create and seed the MongoDB database for use in the workshop.

   ```bash
   cd ~/Fabmedical/scripts
   bash create_and_seed_database.sh
   ```

### Build Docker Images

1. Navigate to the `content-api` directory and build the `content-api` container image using the Dockerfile in the directory. Note how the deployed Azure Container Registry is referenced. Replace the `SUFFIX` placeholder in the command.


   ```bash
   cd ~/Fabmedical/content-api
   docker image build -t fabmedical[SUFFIX].azurecr.io/content-api:latest .
   ```

2. Repeat this step for the `content-web` image, which serves as the application front-end.

   ```bash
   cd ~/Fabmedical/content-web
   docker image build -t fabmedical[SUFFIX].azurecr.io/content-web:latest .
   ```

3. Observe the built Docker images by running `docker image ls`. The images were tagged with `latest`, but it is possible to use other tag values for versioning.

4. Log in to Azure Container Registry using `docker login fabmedical[SUFFIX].azurecr.io`. Fetch the credentials from the **Access keys** tab of the ACR instance in the Azure portal.

5. Push the two images you built.

   ```bash
   docker image push fabmedical[SUFFIX].azurecr.io/content-api:latest
   docker image push fabmedical[SUFFIX].azurecr.io/content-web:latest
   ```


## Exercise 1: Migrate MongoDB to Cosmos DB using Azure Database Migration Service (30 min)

The next step is to migrate the MongoDB database data to Azure Cosmos DB. This exercise will use the Azure Database Migration Service to migrate the data from the MongoDB database into Azure Cosmos DB.

### Enable Microsoft.DataMigration resource provider

In this task, you will enable the use of the Azure Database Migration Service within your Azure subscription by registering the `Microsoft.DataMigration` resource provider.

1. Open the Azure Cloud Shell.

2. Run the following Azure CLI command to register the `Microsoft.DataMigration` resource provider in your Azure subscription:

   ```sh
   az provider register --namespace Microsoft.DataMigration
   ```

### Provision Azure Database Migration Service

This task will deploy an instance of the Azure Database Migration Service used to migrate the data from MongoDB to Cosmos DB.

1. From the Azure Portal, select **+ Create a resource**.

2. Search the marketplace for **Azure Database Migration Service** and select it.

3. Select **Create**.

4. Select the `Migrate my SQL Server, MySQL, PostgresQL, or MongoDB database(s) to Azure` option when prompted.

5. On the **Basics** tab of the **Create Migration Service** pane, enter the following values:

    - **Resource group**: Select the Resource Group created with this lab.
    - **Migration service name**: Enter a name, such as `fabmedical[SUFFIX]`.
    - **Location**: Choose the Azure Region used for the Resource Group.

6. Select **Next: Networking >>**.

7. On the **Networking** tab, select the **Virtual Network** within the `fabmedical-[SUFFIX]` resource group.

8. Select **Review + create**.

9. Select **Create** to create the Azure Database Migration Service instance.

It may take 5 to 10 minutes to provision the Azure Database Migration Service instance.

### Migrate data to Azure Cosmos DB

In this task, you will create a **Migration project** within Azure Database Migration Service and then migrate the data from MongoDB to Azure Cosmos DB.

1. Navigate to your Build Agent VM in the Azure Portal and copy the Private IP address **(2)**. Then, paste the contents into the text editor of your choice (such as Notepad on Windows, macOS users can use TextEdit) for future use.

2. In the Azure Portal, navigate to the **Azure Database Migration Service** created in a previous step..

3. On the Azure Database Migration Service blade, select **+ New Migration Project** on the **Overview** pane.

4. On the **New migration project** pane, enter the following values, then select **Create and run activity**:

    - **Project name**: fabmedical
    - **Source server type**: MongoDB
    - **Target server type**: CosmosDB (MongoDB API)
    - **Choose type of activity**: Offline data migration

    > **Note:** The **Offline data migration** activity type is selected since you will be performing a one-time migration from MongoDB to Cosmos DB. Also, the migration operation will not update data in the database. In a production scenario, you will choose the migration project activity type that best fits your solution requirements.

5. On the **MongoDB to Azure Database for CosmosDB Offline Migration Wizard** pane, enter the following values for the **Select source** tab:

    - **Mode**: Standard mode
    - **Source server name**: Enter the Private IP Address of the Build Agent VM used in this lab.
    - **Server port**: 27017
    - **Require SSL**: Unchecked

    > **Note:** Leave the **User Name** and **Password** blank as the MongoDB instance on the Build Agent VM for this lab does not have authentication turned on. The Azure Database Migration Service resides in the same VNet as the Build Agent VM, so it can communicate within the VNet directly to the VM without exposing the MongoDB service to the Internet. In production scenarios, you should always have authentication enabled on MongoDB.

6. Select **Next: Select target >>**.

7. On the **Select target** pane, select the following values:

    - **Mode**: Select Cosmos DB target

    - **Subscription**: Select the Azure subscription you're using for this lab.

    - **Select Cosmos DB name**: Select the `fabmedical-[SUFFIX]` Cosmos DB instance.

    Notice, the **Connection String** will automatically populate with the Key for your Azure Cosmos DB instance.

8. Select **Next: Database setting >>**.

10. On the **Database setting** tab, select the `contentdb` **Source Database** so this database from MongoDB will be migrated to Azure Cosmos DB.

11. Select **Next: Collection setting >>**.

12. On the **Collection setting** tab, expand the **contentdb** database, and ensure both the **sessions** and **speakers** collections are selected for migration. Also, update the **Throughput (RU/s)** to `400` for both collections.

13. Select **Next: Migration summary >>**.

14. On the **Migration summary** tab, enter `MigrateData` in the **Activity name** field, then select **Start migration** to initiate the migration of the MongoDB data to Azure Cosmos DB.

15. The status for the migration activity will be shown. The migration will only take a few seconds to complete. Select **Refresh** to reload the status to ensure it shows a **Status** of **Complete**.

16. Navigate to the **Cosmos DB Account** for the lab within the Azure Portal to verify the data was migrated, then select the **Data Explorer**. You will see the `speakers` and `sessions` collections listed within the `contentdb` database, and you will be able to explore the documents within.

## Exercise 2: Deploy the solution to Azure Kubernetes Service (60 minutes)

In this exercise, you will connect to the Azure Kubernetes Service cluster you created before the hands-on lab and deploy the Docker application to the cluster.

### Tunnel into the Azure Kubernetes Service cluster

This task will gather the information you need about your Azure Kubernetes Service cluster to connect to the cluster and execute commands to connect to the Kubernetes management dashboard from the cloud shell.

> **Note**: The following tasks should be executed in cloud shell and not the build agent VM, so disconnect from build agent VM if still connected.

1. Verify that you are connected to the correct subscription with the following command to show your default subscription:

   ```bash
   az account show
   ```

   - Ensure you are connected to the correct subscription. List your subscriptions and then set the subscription by its id with the following commands (similar to what you did in cloud shell before the lab):

   ```bash
   az account list
   az account set --subscription {id}
   ```

2. Configure kubectl to connect to the Kubernetes cluster:

   ```bash
   az aks get-credentials -a --name fabmedical-SUFFIX --resource-group fabmedical-SUFFIX
   ```

3. Test that the configuration is correct by running a simple kubectl command to produce a list of nodes:

   ```bash
   kubectl get nodes
   ```

### Deploy a service using the Azure Portal

This task will deploy the API application to the Azure Kubernetes Service cluster using the Azure Portal.

1. Define a new Namespace for our API deployment. Select the **Namespaces** blade of the fabmedical-[SUFFIX] AKS resource detail page of the Azure Portal, and on the Namespaces tab select **+ Add**.

2. In the **Add with YAML** screen, paste the following YAML and choose **Add**.

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      labels:
        name: ingress-demo
      name: ingress-demo
    ```

3. Define a Service for our API so that the application is accessible within the cluster. Select the **Services and ingresses** blade of the fabmedical-[SUFFIX] AKS resource detail page of the Azure Portal, and on the Services tab, select **+ Add**.

4. In the **Add with YAML** screen, paste the YAML below and choose **Add**.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: api
      name: api
      namespace: ingress-demo
    spec:
      ports:
        - name: api-traffic
          port: 3001
          protocol: TCP
          targetPort: 3001
      selector:
        app: api
      sessionAffinity: None
      type: ClusterIP
    ```

5. Select **Workloads** under the **Kubernetes resources** section in the left navigation.

6. From the Workloads view, with **Deployments** selected (the default), then select **+ Add**.

7. In the **Add with YAML** screen that loads, paste the following YAML and update the `[LOGINSERVER]` placeholder with the name of the ACR instance.

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
          app: api
      name: api
      namespace: ingress-demo
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: api
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
          labels:
              app: api
          name: api
        spec:
          containers:
          - image: [LOGINSERVER].azurecr.io/content-api
            env:
              - name: MONGODB_CONNECTION
                valueFrom:
                  secretKeyRef:
                    name: cosmosdb
                    key: db
            name: api
            imagePullPolicy: Always
            livenessProbe:
              httpGet:
                  path: /
                  port: 3001
              initialDelaySeconds: 30
              periodSeconds: 20
              timeoutSeconds: 10
              failureThreshold: 3
            ports:
              - containerPort: 3001
                hostPort: 3001
                protocol: TCP
            resources:
              requests:
                  cpu: 1000m
                  memory: 128Mi
            securityContext:
              privileged: false
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
    ```

8. Select **Add** to initiate the deployment. This can take a few minutes after which you will see the deployment listed.

9. Select the **api** deployment to open the failing Deployment and observe the failing pod.

10. Select the failing pod from the list of pods in the `api` deployment and select **Events**.  Observe that the pod is failing to start because the `cosmosdb` secret is not present.

11. Navigate to your resource group in the Azure Portal and find your Cosmos DB. Select the Cosmos DB resource to view details.

11. Under **Quick Start** select the **Node.js** tab and copy the **Node.js 3.0 connection string**.

12. Modify the copied connection string by adding the database `contentdb` to the URL, along with a replicaSet of `globaldb`. The resulting connection string should look like the below sample. Note that you may need to modify the endpoint URL.

    > **Note**: Username and password redacted for brevity.

    ```text
    mongodb://<USERNAME>:<PASSWORD>@fabmedical-<SUFFIX>.documents.azure.com:10255/contentdb?ssl=true&replicaSet=globaldb
    ```

    > **Note**: This lab has referenced the Cosmos DB API version 3.2. However, if your copied connection string has the endpoint suffix `.mongo.cosmos.azure.com`, feel free to use it, as that will reference the 3.6 API. The lab is compatible with both versions. Here is how the connection string shown above will appear with the 3.6 API:

    ```text
    mongodb://<USERNAME>:<PASSWORD>@fabmedical-<SUFFIX>.mongo.cosmos.azure.com:10255/contentdb?ssl=true&replicaSet=globaldb
    ```


13. You will setup a Kubernetes secret to store the connection string and configure the `content-api` application to access the secret. First, you must base64 encode the secret value. Open your Azure Cloud Shell window and use the following command to encode the connection string and then, copy the output.

    > **Note**: Double quote marks surrounding the connection string are required to successfully produce the required output.

    ```bash
    echo -n "[CONNECTION STRING VALUE]" | base64 -w 0 - | echo $(</dev/stdin)
    ```


14. Return to the AKS blade in the Azure Portal and select **Configuration** under the **Kubernetes resources** section. Select **Secrets** and choose **+ Add**.

15. In the **Add with YAML** screen, paste following YAML and replace the placeholder with the encoded connection string from your clipboard and choose **Add**. Note that YAML is position sensitive so you must ensure indentation is correct when typing or pasting.

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: cosmosdb
      namespace: ingress-demo
    type: Opaque
    data:
      db: <base64 encoded value>
    ```

 
16. Sort the Secrets list by name and you should now see your new secret displayed.

17. View the details for the **cosmosdb** secret by selected it in the list.

18. Create a new deployment manifest, `api.deployment.yml` and add the YAML content below to the file. Modify the `LOGINSERVER` placeholder for ACR.

    ```bash
    cd ~/Fabmedical
    code api.deployment.yml
    ```

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
          app: api
      name: api
      namespace: ingress-demo
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: api
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
          labels:
              app: api
          name: api
        spec:
          containers:
          - image: [LOGINSERVER].azurecr.io/content-api
            env:
              - name: MONGODB_CONNECTION
                valueFrom:
                  secretKeyRef:
                    name: cosmosdb
                    key: db
            name: api
            imagePullPolicy: Always
            livenessProbe:
              httpGet:
                  path: /
                  port: 3001
              initialDelaySeconds: 30
              periodSeconds: 20
              timeoutSeconds: 10
              failureThreshold: 3
            ports:
              - containerPort: 3001
                hostPort: 3001
                protocol: TCP
            resources:
              requests:
                  cpu: 1000m
                  memory: 128Mi
            securityContext:
              privileged: false
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30    
    ```

19. Save the file and close the editor.

20. Update the api deployment by using `kubectl` to deploy the API.

    ```bash
    kubectl delete deployment api -n ingress-demo 
    kubectl create -f api.deployment.yml
    ```

21. In the Azure Portal return to Events blade of the deployed pod in the api deployment (see Step 5). The last log should show as connected to MongoDB.

### Deploy a service using kubectl

In this task, deploy the web service using `kubectl`.

1. Open a **new** Azure Cloud Shell console.

2. Create a text file called `web.deployment.yml` in the `~/Fabmedical` folder using the Azure Cloud Shell
   Editor.

   ```bash
   cd ~/Fabmedical
   code web.deployment.yml
   ```

3. Copy and paste the following text into the editor:

    > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: web
      name: web
      namespace: ingress-demo
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: web
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: web
          name: web
        spec:
          containers:
          - image: [LOGINSERVER].azurecr.io/content-web
            env:
              - name: CONTENT_API_URL
                value: http://api:3001
            livenessProbe:
              httpGet:
                path: /
                port: 3000
              initialDelaySeconds: 30
              periodSeconds: 20
              timeoutSeconds: 10
              failureThreshold: 3
            imagePullPolicy: Always
            name: web
            ports:
              - containerPort: 3000
                hostPort: 80
                protocol: TCP
            resources:
              requests:
                cpu: 1000m
                memory: 128Mi
            securityContext:
              privileged: false
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
    ```

4. Update the `[LOGINSERVER]` entry to match the name of your ACR Login Server.

5. Select the **...** button and choose **Save**.

6. Select the **...** button again and choose **Close Editor**.

7. Create a text file called `web.service.yml` in the `~/Fabmedical` folder using the Azure Cloud Shell Editor.

    ```bash
    code web.service.yml
    ```

8. Copy and paste the following text into the editor:

    > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: web
      name: web
      namespace: ingress-demo
    spec:
      ports:
        - name: web-traffic
          port: 80
          protocol: TCP
          targetPort: 3000
      selector:
        app: web
      sessionAffinity: None
      type: LoadBalancer
    ```

9. Save changes and close the editor.

10. Execute the commands below to deploy the application described by the YAML files. You will receive a message indicating the items `kubectl` has created a web deployment and a web service.

    ```bash
    cd ~/Fabmedical
    kubectl create --save-config=true -f web.deployment.yml -f web.service.yml
    ```

11. Return to the AKS blade in the Azure Portal. From the navigation menu, under **Kubernetes resources**, select the **Services and ingresses** view. You should be able to access the website via an external endpoint.

12. In the top navigation, select the `speakers` and `sessions` links.

### Review Azure Monitor for Containers

This task will access and review the various logs and dashboards made available by Azure Monitor for Containers.

1. Select the resource group you created named `fabmedical-SUFFIX`, and then choose your `Kubernetes Service` Azure resource from the Azure Portal.

2. From the Monitoring blade, select **Insights**.

3. Review the various available dashboards and take a deeper look at the different metrics and logs available on the Cluster, Nodes, Controllers, and deployed Containers.


4. To review the Containers dashboards and see more detailed information about each container, select the **Containers** tab.

5. Filter by container name and search for the **web** containers; observe all the containers created in the Kubernetes cluster with the pod names. 

6. The CPU Usage metric is selected by default, displaying all CPU information for the selected container. To switch to another metric, open the metric dropdown list and select a different metric.

7. Upon selecting any pod, the right panel will display all the information related to the chosen metric, which would be the case when selecting any other metric. The portal will show the details on the right panel for the selected pod.

8. To display the logs for any container, simply select it and view the right panel and you will find the "View in Log Analytics" option, which will list all logs for this specific container.

9. For each log entry you can display more information by expanding the log entry to view the below details.


## Exercise 3: Scale the application and test HA (15 minutes)

At this point, you have deployed a single instance of the web and API service containers. In this exercise, you will increase the number of container instances for the web service and scale the front-end on the existing cluster.

### Increase service instances from the Azure Portal

This task will increase the number of instances for the API deployment in the AKS Azure Portal blade. While it is deploying, you will observe the changing status.

1. In the AKS blade in the Azure Portal, select **Workloads** and then select the **API** deployment.

2. Select **YAML** in the window that loads and scroll down until you find **replicas**. Change the number of replicas to **2**, then select **Review + save**. Finally, check **Confirm manifest change** and select **Save** when prompted.

    > **Note**: If the deployment completes quickly, you may not see the deployment Waiting states in the portal, as described in the following steps.

3. From the Replica Set view for the API, you will see it is now deploying and that there is one healthy instance and one pending instance.

4. From the navigation menu, select **Workloads**. Note that the api Deployment has an alert and shows a pod count 1 of 2 instances (shown as `1/2`).

    > **Note**: If you receive an error about insufficient CPU, that is OK. We will see how to deal with this in the next Task (Hint: you can use the **Insights** option in the AKS Azure Portal to review the **Node** status and view the Kubernetes event logs).

    At this point, here is a health overview of the environment:

    - One Deployment and one Replica Set are each healthy for the web service.

    - The api Deployment and Replica Set are in a warning state.

    - Two pods are healthy in the 'ingress-demo' namespace.

5. Open the Contoso Neuro Conference web application. The application should still work without errors as you navigate to Speakers and Sessions pages.

   - Navigate to the `/stats` page. You will see information about the hosting environment including:

     - **webTaskId:** The task identifier for the web service instance.

     - **taskId:** The task identifier for the API service instance.

     - **hostName:** The hostname identifier for the API service instance.

     - **pid:** The process id for the API service instance.

     - **mem:** Some memory indicators returned from the API service instance.

     - **counters:** Counters for the service itself, as returned by the API service instance.

     - **uptime:** The up time for the API service.

### Resolve failed provisioning of replicas

This task will resolve the failed API replicas. These failures occur due to the clusters' inability to meet the requested resources.

1. In the AKS blade in the Azure Portal select **Workloads** and then select the **API** deployment. Select the **YAML** navigation item.

2. In the **YAML** screen scroll down and change the following items:

    - Modify **ports** and remove the **hostPort**. Two Pods cannot map to the same host port.

      ```yaml
      ports:
        - containerPort: 3001
          protocol: TCP
      ```

    - Modify the **cpu** and set it to **100m**. CPU is divided between all Pods on a Node.

      ```yaml
      resources:
        requests:
          cpu: 100m
          memory: 128Mi
      ```

   Select **Review + save** and, when prompted, confirm the changes, and select **Save**.

3. Return to the **Workloads** main view on the AKS Azure Portal and observe that the Deployment has two healthy Pods.

### Restart containers and test HA

This task will restart containers and validate that the restart does not impact the running service.

1. Open the sample web application and navigate to the "Stats" page as shown.


2. Navigate to the api Deployment of the fabmedical-[SUFFIX] AKS cluster in the Azure Portal. Open the YAML via the **YAML** blade and increase the required replica count to `4` in the api Deployment YAML. (See steps in Exercise 4, Task 1).

3. After a few moments, you will find that the API deployment is running four replicas successfully.

4. Return to the browser tab with the web application stats page loaded. Refresh the page over and over. Observe that the api hostname periodically changes between the four api pod instances. Likewise, the task id and pid might also switch between the four api pod instances.

5. After refreshing enough times to see that the `hostName` value is changing and the service remains healthy, you can open the **Replica Sets** view for the API in the Azure Portal.

6. On this view, you can see that the hostname value shown in the web application stats page matches the pod names for the pods running.

7. Select two of the Pods at random and choose **Delete**. Select **Confirm delete**, and press **Delete** again.

8. Kubernetes will launch new Pods to meet the required replica count. Depending on your view, you may see the old instances in the Terminating state and new instances in the ContainerCreating state.

9. Return to the API Deployment and scale it back to a value of `1` replica. See Step 2 above for how to do this if you are unsure.

10. Return to the sample website's stats page in the browser and refresh while Kubernetes scales down the number of Pods. You will notice that only one API hostname shows up, even though you may still see several running pods in the API replica set view. This behavior is because Kubernetes will no longer send traffic to the pods it has selected to terminate, even though several pods are running. As a result, only one pod will show in the API Replica Set view in a few moments.

### Configure Cosmos DB Autoscale

This task will set up Autoscaling on Azure Cosmos DB.

1. In the Azure Portal, navigate to the `fabmedical-[SUFFIX]` **Azure Cosmos DB Account**.

2. Select **Data Explorer**.

3. Within **Data Explorer**, expand the `contentdb` database, then expand the `sessions` collection.

4. Under the `sessions` collection, select **Scale & Settings**.

5. On the **Scale & Settings**, select **Autoscale** for the **Throughput** setting under **Scale**.

6. Select **Save**.

7. Perform the same operation on the `speakers` collection to enable **Autoscale** on it.

### Test Cosmos DB Autoscale

This task will run a performance test script that will test the Autoscale feature of Azure Cosmos DB so you can see that it will now scale greater than 400 RU/s.

1. In the Azure Portal, navigate to the `fabmedical-[SUFFIX]` **Cosmos DB account**.

2. Select **Connection String** under **Settings**.

3. On the **Connection String** pane, copy the **HOST**, **USERNAME**, and **PRIMARY PASSWORD** values from the **Read-write Keys** selector. Save these for use later.

    >**Note**: In your Cosmos DB account, you may see that the host endpoint uses `.mongo.cosmos.azure.com`, which is for version 3.6 of Mongo DB. The endpoint shown here is `.documents.azure.com`, which is for version 3.2 of Mongo DB. You can use either endpoint for the purposes of this Task. If you are curious about the new features added to version 3.6 (that do not affect the application in this lab), consult [this](https://devblogs.microsoft.com/cosmosdb/upgrade-your-server-version-from-3-2-to-3-6-for-azure-cosmos-db-api-for-mongodb/) post.

4. Open the Azure Cloud Shell, and **SSH** to the **Build agent VM**.

5. On the **Build agent VM**, navigate to the `~/Fabmedical` directory.

    ```bash
    cd ~/Fabmedical
    ```

6. Run the following command to open the `perftest.sh` script for editing in Vi.

    ```bash
    vi perftest.sh
    ```

7. There are several variables declared at the top of the `perftest.sh` script. Modify the **host**, **username**, and **password** variables by setting their values to the corresponding Cosmos DB Connection String values that were copied previously.

    > Press `i` on your keyboard to enter insert mode, where you can alter the file.

8. Save the file and exit Vim.

    > You can do this by pressing the `Esc` key on your keyboard, followed by `:wq`.

9. Run the following command to execute the `perftest.sh` script to run a small load test against Cosmos DB. This script will consume RU's in Cosmos DB by inserting many documents into the Sessions container.

    ```bash
    bash ./perftest.sh
    ```

    > **Note:** The script will take a minute to complete executing.

10. Once the script has completed, navigate back to the **Cosmos DB account** in the Azure portal.

11. Scroll down on the **Overview** pane of the **Cosmos DB account** blade and locate the **Request Charge** graph.

    > **Note:** It may take 2 - 5 minutes for the activity on the Cosmos DB collection to appear in the activity log. Wait a few minutes and refresh the pane if the recent Request charge doesn't show up immediately.

12. Notice that the **Request charge** now shows there was activity on the **Cosmos DB account** that exceeded the 400 RU/s limit that was previously set before Autoscale was turned on.

## Exercise 4: Working with services and routing application traffic (30 minutes)

In the previous exercise, we restricted the scale properties of the service. This exercise will configure the api deployments to create pods that use dynamic port mappings to eliminate the port resource constraint during scale activities.

Kubernetes services can discover the ports assigned to each pod, allowing you to run multiple pod instances on the same agent node --- something that is not possible when you configure a specific static port (such as 3001 for the API service).

### Update an external service to support dynamic discovery with a load balancer

In this task, you will update the web service to support dynamic discovery through an Azure load balancer.

1. From AKS **Kubernetes resources** menu, select **Deployments** under **Workloads**. From the list, select the **web** deployment.

2. Select **YAML**, then select the **JSON** tab.

3. Locate the replicas node and update the required count to a value of `4`.

4. Next, scroll to the web containers spec as shown in the screenshot. Remove the hostPort entry for the web container's port mapping.

5. Select **Review + save** and then confirm the change and **Save**.

6. Check the status of the scale out by refreshing the web deployment's view. From the navigation menu, select Pods from under Workloads. Select the web pods. You should see an error like that shown in the following screenshot from this view.

Like the API deployment, the web deployment used a fixed _hostPort_, and the number of available agent nodes limited your ability to scale. However, after resolving this issue for the web service by removing the _hostPort_ setting, the web deployment cannot scale past two pods due to CPU constraints. The deployment requests more CPU than the web application needs; we will fix this constraint in the next task.

### Adjust CPU constraints to improve scale

This task will modify the CPU requirements for the web service to scale out to more instances.

1. Re-open the JSON view for the web deployment and then find the **CPU** resource requirements for the web container. Change this value to `125m`.

2. Select **Review + save**, confirm the change, and select **Save** to update the deployment.

3. From the navigation menu, select **Replica Sets** under **Workloads**. From the view's Replica Sets list select the web replica set.

4. Observe four web pods in the running state when the deployment update completes.

### Perform a rolling update

This task will edit the web application source code to add Application Insights and update the Docker image used by the deployment. Then you will perform a rolling update to demonstrate how to deploy a code change.

1. Execute this command in Azure Cloud Shell to retrieve the instrumentation key for the `content-web` Application Insights resource:

    ```bash
    az resource show -g fabmedical-[SUFFIX] -n content-web --resource-type "Microsoft.Insights/components" --query properties.InstrumentationKey -o tsv
    ```

    Copy this value. You will use it later.

    > **Note**: We must execute commands for this and later steps from an Azure Cloud Shell terminal that does not have an active SSH session with the build agent VM. This step and the following steps require the existence of the `api.deployment.yml`, `web.deployment.yml`, and `web.service.yml` files in your `~/Fabmedical` repository root; we should have generated these files in the `~/Fabmedical` repository root via previous steps in this lab from your Azure Cloud Shell terminal, and not on the build agent VM. If these files are not present at that location, please review previous lab steps up to this point and ensure the creation of these files in the correct place before proceeding.

2. From an Azure Cloud Shell terminal that does **NOT** have an active SSH session to the build agent VM update your Fabmedical repository files by pulling the latest changes from the git repository and then updating deployment YAML files.

    ```bash
    cd ~/Fabmedical
    kubectl get deployment api -n ingress-demo -o=yaml > api.deployment.yml
    kubectl get deployment web -n ingress-demo -o=yaml > web.deployment.yml
    git pull
    ```

    > **Note**: The calls to `kubectl` are necessary to fetch recent changes to the port and CPU resource configurations made in previous steps for the `web` and `api` deployments.

3. Install support for Application Insights.

    ```bash
    cd ~/Fabmedical/content-web
    npm install applicationinsights --save
    ```

    > **Note**: Make sure to include the `--save` argument. Without this, a reference to the `applicationinsights` npm package will not get added to the `package.json` file of the `content-web` nodejs project, resulting in a deployment failure in later steps.

4. Edit the `app.js` file using Vim or Visual Studio Code remote and add the following lines immediately after `express` is instantiated on line 6:

    ```javascript
    const appInsights = require("applicationinsights");
    appInsights.setup("[YOUR APPINSIGHTS KEY]");
    appInsights.start();
    ```

    ![A screenshot of the code editor showing updates in context of the app.js file](media/hol-2019-10-02_12-33-29.png "AppInsights updates in app.js")

5. Save changes and close the editor.

6. Add the following entries to the path triggers in the `content-web.yml` workflow file in the `.github/workflows` folder.

    ```yaml
    on:
      push:
        branches:
        - main
        paths:
        - 'content-web/**'
        - web.deployment.yml  # These two file
        - web.service.yml     # entries here
    ```

7. Uncomment the following task in the `content-web.yml` workflow file in the `.github/workflows` folder. Be sure to indent the YAML formatting of the task to be consistent with the formatting of the existing file.

    ```yaml 
          - name: Deploy to AKS
            uses: azure/k8s-deploy@v1
            with:
              manifests: |
                web.deployment.yml
                web.service.yml
              images: |
                ${{ env.containerRegistry }}/${{ env.imageRepository }}:${{ env.tag }}
              imagepullsecrets: |
                ingress-demo-secret
              namespace: ingress-demo
    ```

8. Add the following entries to the path triggers in the `content-api.yml` workflow file in the `.github/workflows` folder.

    ```yaml
    on:
      push:
        branches:
        - main
        paths:
        - 'content-api/**'
        - api.deployment.yml  # These two file
        - api.service.yml     # entries here
    ```

9. Uncomment the following tasks in the `content-api.yml` workflow file in the `.github/workflows` folder. Be sure to indent the YAML formatting of the task to be consistent with the formatting of the existing file.

    ```yaml
          - uses: Azure/aks-set-context@v1
            with:
              creds: '${{ secrets.AZURE_CREDENTIALS }}'
              cluster-name: '${{ env.clusterName }}'
              resource-group: '${{ env.resourceGroupName }}'
              
          - name: Deploy to AKS
            uses: azure/k8s-deploy@v1
            with:
              manifests: |
                api.deployment.yml
                api.service.yml
              images: |
                ${{ env.containerRegistry }}/${{ env.imageRepository }}:${{ env.tag }}
              imagepullsecrets: |
                ingress-demo-secret
              namespace: ingress-demo
    ```

    > **Note**: Ensure the following files from [Exercise 2](#exercise-2-deploy-the-solution-to-azure-kubernetes-service), Tasks [2](#task-2-deploy-a-service-using-the-azure-portal) and [3](#task-3-deploy-a-service-using-kubectl), are present in the git repository root.

    ```bash
    api.deployment.yml
    web.deployment.yml
    web.service.yml
    ```

10. Push these changes to your repository so that GitHub Actions CI will build and deploy a new Container image.

   ```bash
   git add .
   kubectl delete deployment web -n ingress-demo
   kubectl delete deployment api -n ingress-demo
   git commit -m "Added Application Insights"
   git push
   ```

11. Visit the `content-web` and `content-api` Actions for your GitHub Fabmedical repository and observe the images being built and deployed into the Kubernetes cluster.

12. While the pipelines rune, return the Azure Portal in the browser.

13. From the navigation menu, select **Replica Sets** under **Workloads**. From this view, you will see a new replica set for the web, which may still be in the process of deploying (as shown below) or already fully deployed.

    ![At the top of the list, a new web replica set is listed as a pending deployment in the Replica Set box.](media/2021-03-26-18-25-30.png "Pod deployment is in progress")

14. While the deployment is in progress, you can navigate to the web application and visit the stats page at `/stats`. Refresh the page as the rolling update executes. Observe that the service is running normally, and tasks continue to be load balanced.

    ![On the Stats page, the hostName is highlighted.](media/image145.png "On Stats page hostName is displayed")

### Configure Kubernetes Ingress

This task will set up a Kubernetes Ingress using an [Nginx proxy server](https://nginx.org/en/) to take advantage of path-based routing and TLS termination.

1. Run the following command from an Azure Cloud Shell terminal to add the Nginx stable Helm repository:

    ```bash
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    ```

2. Update your helm package list.

   ```bash
   helm repo update
   ```

   > **Note**: If you get a "no repositories found." error, then run the following command. This will add back the official Helm "stable" repository.
   >
   > ```bash
   > helm repo add stable https://charts.helm.sh/stable 
   > ```

3. Install the Ingress Controller resource to handle ingress requests as they come in. The Ingress Controller will receive a public IP of its own on the Azure Load Balancer and handle requests for multiple services over ports 80 and 443.

   ```bash
   helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-demo \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux
   ```

4. In the Azure Portal under **Services and ingresses**, copy the IP Address for the **External IP** for the `nginx-ingress-RANDOM-nginx-ingress` service.

    > **Note**: It could take a few minutes to refresh, alternately, you can find the IP using the following command in Azure Cloud Shell.
    >
    > ```bash
    > kubectl get svc --namespace ingress-demo
    > ```
    >
   ![A screenshot of Azure Cloud Shell showing the command output.](media/Ex4-Task5.5a.png "View the ingress controller LoadBalancer")

5. Open the [Azure Portal Resource Groups blade](https://portal.azure.com/?feature.customPortal=false#blade/HubsExtension/BrowseResourceGroups) and locate the Resource Group automatically created to host the Node Pools for AKS. It will have the naming format of `MC_fabmedical-[SUFFIX]_fabmedical-[SUFFIX]_[REGION]`.

6. Within the Azure Cloud Shell, create a script to update the public DNS name for the external ingress IP.

   ```bash
   cd ~/Fabmedical
   code update-ip.sh
   ```

   Paste the following as the contents. Be sure to replace the following placeholders in the script:

   - **[INGRESS PUBLIC IP]**: Replace this with the IP Address copied from step 4.
   - **[AKS NODEPOOL RESOURCE GROUP]**: Replace with the name of the Resource Group copied from step 5.
   - **[SUFFIX]**: Replace this with the same SUFFIX value used previously for this lab.

   ```bash
   #!/bin/bash

   # Public IP address
   IP="[INGRESS PUBLIC IP]"

   # Resource Group that contains AKS Node Pool
   KUBERNETES_NODE_RG="[AKS NODEPOOL RESOURCE GROUP]"

   # Name to associate with public IP address
   DNSNAME="fabmedical-[SUFFIX]-ingress"

   # Get the resource-id of the public ip
   PUBLICIPID=$(az network public-ip list --resource-group $KUBERNETES_NODE_RG --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

   # Update public ip address with dns name
   az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
   ```

7. Save changes and close the editor.

8. Run the update script.

   ```bash
   bash ./update-ip.sh
   ```

9. Verify the IP update by visiting the URL in your browser.

    > **Note**: It is normal to receive a 404 message at this time.

    ```text
    http://fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com/
    ```

10. Use helm to install `cert-manager`, a tool that can provision SSL certificates automatically from letsencrypt.org.

    ```bash
    kubectl create namespace cert-manager
    kubectl label namespace cert-manager cert-manager.io/disable-validation=true
    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager.yaml
    ```

11. Create a custom `ClusterIssuer` resource for the `cert-manager` service to use when handling requests for SSL certificates.

    ```bash
    cd ~/Fabmedical
    code clusterissuer.yml
    ```

    ```yaml
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        # The ACME server URL
        server: https://acme-v02.api.letsencrypt.org/directory
        # Email address used for ACME registration
        email: user@fabmedical.com
        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-prod
        # Enable HTTP01 validations
        solvers:
        - http01:
            ingress:
              class: nginx
    ```

12. Save changes and close the editor.

13. Create the issuer using `kubectl`.

    ```bash
    kubectl create --save-config=true -f clusterissuer.yml
    ```

14. Now you can create a certificate object.

    > **Note**:
    >
    > Cert-manager might have already created a certificate object for you using ingress-shim.
    >
    > To verify that the certificate was created successfully, use the `kubectl describe certificate tls-secret` command.
    >
    > If a certificate is already available, skip to step 16.

    ```bash
    cd ~/Fabmedical
    code certificate.yml
    ```

    Use the following as the contents and update the `[SUFFIX]` and `[AZURE-REGION]` to match your ingress DNS name.

    ```yaml
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: tls-secret
    spec:
      secretName: tls-secret
      dnsNames:
        - fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
      issuerRef:
        name: letsencrypt-prod
        kind: ClusterIssuer
    ```

15. Save changes and close the editor.

16. Create the certificate using `kubectl`.

    ```bash
    kubectl create --save-config=true -f certificate.yml
    ```

    > **Note**: To check the status of the certificate issuance, use the `kubectl describe certificate tls-secret` command and look for an _Events_ output similar to the following:
    >
    > ```text
    > Type    Reason         Age   From          Message
    > ----    ------         ----  ----          -------
    > Normal  Generated           38s   cert-manager  Generated new private key
    > Normal  GenerateSelfSigned  38s   cert-manager  Generated temporary self signed certificate
    > Normal  OrderCreated        38s   cert-manager  Created Order resource "tls-secret-3254248695"
    > Normal  OrderComplete       12s   cert-manager  Order "tls-secret-3254248695" completed successfully
    > Normal  CertIssued          12s   cert-manager  Certificate issued successfully
    > ```

    It can take between 5 and 30 minutes before the tls-secret becomes available. This is due to the delay involved with provisioning a TLS cert from letsencrypt.

17. Now you can create an ingress resource for the content applications.

    ```bash
    cd ~/Fabmedical
    code content.ingress.yml
    ```

    Use the following as the contents and update the `[SUFFIX]` and `[AZURE-REGION]` to match your ingress DNS name:

    ```yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: content-ingress
      namespace: ingress-demo
      annotations:
        kubernetes.io/ingress.class: nginx
        nginx.ingress.kubernetes.io/rewrite-target: /$1
        nginx.ingress.kubernetes.io/use-regex: "true"
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
        cert-manager.io/cluster-issuer: letsencrypt-prod
    spec:
      tls:
      - hosts:
          - fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
        secretName: tls-secret
      rules:
      - host: fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
        http:
          paths:
          - path: /(.*)
            backend:
              serviceName: web
              servicePort: 80
          - path: /content-api/(.*)
            backend:
              serviceName: api
              servicePort: 3001
    ```

18. Save changes and close the editor.

19. Create the ingress using `kubectl`.

    ```bash
    kubectl create --save-config=true -f content.ingress.yml
    ```

20. Refresh the ingress endpoint in your browser. You should be able to visit the speakers and sessions pages and see all the content.

21. Visit the API directly, by navigating to `/content-api/sessions` at the ingress endpoint.

22. Test TLS termination by visiting both services again using `https`.

    > **Note**: It can take between 5 and 30 minutes before the SSL site becomes available. This is due to the delay involved with provisioning a TLS cert from letsencrypt.

## After the Hands-on Lab

1. Delete all Azure resources created in support of this Hands-on lab.

1. End of **Workshop Day**

1. Continue in the **Mentoria Arquiteto Cloud**.










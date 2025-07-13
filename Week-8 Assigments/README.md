----------------------
# **Week- 8 Assignment**
----------------------


# *Configure dashboard and queries for work items.*

###  Step 1: Navigate to Your Azure DevOps Project

1. Go to [https://dev.azure.com/](https://dev.azure.com/).
2. Sign in with your credentials.
3. Click on your **organization** → then select your **project**.

---

###  Step 2: Create a Work Item Query

1. From the **left sidebar**, select **Boards → Queries**.
2. Click **“New Query”** on the top.
3. Define your query filters:

   * Example: `Work Item Type = Bug`, `State = Active`, `Assigned To = @Me`.
4. Use **AND/OR** operators to build complex filters.
5. Click **“Run Query”** to see the results.
6. Click **Save Query** → enter a **Name** (e.g., “My Active Bugs”).

   * Choose where to save: **My Queries** or a **Shared folder** (visible to team).
   * Choose folder: e.g., `Shared Queries / Sprint 1`.

---

###  Step 3: Create or Edit a Dashboard

1. Go to **Dashboards** from the left panel.
2. If you want to:

   * **Use an existing dashboard** → click its name.
   * **Create a new one** → click **“New Dashboard”** at the top right.

     * Name your dashboard (e.g., “Sprint 1 Overview”).
     * Choose team and visibility (Private/Team).
     * Click **Create**.

---

###  Step 4: Add Widgets to Display Work Item Queries

1. On your dashboard, click **“Edit”** in the top right.
2. Click **“Add a widget”**.
3. Choose a relevant widget like:

   * **Query Results** – displays a table of work items.
   * **Work Item Chart** – visualizes data from queries (e.g., Pie, Bar).
   * **Sprint Burndown** or **Sprint Capacity** – useful for Scrum projects.
4. Click **Add** on your chosen widget(s).
5. Configure the widget:

   * For **Query Results**:

     * Set **Query** = your saved query (e.g., “My Active Bugs”).
     * Select columns to show (e.g., Title, State, Assigned To).
   * For **Work Item Chart**:

     * Select query.
     * Choose chart type (e.g., Pie chart of “Bug State”).
6. Click **Done Editing** once all widgets are placed and configured.

---

###  Step 5: Share and Refresh the Dashboard

* Dashboards are **auto-saved**.
* To share:

  * Click the **gear icon ⚙️** in top-right of dashboard → **Set permissions**.
* To refresh:

  * Click the **refresh icon** on the dashboard or individual widgets.

---



-------
# *Use Pipeline Variables While Configuring Pipelines in Azure DevOps*

---

## 1. Understanding Pipeline Variables

### Types of Variables:

| Type                  | Defined Where              | Scope              |
| --------------------- | -------------------------- | ------------------ |
| Pipeline variables    | Inline in YAML or UI       | Pipeline-wide      |
| Variable groups       | Library > Variable Groups  | Multiple pipelines |
| Output variables      | In steps, passed to others | Step/Job level     |
| Environment variables | Automatically provided     | System-scope       |

---

## 2. Create and Use Variables in YAML Pipeline

### Step 1: Define Variables Inline in YAML

```yaml
variables:
  appName: 'sample-app'
  environment: 'development'
  buildConfig: 'Release'
```

These are available throughout the pipeline.

---

### Step 2: Use Variables in Steps

```yaml
steps:
- script: echo "Building $(appName) in $(buildConfig) mode for $(environment) environment"
```

---

### Step 3: Use Variables with Conditions

```yaml
steps:
- script: echo "This step runs only in development"
  condition: eq(variables['environment'], 'development')
```

---

## 3. Use Variables from Azure DevOps UI (Runtime)

### Step 1: Open Pipeline in Azure DevOps

* Go to `Pipelines` → Select your pipeline → Click `Run pipeline`.

### Step 2: Add Variables at Runtime

* Click **Variables**.
* Add name and value, such as:

  * `appName = user-service`
  * `environment = test`
* Click **Run**.

These override any YAML-defined variables.

---

## 4. Use Variable Groups from Library

### Step 1: Create a Variable Group

* Go to `Pipelines` → `Library`.
* Click `+ Variable Group`.
* Enter group name (e.g., `DevSettings`).
* Add variables like:

  * `connectionString = db-dev-conn`
  * `apiKey = dev-api-key`
* Save the group.

### Step 2: Reference in YAML

```yaml
variables:
- group: DevSettings
```

Then you can use the variables as:

```yaml
steps:
- script: echo "Using connection: $(connectionString)"
```

---

## 5. Use Secret Variables

### Step 1: Mark Variable as Secret

* While creating variables in Library or UI, check **Keep this value secret**.

### Step 2: Use in YAML

```yaml
variables:
- group: DevSecrets

steps:
- script: echo "Secret is not printed"
  env:
    MY_SECRET: $(mySecretVariable)
```

Secret values are not shown in logs.

---

## 6. Define Output Variables from a Step

### Step 1: Set Variable

```yaml
steps:
- script: echo "##vso[task.setvariable variable=myVar;isOutput=true]myValue"
  name: SetStep
```

### Step 2: Use Output Variable in Another Step or Job

```yaml
- script: echo "The value is $(SetStep.myVar)"
```

---

## 7. Set Variables per Stage or Job

```yaml
stages:
- stage: Build
  variables:
    stageVar: 'build-stage'
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Stage variable: $(stageVar)"
```

---

## 8. Use Environment Variables (Predefined)

Azure DevOps provides built-in variables:

* `Build.BuildId`
* `Build.Repository.Name`
* `Agent.OS`
* `System.TeamProject`

Example:

```yaml
steps:
- script: echo "Project: $(System.TeamProject)"
```


---

# *Configure Azure Pipelines with Variable Groups, Task Groups, and Stage Scopes*

---

## 1. Create Variable Groups with Scoped Variables

### Step 1: Go to Azure DevOps Library

1. Navigate to: `Pipelines → Library`
2. Click **+ Variable Group**
3. Name the group (e.g., `AppSettings`)
4. Add variables:

   * `appName = myapp`
   * `connectionString = dev-db`
   * `env = development`
5. (Optional) Mark secrets using the checkbox
6. Click **Save**

Repeat for other environments if needed (e.g., `ProdSettings`, `TestSettings`).

---

## 2. Use Variable Groups in YAML Pipeline with Stage Scope

You can scope variable groups to specific stages by referencing them inside each stage:

```yaml
trigger:
- main

stages:
- stage: Dev
  variables:
    - group: AppSettings
  jobs:
  - job: DeployDev
    steps:
    - script: echo "Deploying $(appName) to $(env) using $(connectionString)"

- stage: Prod
  variables:
    - group: ProdSettings
  jobs:
  - job: DeployProd
    steps:
    - script: echo "Deploying $(appName) to $(env) using $(connectionString)"
```

> Variables in the variable group are **only available within the stage where the group is defined**.

---

## 3. Create a Task Group (Reusable Set of Steps)

Task Groups are available only via the **Classic UI** editor, but you can simulate them in YAML using templates.

### Step 1: Create a Task Group (Classic)

1. Go to `Pipelines → Pipelines (classic)`
2. Open or create a new build/release pipeline
3. Add a few tasks (e.g., restore, build, publish)
4. Select multiple tasks → right-click → **Create Task Group**
5. Name the Task Group (e.g., `BuildAppGroup`)
6. Define parameters (like `buildConfiguration`)
7. Save the task group

> You can reuse this task group in multiple classic pipelines.

---

## 4. Simulate Task Groups in YAML (Using Templates)

YAML does not support classic task groups directly, but you can create **template files** for reusable tasks.

### Step 1: Create a Template File (`build-task.yml`)

```yaml
parameters:
  buildConfiguration: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration ${{ parameters.buildConfiguration }}'
```

### Step 2: Use Template in Main YAML Pipeline

```yaml
stages:
- stage: Build
  variables:
    - group: AppSettings
  jobs:
  - job: BuildJob
    steps:
    - template: build-task.yml
      parameters:
        buildConfiguration: 'Release'
```

---

## 5. Set Variable Scope by Stage, Job, and Step

### A. Scope to Stage

```yaml
stages:
- stage: Test
  variables:
    stageVar: 'test-stage'
  jobs:
  - job: TestJob
    steps:
    - script: echo "Stage scoped var: $(stageVar)"
```

### B. Scope to Job

```yaml
jobs:
- job: UnitTests
  variables:
    jobVar: 'unit-job'
  steps:
  - script: echo "Job scoped var: $(jobVar)"
```

### C. Scope to Step (set dynamically)

```yaml
steps:
- script: echo "##vso[task.setvariable variable=stepVar]step-value"
- script: echo "Value: $(stepVar)"
```

---

## 6. Using Secret Variables

To keep variables like `connectionString` or `apiKey` secure:

1. In the Variable Group, mark them as **secret**
2. Use them like normal variables in YAML:

   ```yaml
   steps:
   - script: echo "Connecting with $(connectionString)"
     env:
       conn: $(connectionString)
   ```

> Secret variables are masked in logs automatically.

---







---

# *How to Create a Service Connection in Azure DevOps*

---

## 1. Prerequisites

* You must have **Project Administrator** permissions in the Azure DevOps project.
* You must have access to the **Azure subscription** you want to connect to.
* The **Azure DevOps organization** should be linked to an Azure Active Directory (AAD).

---

## 2. Steps to Create Azure Service Connection (Service Principal)

### Step 1: Go to Project Settings

1. Open your Azure DevOps project.
2. In the bottom-left corner, click on **Project Settings**.

---

### Step 2: Open Service Connections

1. Under **Pipelines**, click **Service connections**.
2. Click the **+ New service connection** button (top right).

---

### Step 3: Select Service Connection Type

1. In the popup, select **Azure Resource Manager**.
2. Choose **Service principal (automatic)** for most cases (recommended).

   * Use **Service principal (manual)** if you want to use custom credentials.
3. Click **Next**.

---

### Step 4: Configure the Azure Connection

1. **Scope level**: Select `Subscription` or `Management Group` depending on your need.
2. **Azure Subscription**: Select from the list (requires login).
3. **Resource Group** (optional): Select one if you want to limit scope.
4. **Service Connection Name**: Provide a name (e.g., `AzureConnection`).
5. **Security**: Optionally, restrict access to specific users or pipelines.
6. Click **Verify and Save**.

---

## 3. Use the Service Connection in YAML Pipelines

After creation, you can refer to it using the name you provided:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'AzureConnection'  # Service connection name
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az group list
```

Or for resource deployment:

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    azureResourceManagerConnection: 'AzureConnection'
    location: 'East US'
    templateLocation: 'Linked artifact'
    csmFile: 'azuredeploy.json'
    deploymentMode: 'Incremental'
```

---

## 4. Other Common Service Connection Types

You can repeat the same steps for other services:

* **Docker Registry**: Used for pushing/pulling images.
* **GitHub**: To access GitHub repositories.
* **Kubernetes**: For deploying containers to AKS or other clusters.
* **Generic**: For custom HTTP endpoints.

---

## 5. Managing Service Connection Permissions

After creating the connection:

1. Go to **Project Settings → Service connections**.
2. Click the connection name.
3. Click the **3 dots (···)** > **Security**.
4. Configure who can use, edit, or manage the connection.

---




---

# *How to Create a Linux/Windows Self-hosted Agent for Azure DevOps*

---

## 1. Prerequisites

* You must be a **Project Administrator** or **Organization Owner** in Azure DevOps.
* A machine (physical, VM, cloud) with:

  * Internet access
  * A supported OS (Linux or Windows)
  * .NET Core installed (required for agent)
  * Sudo/admin permissions

---

## 2. Create a Personal Access Token (PAT) *(only needed in manual config)*

If you're using the **manual configuration**, you'll need a **PAT**:

1. Go to `https://dev.azure.com/your-org-name/`
2. Click on your profile (top right) → **Security**.
3. Click **New Token**:

   * Name: `agent-pat`
   * Organization: Your organization
   * Expiration: 30/60/90 days
   * Scopes:

     * **Agent Pools (Read & Manage)**
     * **Deployment Group (Read & Manage)** (if needed)
4. Click **Create** → Copy the token

---

## 3. Add a New Agent in Azure DevOps

1. Go to `https://dev.azure.com/{your_organization}`
2. Navigate to your project → **Project Settings** (bottom left)
3. Under **Pipelines**, click **Agent pools**
4. Select an existing pool (e.g., `Default`) or click **Add pool** to create a new one
5. Click **New agent**
6. Choose:

   * **OS**: Windows, Linux, macOS
   * **Architecture**: x64 (usually)
7. Follow the instructions provided there (same as below)

---

## 4. Set Up a Self-Hosted Agent on Windows

### Step 1: Download and Extract Agent

Open PowerShell or CMD:

```powershell
mkdir C:\agent
cd C:\agent
Invoke-WebRequest -Uri https://vstsagentpackage.azureedge.net/agent/3.236.1/vsts-agent-win-x64-3.236.1.zip -OutFile agent.zip
Expand-Archive -Path agent.zip -DestinationPath .
```

### Step 2: Configure the Agent

```powershell
.\config.cmd
```

Provide:

* Azure DevOps URL: `https://dev.azure.com/your-org`
* Authentication type: **PAT**
* PAT: Paste your token
* Agent Pool: `Default` or custom pool
* Agent Name: `win-agent-01`
* Work Folder: Default

### Step 3: Install the Agent as a Windows Service

```powershell
.\svc install
.\svc start
```

---

## 5. Set Up a Self-Hosted Agent on Linux

### Step 1: Download and Extract Agent

```bash
mkdir myagent && cd myagent
wget https://vstsagentpackage.azureedge.net/agent/3.236.1/vsts-agent-linux-x64-3.236.1.tar.gz
tar zxvf vsts-agent-linux-x64-3.236.1.tar.gz
```

### Step 2: Configure the Agent

```bash
./config.sh
```

Provide:

* Azure DevOps URL: `https://dev.azure.com/your-org`
* Authentication type: **PAT**
* PAT: Paste your token
* Agent Pool: `Default` or custom pool
* Agent Name: `linux-agent-01`
* Work Folder: Default

### Step 3: Install and Start Agent as a Service

Install dependencies first:

```bash
sudo ./bin/installdependencies.sh
```

Then install the service:

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## 6. Verify Agent Status

1. Go to **Project Settings → Agent Pools**
2. Click on the pool where you registered the agent
3. You should see your **agent listed and marked as "online"**

---




---

# *Apply Pre and Post-Deployment Approvers in Azure DevOps Release Pipeline*

> This applies to **Classic Release Pipelines** (not YAML pipelines). If you're using YAML, approvals are handled via **environments and checks**.

---

## 1. Navigate to Release Pipelines

1. Open your Azure DevOps organization and project.
2. Go to **Pipelines → Releases**.
3. Open an existing release pipeline or click **“New pipeline”** to create one.
4. Add one or more **stages** (environments), e.g., `Dev`, `Test`, `Prod`.

---

## 2. Configure Pre-Deployment Approvers

### Step 1: Go to Stage Settings

1. Hover over the stage (e.g., **Dev**) in the pipeline visual designer.
2. Click the **“person + lock”** icon (appears on stage border) or click the **stage name** to open **stage settings**.

### Step 2: Add Pre-Deployment Approvers

1. Under **Pre-deployment conditions**, click **“Pre-deployment approvals”**.
2. Click **+ Add user(s)**.
3. Select one or more users or groups who will approve the deployment.
4. Optional:

   * Enable **Requestor should not approve** to prevent self-approval.
   * Set a **timeout** (e.g., 2 days) after which the approval is auto-rejected.
5. Click **Save**.

---

## 3. Configure Post-Deployment Approvers

### Step 1: Go to Stage Settings

1. While still in the stage settings pane, scroll down to **Post-deployment conditions**.
2. Click **“Post-deployment approvals”**.

### Step 2: Add Post-Deployment Approvers

1. Click **+ Add user(s)**.
2. Add required approvers (can be different from pre-deployment approvers).
3. Set timeout and approval rules as needed.
4. Click **Save**.

---

## 4. Save and Create a Release

1. Click **Save** at the top-right of the release pipeline editor.
2. Click **Create Release**.
3. Choose artifacts, select stages, and click **Create**.

---

## 5. Approval in Action

When a deployment reaches a stage with configured **pre-approvers**:

* Azure DevOps **pauses the pipeline**.
* The designated user(s) receive an **email and notification**.
* They can **approve or reject** from:

  * The Azure DevOps portal (under Releases)
  * Email notification link

After the stage runs successfully, if **post-approvers** are configured:

* The system **waits again** for approval before moving to the next stage.

---






---

# *CI/CD Pipeline: Build → Push to ACR → Deploy to AKS (Azure DevOps YAML)*

---

## Prerequisites

* An Azure DevOps project
* Azure resources:

  * Azure Kubernetes Service (AKS)
  * Azure Container Registry (ACR)
* A working `Dockerfile` and Kubernetes manifest files (e.g., `deployment.yaml`, `service.yaml`)
* Azure DevOps **service connections**:

  * One for **Azure Resource Manager** (for AKS access)
  * One for **Docker Registry (ACR)**

---

## 1. Create Azure DevOps Service Connections

### A. Docker Registry (ACR)

1. Go to **Project Settings → Service Connections**
2. Click **New Service Connection → Docker Registry**
3. Select **Azure Container Registry**
4. Fill in:

   * Docker Registry: Select your ACR
   * Service Connection Name: `acr-connection`
5. Save

### B. Azure Resource Manager (for AKS)

1. **New Service Connection → Azure Resource Manager**
2. Choose **Service Principal (automatic)**
3. Choose your subscription and AKS resource group
4. Name it `azure-aks-connection`
5. Save

---

## 2. Project Folder Structure

```
project-root/
│
├── Dockerfile
├── app/                 # Source code
├── manifests/
│   ├── deployment.yaml
│   └── service.yaml
└── azure-pipelines.yml  # Pipeline file
```

---

## 3. YAML Pipeline: `azure-pipelines.yml`

```yaml
trigger:
- main

variables:
  imageName: myapp
  acrName: myacrname.azurecr.io
  aksNamespace: default

stages:

- stage: BuildAndPush
  displayName: Build and Push Docker Image
  jobs:
  - job: DockerBuildPush
    displayName: Build and Push to ACR
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      displayName: Build Docker Image
      inputs:
        containerRegistry: 'acr-connection'
        repository: '$(imageName)'
        command: 'buildAndPush'
        Dockerfile: '**/Dockerfile'
        tags: |
          $(Build.BuildId)

- stage: DeployToAKS
  displayName: Deploy to AKS
  dependsOn: BuildAndPush
  jobs:
  - job: AKSDeploy
    displayName: Deploy to AKS
    pool:
      vmImage: 'ubuntu-latest'
    steps:

    - task: Kubernetes@1
      displayName: Set AKS Context
      inputs:
        connectionType: 'Azure Resource Manager'
        azureSubscription: 'azure-aks-connection'
        azureResourceGroup: '<YourResourceGroup>'
        kubernetesCluster: '<YourAKSClusterName>'
        command: 'login'

    - task: KubernetesManifest@0
      displayName: Deploy Kubernetes Manifests
      inputs:
        action: deploy
        kubernetesServiceConnection: 'azure-aks-connection'
        namespace: '$(aksNamespace)'
        manifests: |
          manifests/deployment.yaml
          manifests/service.yaml
        containers: |
          $(acrName)/$(imageName):$(Build.BuildId)
```

---

## 4. Kubernetes Deployment Manifest Example

`manifests/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myacrname.azurecr.io/myapp:latest
        ports:
        - containerPort: 80
```

> Replace `latest` with `$(Build.BuildId)` to pin versioned images.

---

## 5. Grant AKS Pull Access to ACR

If ACR and AKS are in the same subscription, run this command:

```bash
az aks update -n <AKS_NAME> -g <RESOURCE_GROUP> --attach-acr <ACR_NAME>
```

---

## 6. Commit and Run Pipeline

1. Push `azure-pipelines.yml` to your `main` branch
2. Go to **Pipelines → New Pipeline → Azure Repos Git → Select your repo**
3. Choose **Existing YAML file**
4. Run the pipeline

---

## Output

* A Docker image is built and pushed to ACR
* The image is deployed to AKS using your Kubernetes manifests

---



---

# *CI/CD Pipeline: Build Docker Image → Push to ACR → Deploy to ACI*

---

## 1. Prerequisites

You must have:

* An Azure DevOps Project
* An Azure Container Registry (ACR)
* A Resource Group where ACI will be deployed
* A **Dockerfile** and source code in your repository

### Azure DevOps Service Connections

Create **two service connections** under **Project Settings → Service Connections**:

1. **Docker Registry (ACR)**

   * Type: Docker Registry
   * Registry Type: Azure Container Registry
   * Name: `acr-connection`

2. **Azure Resource Manager**

   * Type: Azure Resource Manager
   * Scope: Subscription
   * Name: `azure-rm-connection`

---

## 2. Folder Structure

```
/project-root
│
├── Dockerfile
├── app/                 # Your application code
└── azure-pipelines.yml  # Pipeline file
```

---

## 3. YAML Pipeline: `azure-pipelines.yml`

```yaml
trigger:
- main

variables:
  imageName: myapp
  acrLoginServer: myacr.azurecr.io
  acrConnection: acr-connection
  azureSubscription: azure-rm-connection
  resourceGroup: my-resource-group
  containerGroup: my-container-group
  location: eastus

stages:

# Stage 1: Build and Push Docker Image to ACR
- stage: BuildAndPush
  displayName: Build and Push Docker Image
  jobs:
  - job: DockerJob
    displayName: Build and Push
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      displayName: Build and Push Image to ACR
      inputs:
        command: buildAndPush
        containerRegistry: $(acrConnection)
        repository: $(imageName)
        Dockerfile: '**/Dockerfile'
        tags: |
          $(Build.BuildId)

# Stage 2: Deploy to ACI
- stage: DeployToACI
  displayName: Deploy to Azure Container Instances
  dependsOn: BuildAndPush
  jobs:
  - job: DeployJob
    displayName: Deploy to ACI
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: AzureCLI@2
      displayName: Deploy to ACI
      inputs:
        azureSubscription: $(azureSubscription)
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          az group create --name $(resourceGroup) --location $(location)
          az container create \
            --resource-group $(resourceGroup) \
            --name $(containerGroup) \
            --image $(acrLoginServer)/$(imageName):$(Build.BuildId) \
            --registry-login-server $(acrLoginServer) \
            --registry-username $(ACR_USERNAME) \
            --registry-password $(ACR_PASSWORD) \
            --dns-name-label $(containerGroup)-dns \
            --ports 80
        env:
          ACR_USERNAME: $(acr-username)
          ACR_PASSWORD: $(acr-password)
```

---

## 4. Configure Secret Variables (ACR Credentials)

In **Azure DevOps → Pipelines → Library**:

1. Click **+ Variable Group** → name it `acr-secrets`
2. Add:

   * `acr-username`: your ACR admin username
   * `acr-password`: your ACR admin password
3. Mark both as **secret**
4. In your pipeline, reference this group:

```yaml
variables:
- group: acr-secrets
```

Add this **at the top** of the YAML file.

---

## 5. Result

Once you commit and run the pipeline:

* Azure DevOps builds and pushes the image to ACR
* ACI pulls and runs the container using the latest image tag (`Build.BuildId`)
* You can access the container using the generated DNS:
  `http://<containerGroup>-dns.<region>.azurecontainer.io`

---

## Optional Enhancements

* Replace ACI commands with a **Bicep/ARM template** for better repeatability
* Add **cleanup step** to delete old container groups
* Set up **approval gates** before production deployment

---





---

# *CI/CD Pipeline: Build and Deploy .NET App to Azure App Service*

---

##  Prerequisites

Before you begin:

* You have a **.NET Core or .NET 6/7/8** application with a `*.csproj` and `Program.cs`
* Azure App Service is already created in your Azure portal
* You have a Git repo with your code in Azure Repos or GitHub
* You created a **Service Connection** in Azure DevOps:

  * Type: **Azure Resource Manager**
  * Name: `azure-service-connection`

---

##  Folder Structure Example

```
/my-dotnet-app
│
├── Controllers/
├── Views/
├── Program.cs
├── my-dotnet-app.csproj
└── azure-pipelines.yml
```

---

##  Step-by-Step Pipeline (`azure-pipelines.yml`)

```yaml
trigger:
- main

variables:
  buildConfiguration: 'Release'
  azureSubscription: 'azure-service-connection'     # Name of service connection
  appName: 'your-app-service-name'                  # Azure App Service name
  projectRoot: '.'                                  # Path to .csproj file

pool:
  vmImage: 'windows-latest'

stages:

- stage: Build
  displayName: Build Stage
  jobs:
  - job: BuildJob
    displayName: Build .NET Application
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: '8.0.x'

    - script: dotnet build $(projectRoot) --configuration $(buildConfiguration)
      displayName: 'Build Project'

    - task: DotNetCoreCLI@2
      displayName: 'Publish .NET App'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: true

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifact'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: Deploy
  displayName: Deploy Stage
  dependsOn: Build
  jobs:
  - deployment: DeployApp
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            displayName: 'Deploy to Azure App Service'
            inputs:
              azureSubscription: $(azureSubscription)
              appType: 'webApp'
              appName: $(appName)
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

##  Setup in Azure DevOps

### Step 1: Create Service Connection

1. Go to **Project Settings → Service Connections**
2. Click **New Service Connection**
3. Choose **Azure Resource Manager**
4. Select **Service Principal (automatic)** and your subscription
5. Choose the correct App Service resource group
6. Name it `azure-service-connection`

### Step 2: Create Pipeline

1. Go to **Pipelines → New Pipeline**
2. Choose your repo → select **YAML**
3. Use existing file `azure-pipelines.yml` or create new
4. Run the pipeline

---




---

# *CI/CD Pipeline: React App → Build → Deploy to Azure VM*

---

##  Prerequisites

1. A **React app** created with `create-react-app` or similar
2. An **Azure Virtual Machine** (Linux or Windows) running and accessible via **SSH**
3. Azure DevOps project with the React app code in a repository
4. SSH access (username, IP, port, private key)
5. A web server installed on the VM (like **Nginx**, **Apache**, or **Node.js server**)

---

##  Folder Structure Example

```
/my-react-app
│
├── public/
├── src/
├── package.json
├── .env
└── azure-pipelines.yml
```

---

##  Step 1: Configure the Azure VM

On your Azure VM (Linux):

```bash
sudo apt update
sudo apt install nginx -y
sudo ufw allow 'Nginx Full'
```

Ensure you have an SSH key pair. You’ll use the **private key** in the DevOps pipeline.

---

##  Step 2: Set Up SSH in Azure DevOps

### Create Pipeline Variables or Variable Group:

1. Go to **Pipelines → Library** → **+ Variable Group**
2. Add variables:

   * `vm_host`: e.g., `your-vm-ip` or DNS
   * `vm_port`: `22`
   * `vm_username`: SSH user, e.g., `azureuser`
   * `vm_target_path`: e.g., `/var/www/html`
   * `ssh_private_key`: Your **SSH private key** (mark as secret)

---

##  Step 3: Azure Pipeline YAML (`azure-pipelines.yml`)

```yaml
trigger:
- main

variables:
- group: ReactVMDeploy

pool:
  vmImage: 'ubuntu-latest'

stages:

- stage: Build
  displayName: Build React App
  jobs:
  - job: BuildJob
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
      displayName: 'Install Node.js'

    - script: |
        npm install
        npm run build
      displayName: 'Build React App'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: 'build'
        ArtifactName: 'react-build'
        publishLocation: 'Container'

- stage: Deploy
  displayName: Deploy to Azure VM
  dependsOn: Build
  jobs:
  - deployment: DeployToVM
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:

          - download: current
            artifact: react-build

          - task: CopyFilesOverSSH@0
            inputs:
              sshEndpoint: 'vm-ssh-connection'   # Define this service connection manually
              sourceFolder: '$(Pipeline.Workspace)/react-build'
              contents: '**'
              targetFolder: '$(vm_target_path)'
              cleanTargetFolder: true
              overwrite: true

          - task: SSH@0
            inputs:
              sshEndpoint: 'vm-ssh-connection'
              runOptions: inline
              inline: |
                sudo systemctl restart nginx
```

---

##  Step 4: Create SSH Service Connection

1. Go to **Project Settings → Service Connections**
2. Click **New service connection → SSH**
3. Name: `vm-ssh-connection`
4. Provide:

   * Host: `$(vm_host)`
   * Port: `$(vm_port)`
   * Username: `$(vm_username)`
   * Paste your **SSH private key**
5. Save

---

##  Result

* The React app is built using `npm run build`
* The contents of the `build/` folder are securely copied to the Azure VM (e.g., `/var/www/html`)
* Nginx (or your server) serves the app at the public IP or DNS

Access it via:

```
http://<vm_public_ip>/
```

---

##  Optional Enhancements

* Use **nginx.conf** for custom routing (`index.html` fallback)
* Setup **domain + SSL (Let's Encrypt)** on VM
* Add **approval gate** before deploy stage
* Configure **environment variables** from Azure Key Vault

---


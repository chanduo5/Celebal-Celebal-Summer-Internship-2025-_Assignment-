--------------------
# *Final project*
--------------------


## **Goal**

Automatically deploy your containerized application to **Azure Container Instance (ACI)** whenever you push changes to your code repository, using **Azure DevOps pipelines**.

---


### STEP 1: Create Required Azure Resources

#### 1.1 Create a Resource Group

1. Go to [Azure Portal]
2. Search for **Resource groups**
3. Click **+ Create**
4. Enter:

   * Name: `aci-devops-rg`
   * Region: Choose one (e.g., `East US`)
5. Click **Review + Create → Create**


![screenshorts](screenshorts/Screenshot%202025-07-08%20131538.png)

---


#### 1.2 Create Azure Container Registry (ACR)

1. Search for **Container registries**
2. Click **+ Create**
3. Enter:

   * Registry name: `acichanduacr`
   * Resource group: `aci-devops-rg`
   * SKU: **Basic**
   * Admin user: **Enable**
4. Click **Review + Create → Create**



![screenshorts](screenshorts/Screenshot%202025-07-08%20131431.png)
![screenshorts](screenshorts/Screenshot%202025-07-08%20131519.png)



---

###  STEP 2: Prepare Your Application Code


#### Example: Flask app

**`app.py`**

```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "Hello from Azure Container Instance!"
```

**`requirements.txt`**

```
flask
```

**`Dockerfile`**

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```


---

###  STEP 3: Create Service Connection in Azure DevOps

1. Go to your [Azure DevOps project](https://dev.azure.com)
2. Go to **Project settings > Service connections**
3. Click **+ New service connection**
4. Choose **Azure Resource Manager**
5. Choose **Service Principal (automatic)**
6. Select your **Azure Subscription**
7. Scope Level: Select **Subscription**
8. Name: `AzureConnection`
9. Click **Save**

   
![screenshorts](screenshorts/Screenshot%202025-07-08%20140141.png)
![screenshorts](screenshorts/Screenshot%202025-07-08%20141207.png)

---

###  STEP 4: Create CI Pipeline (Build + Push to ACR)

#### 4.1 Go to Pipelines > Pipelines

1. Click **New Pipeline**
2. Select **Azure Repos Git** (or GitHub if used)
3. Select your repository
4. Choose **Starter pipeline**
5. Replace content with:

```yaml
trigger:
- main

variables:
  imageName: 'flaskapp'
  acrName: 'acichanduacr'                  # Your ACR name
  acrLoginServer: 'acichanduacr.azurecr.io'
  resourceGroup: 'aci-devops-rg'
  containerName: 'flask-container'
  location: 'eastus'

stages:

# ---------------------
# CI Stage: Build + Push to ACR
# ---------------------
- stage: BuildAndPush
  displayName: Build and Push Docker Image
  jobs:
  - job: BuildAndPush
    displayName: Build & Push Image
    pool:
      vmImage: 'ubuntu-latest'
    steps:

    - task: Docker@2
      displayName: Build and Push to ACR
      inputs:
        containerRegistry: 'ACRConnection'         # Docker Registry service connection
        repository: '$(imageName)'
        command: 'buildAndPush'
        Dockerfile: '**/Dockerfile'
        tags: |
          latest

# ---------------------
# CD Stage: Deploy to Azure Container Instance
# ---------------------
- stage: Deploy
  displayName: Deploy to ACI
  dependsOn: BuildAndPush
  jobs:
  - job: DeployACI
    displayName: Deploy to Azure Container Instance
    pool:
      vmImage: 'ubuntu-latest'
    steps:

    - task: AzureCLI@2
      displayName: Deploy ACI with latest image
      inputs:
        azureSubscription: 'AzureConnection'        # Azure Resource Manager service connection
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az container create \
            --resource-group $(resourceGroup) \
            --name $(containerName) \
            --image $(acrLoginServer)/$(imageName):latest \
            --registry-login-server $(acrLoginServer) \
            --registry-username $(ACR_USERNAME) \
            --registry-password $(ACR_PASSWORD) \
            --dns-name-label flaskaci$(Build.BuildId) \
            --ports 80 \
            --location $(location)

```
-----------------------------------------

![screenshorts](screenshorts/Screenshot%202025-07-08%20141327.png)

---------------------------------------------
6. Click **Save and Run**

 This will:

* Build Docker image
* Push it to Azure Container Registry
  
![screenshorts](screenshorts/Screenshot%202025-07-08%20144107.png)

---

### STEP 5: Create CD Pipeline (Deploy to ACI)

#### 5.1 Create a New Pipeline (Release Pipeline)

1. Go to **Pipelines > Releases**
2. Click **+ New pipeline**
3. **Select an empty job**

#### 5.2 Add Artifact

1. Click **Add an artifact**
2. Select:

   * Source type: **Build**
   * Source: select the build pipeline you created
   * Default version: Latest
3. Click **Add**

#### 5.3 Define Deployment Stage

1. Click **Stage 1 > Tasks**
2. Rename to: `Deploy to ACI`
3. Click **+** to add task
4. Search **Azure CLI** → Add

#### 5.4 Configure Azure CLI Task

1. Azure Subscription: `AzureConnection`
2. Script type: `Inline script`
3. Inline Script:

```bash
az container create \
  --resource-group aci-devops-rg \
  --name flask-container \
  --image acichanduacr.azurecr.io/flaskapp:latest \
  --registry-login-server acichanduacr.azurecr.io \
  --registry-username $(acr-username) \
  --registry-password $(acr-password) \
  --dns-name-label flask-aci$(Release.ReleaseId) \
  --ports 80
```

![screenshorts](screenshorts/Screenshot%202025-07-08%20134419.png)

---

#### 5.5 Create Pipeline Variables

1. Go to **Variables**
2. Add:

   * Name: `acr-username` → Value: (From ACR > Access Keys)
   * Name: `acr-password` → Value: (From ACR > Access Keys)
   * Set both as **Secrets**

---

#### 5.6 Save and Trigger Release

1. Click **Save**
2. Click **Create Release > Create**
3. Monitor the pipeline and logs

 This will:

* Deploy the Docker image to Azure Container Instance
* Create a public DNS link

---

###  STEP 6: Verify Your Deployment

1. Go to **Azure Portal > Container Instances**
2. Select `flask-container`
3. Copy the **FQDN** (e.g., `http://flask-aci1234.eastus.azurecontainer.io`)
4. Paste in browser → App should load!

---

![screenshorts](screenshorts/Screenshot%202025-07-08%20144107.png)
![screenshorts](screenshorts/Screenshot%202025-07-08%20145701.png)


# *For expected output there is some problem that Azure DevOps does not enable hosted agents (the cloud build runners)*

# *✅ Solution: Request Free Hosted Agent Parallelism*
## You need to request Microsoft to enable free pipeline minutes for your DevOps organization.

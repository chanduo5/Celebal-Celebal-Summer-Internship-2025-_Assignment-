# *Week-6 {Assignment}*





------------------------------------------------------------------------------------------------------------------
# **Deploy Replica Set and Replication Controller, and deployment. Also learn the advantages and disadvantages of each.**
------------------------------------------------------------------------------------------------------------------


## Step-by-Step Guide

---

### Step 1: Create an AKS Cluster in Azure

#### Using Azure Portal

1. Go to **Azure Portal → Kubernetes services → Create → Kubernetes cluster**.
2. Under **Basics**:

   * Subscription: Your Azure subscription
   * Resource Group: Create or select an existing one
   * Cluster Name: `myAKSCluster`
   * Region: Choose your closest Azure region
   * Kubernetes version: Use default or latest
3. Under **Node Pools**:

   * Node count: 1 or 2
4. Review and create → Wait for deployment

> Once complete, go to the cluster overview page.

---

### Step 2: Connect to Your AKS Cluster

#### Using Azure CLI

```bash
az login
az aks get-credentials --resource-group <Your-ResourceGroup> --name myAKSCluster
```

> Example:

```bash
az aks get-credentials --resource-group MyResourceGroup --name myAKSCluster
```

Now verify with:

```bash
kubectl get nodes
```

---

### Step 3: Create YAML for ReplicationController

Create a file named `rc.yaml`:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 2
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f rc.yaml
kubectl get pods
```

---

### Step 4: Create YAML for ReplicaSet

Create a file named `replicaset.yaml`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f replicaset.yaml
kubectl get replicasets
kubectl get pods
```

---

### Step 5: Create YAML for Deployment

Create a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

---

### Step 6: Expose Services (Optional)

To expose one of the deployments:

```bash
kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80
kubectl get service
```

> Wait a few minutes for Azure to provision an external IP.

---








------------------------------------------------------------
# **Kubernetes service types (ClusterIP, NodePort, LoadBalancer)**
--------------------------------------------------------------

---

## Step-by-Step: Service Types in AKS

---

### 1. ClusterIP (Default – Internal only)

#### Use Case:

* Internal communication between microservices.
* Not accessible from outside the AKS cluster.

#### YAML Manifest (clusterip-service.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

#### Apply:

```bash
kubectl apply -f clusterip-service.yaml
```

#### Test Access:

You must **exec into a pod** inside the cluster to test:

```bash
kubectl run test-pod --image=busybox -it --restart=Never -- sh
wget --spider clusterip-service
```

---

### 2. NodePort (External access via AKS node IP + port)

#### Use Case:

* For testing or external access **without LoadBalancer**.
* Not ideal for production unless behind a cloud-native Load Balancer.

#### YAML Manifest (nodeport-service.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080  # (Optional, otherwise auto-assigned)
```

#### Apply:

```bash
kubectl apply -f nodeport-service.yaml
```

#### Access:

1. Get node external IP (AKS uses internal IPs by default, so use Azure Public IP if attached):

```bash
kubectl get nodes -o wide
```

2. Then access via:

```bash
http://<NodeIP>:30080
```

 In managed AKS, NodePort isn’t reliably accessible unless you use a **Load Balancer or port forwarding** (next option recommended).

---

### 3. LoadBalancer (Best practice for Azure public access)

#### Use Case:

* **Expose a service publicly** using an **Azure Load Balancer**.
* Most common for exposing apps to the internet.

#### YAML Manifest (loadbalancer-service.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

#### Apply:

```bash
kubectl apply -f loadbalancer-service.yaml
```

#### Access:

```bash
kubectl get service loadbalancer-service
```

Look for the **EXTERNAL-IP** column:

```bash
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
loadbalancer-service    LoadBalancer   10.0.245.55    20.204.33.11    80:30769/TCP
```

Open:

```
http://<EXTERNAL-IP>
```

This IP is automatically provisioned by **Azure Standard Load Balancer**.

---







--------------------------------------------------------------------------
# **PersistentVolume (PV) and PersistentVolumeClaim (PVC)**
-------------------------------------------------------------------------

---

## What are PV and PVC?

| Term                            | Description                                                                                  |
| ------------------------------- | -------------------------------------------------------------------------------------------- |
| **PersistentVolume (PV)**       | A piece of storage in the cluster provisioned by an admin or dynamically by a storage class. |
| **PersistentVolumeClaim (PVC)** | A request for storage by a user (e.g., size, access mode).                                   |
| **StorageClass (SC)**           | Defines the "class" or type of storage (e.g., AzureDisk, AzureFile).                         |

---

## Storage Options in Azure AKS

In AKS, PVs are typically backed by:

* **Azure Disks** (Block storage, good for one Pod)
* **Azure Files** (File share, supports multi-node access)

---

## Step-by-Step: Static and Dynamic Provisioning

---

### Option 1: Static Provisioning (Manually create PV)

---

#### Step 1: Create PersistentVolume (pv.yaml)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  azureDisk:
    kind: Managed
    diskName: myAKSDisk
    diskURI: /subscriptions/<SUB_ID>/resourceGroups/<RG>/providers/Microsoft.Compute/disks/myAKSDisk
  persistentVolumeReclaimPolicy: Retain
```

> You must manually create `myAKSDisk` in Azure Portal or CLI first.

---

#### Step 2: Create PersistentVolumeClaim (pvc-static.yaml)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeName: static-pv
```

---

#### Apply:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc-static.yaml
```

---

### Option 2: Dynamic Provisioning (Recommended in AKS)

Azure handles volume creation using a `StorageClass`.

---

#### Step 1: Default StorageClass (already available in AKS)

Check it:

```bash
kubectl get storageclass
```

You’ll typically see:

```
default (true) kubernetes.io/azure-disk ...
```

You can also create your own SC:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk-sc
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

---

#### Step 2: Create PVC (pvc-dynamic.yaml)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: azure-disk-sc  # or omit to use default
  resources:
    requests:
      storage: 2Gi
```

---

#### Apply:

```bash
kubectl apply -f pvc-dynamic.yaml
```

Then check the status:

```bash
kubectl get pvc
kubectl get pv
```

You’ll see a **new Azure disk automatically created** and bound.

---

### Step 3: Use PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/mnt/storage"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

Apply it:

```bash
kubectl apply -f pod.yaml
```

---

## Access Modes Explained

| Access Mode   | Description                             |
| ------------- | --------------------------------------- |
| ReadWriteOnce | One node can read/write                 |
| ReadOnlyMany  | Many nodes can read (not write)         |
| ReadWriteMany | Many nodes can read/write (Azure Files) |

---






-----------------------------------------------------------------------------------------------------------
# **Managing Kubernetes with Azure Kubernetes Service (AKS), Creating and managing AKS clusters, Scaling and upgrading AKS clusters**
-------------------------------------------------------------------------------------------------------------



---

## 1. What is AKS?

**Azure Kubernetes Service (AKS)** is a **managed Kubernetes service** in Azure, making it easy to deploy and manage containerized apps without handling Kubernetes master nodes or the underlying infrastructure.

---

## 2. Creating AKS Clusters

#### A. Using Azure Portal

1. **Go to** Azure Portal → **Kubernetes services → + Create**
2. **Basics Tab**

   * Subscription: your subscription
   * Resource Group: create new or select existing
   * Cluster name: `myAKSCluster`
   * Region: select nearest (e.g., East US, Central India)
   * Kubernetes version: latest (recommended)
3. **Node Pools**

   * Node size: Standard\_DS2\_v2 (default)
   * Node count: 1 or more (can scale later)
4. **Authentication**, **Networking** → Leave defaults (or customize if needed)
5. **Review + Create** → Click **Create**

---

#### B. Using Azure CLI

```bash
# Login
az login

# Create resource group
az group create --name myResourceGroup --location eastus

# Create AKS cluster with 2 nodes
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys
```

---

## 3. Managing and Accessing Clusters

#### Connect with `kubectl`

```bash
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Verify:

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

#### View AKS Cluster in Portal

Go to Azure Portal → Kubernetes services → `myAKSCluster`

---

## 4. Scaling AKS Clusters

#### A. Scale Node Count

##### CLI

```bash
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 4
```

####  Azure Portal

1. Go to AKS cluster → Node pools
2. Select default node pool
3. Click “Scale” → Set desired node count → Save

---

#### B. Enable Cluster Autoscaler (Optional)

```bash
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

---

## 5. Upgrading AKS Clusters

#### A. Check available upgrades

```bash
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

#### B. Upgrade Cluster Version

```bash
az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version <new-version>
```

> Example:

```bash
az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version 1.29.2
```

#### C. Upgrade Node Pool (if needed)

```bash
az aks nodepool upgrade \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --kubernetes-version 1.29.2
```

---




-----------------------------------------------------------------------------------------
# **Configure Taints and Tolerants.**
---------------------------------------------------------------------------------------


---

## What Are Taints and Tolerations?

| Concept                 | Description                                                     |
| ----------------------- | --------------------------------------------------------------- |
| **Taint (on Node)**     | Prevents pods from scheduling unless they tolerate the taint.   |
| **Toleration (on Pod)** | Allows the pod to be scheduled on a node with a matching taint. |

Taints repel pods. Tolerations are how pods say, “I'm okay with that.”

---

### Use Case Example

* You want **“critical workloads”** to run only on **dedicated nodes**, and prevent general workloads from running there.

---

## Step-by-Step in Azure AKS

---

### 1. Add a Taint to a Node

#### Option A: Using Azure CLI

First, get your node name:

```bash
kubectl get nodes
```

Then add a taint to it:

```bash
kubectl taint nodes <NODE_NAME> dedicated=critical:NoSchedule
```

Example:

```bash
kubectl taint nodes aks-nodepool1-12345678-vmss000000 dedicated=critical:NoSchedule
```

> `NoSchedule` means **pods without matching tolerations will not be scheduled here**.

---

### 2. Test Without Toleration (Expected: Fail to schedule)

Create a sample pod **without toleration**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-no-toleration
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply it:

```bash
kubectl apply -f pod-no-toleration.yaml
kubectl describe pod demo-no-toleration
```

>  The pod will **not get scheduled** on the tainted node.

---

### 3. Add a Toleration (Allow Pod on Tainted Node)

Now create a pod **with matching toleration**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-toleration
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "critical"
    effect: "NoSchedule"
```

Apply:

```bash
kubectl apply -f pod-toleration.yaml
```

 This pod will be scheduled on the **tainted node**.

---




--------------------------------------------------------------------------------------------
# **Create and attach persistent volume claims to pods.**
--------------------------------------------------------------------------------------------


---

### What You’ll Build

You’ll create:

* A PVC that requests a **persistent volume**
* A Pod (via Deployment) that **mounts that volume**
* Storage provisioned using Azure Disk (via default StorageClass)

---

## Prerequisites

* AKS Cluster created and `kubectl` connected:

```bash
az login
az aks get-credentials --resource-group <YourRG> --name <YourAKSCluster>
kubectl get nodes
```

---

### Step 1: Create a PersistentVolumeClaim (PVC)

Create a file named `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: default
```

> `ReadWriteOnce` is ideal for **Azure Disk** which supports a single writer pod.

Apply it:

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

Check the status — it should say `Bound` after a few seconds.

---

### Step 2: Create a Deployment that uses the PVC

Create a file named `deployment-with-pvc.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-pvc-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-pvc
  template:
    metadata:
      labels:
        app: nginx-pvc
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: demo-pvc
```

Apply it:

```bash
kubectl apply -f deployment-with-pvc.yaml
```

---

### Step 3: Validate Volume Attachment

Check the pod is running:

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

Check the volume is mounted inside the container:

```bash
kubectl exec -it <pod-name> -- sh
df -h       # shows mounted volumes
ls /usr/share/nginx/html
```

You can create a file inside to test:

```bash
echo "Hello from Azure PVC!" > /usr/share/nginx/html/index.html
```

Then, if you expose the service:

```bash
kubectl expose deployment nginx-pvc-deployment --type=LoadBalancer --port=80
kubectl get service
```

Visit the `EXTERNAL-IP` in your browser and you’ll see the content.

---



------------------------------------------------------------------------------------------
# **Configure health probes for pods.**
-------------------------------------------------------------------------------


---

## What Are Health Probes?

| Probe Type          | Purpose                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Liveness Probe**  | Checks if the container is **alive**. If it fails, the container is restarted.                                       |
| **Readiness Probe** | Checks if the container is **ready to receive traffic**. If it fails, the pod is removed from the service endpoints. |

---

## Step-by-Step: Configure Liveness and Readiness Probes in AKS

---

### 1. Create a Sample Deployment with Probes

Create a file named `nginx-health-probes.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-health-check
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-check
  template:
    metadata:
      labels:
        app: nginx-check
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

#### Explanation:

| Probe         | Method    | Path | Behavior                                                                                                                 |
| ------------- | --------- | ---- | ------------------------------------------------------------------------------------------------------------------------ |
| **Liveness**  | `httpGet` | `/`  | Starts checking 10s after pod start, every 5s. If it fails, the pod is restarted.                                        |
| **Readiness** | `httpGet` | `/`  | Starts checking 5s after pod start, every 5s. If it fails, the pod is marked unready and removed from service endpoints. |

---

### 2. Apply the Deployment

```bash
kubectl apply -f nginx-health-probes.yaml
```

---

### 3. Verify the Probes Are Working

Get the pod name:

```bash
kubectl get pods
```

Describe the pod:

```bash
kubectl describe pod <pod-name>
```

You’ll see `Liveness` and `Readiness` probes configured and their current status.

---

### 4. Test Probe Failures (Optional)

You can break the health check by simulating a failure. For example, if you change the probe path to a non-existent path `/fail`, Kubernetes will fail the probe:

Edit and change:

```yaml
livenessProbe:
  httpGet:
    path: /fail
    port: 80
```

Apply the new manifest, and then:

```bash
kubectl describe pod <pod-name>
```

You'll start seeing **probe failures** and **container restarts**.

---

## Alternative Probe Types

Kubernetes supports other probe types:

#### TCP Probe

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 10
  periodSeconds: 5
```

#### Exec Probe

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 10
```

---



---------------------------------------------------------------------------------
# **Configure autoscaling in your cluster (Horizontal scaling)**
---------------------------------------------------------------------------------



## Step-by-Step Guide to Enable HPA in AKS

---

### 1. Prerequisites

* An AKS cluster is running
* `kubectl` and `Azure CLI` are installed
* You’re connected to the cluster:

```bash
az login
az aks get-credentials --resource-group <YourRG> --name <YourAKSCluster>
```

---

### 2. Enable Metrics Server in AKS

### Option A: Install manually via YAML

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

> Wait a few seconds, then test with:

```bash
kubectl top nodes
kubectl top pods
```

> If you see CPU/Memory stats, the metrics server is working.

---

### 3. Create a Deployment to Autoscale

Use a simple app like `nginx`. Create `hpa-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-autoscale
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-autoscale
  template:
    metadata:
      labels:
        app: nginx-autoscale
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
```

> Adding CPU requests and limits is **mandatory** for HPA to work!

Apply it:

```bash
kubectl apply -f hpa-deployment.yaml
```

---

### 4. Create the Horizontal Pod Autoscaler (HPA)

```bash
kubectl autoscale deployment nginx-autoscale --cpu-percent=50 --min=1 --max=5
```

Check status:

```bash
kubectl get hpa
```

You’ll see something like:

```
NAME              REFERENCE                    TARGETS         MINPODS   MAXPODS   REPLICAS
nginx-autoscale   Deployment/nginx-autoscale   20% / 50%       1         5         1
```

---

### 5. Generate CPU Load (to test auto-scaling)

Run a stress pod to generate load:

```bash
kubectl run -it --rm load-generator --image=busybox /bin/sh
```

Inside the shell:

```sh
while true; do wget -q -O- http://nginx-autoscale; done
```

> In another terminal, run:

```bash
kubectl get hpa
```

After a few minutes, replicas will increase to handle the load.

---

### 6. Stop Load and Watch Scale Down

Once you exit the `busybox` shell, the load stops. HPA will **automatically reduce the replicas** back to the original count when usage falls below the threshold.

---

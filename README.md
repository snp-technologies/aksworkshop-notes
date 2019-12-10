# AKS Workshop Notes
Proctor notes for The Azure Kubernetes Workshop, https://aksworkshop.io, supplemental to the workshop documentation.

<!-- TOC -->

- [AKS Workshop Notes](#aks-workshop-notes)
  - [1.1 Prerequisites](#11-prerequisites)
  - [1.2 Kubernetes basics](#12-kubernetes-basics)
  - [1.3 Application Overview](#13-application-overview)
  - [1.4 Scoring](#14-scoring)
  - [1.5 Tasks](#15-tasks)
- [2 Getting up and running](#2-getting-up-and-running)
  - [2.1 Deploy Kubernetes with Azure Kubernetes Service (AKS)](#21-deploy-kubernetes-with-azure-kubernetes-service-aks)
    - [Concepts](#concepts)
    - [Solution notes](#solution-notes)
    - [Tips](#tips)
  - [2.2. Deploy MongoDB](#22-deploy-mongodb)
    - [Concepts](#concepts-1)
    - [In Hints](#in-hints)
    - [Tips](#tips-1)
    - [Notes](#notes)
  - [2.3 Deploy the Order Capture API](#23-deploy-the-order-capture-api)
    - [Concepts](#concepts-2)
    - [Tasks](#tasks)
    - [Tips](#tips-2)
  - [2.4 Deploy the frontend using Ingress](#24-deploy-the-frontend-using-ingress)
    - [Concepts](#concepts-3)
    - [Tasks](#tasks-1)
  - [2.5 Enable SSL/TLS on ingress](#25-enable-ssltls-on-ingress)
    - [Concepts](#concepts-4)
    - [Tasks](#tasks-2)
      - [Order capture TLS Steps](#order-capture-tls-steps)
    - [Tips](#tips-3)
  - [2.6 Monitoring](#26-monitoring)
    - [Concepts](#concepts-5)
    - [Tasks](#tasks-3)
    - [Tips](#tips-4)
    - [Resources](#resources)
  - [2.7 Scaling](#27-scaling)
    - [Concepts](#concepts-6)
    - [Tasks](#tasks-4)
    - [Tips](#tips-5)
    - [Resources](#resources-1)
  - [2.8 Create private highly available container registry](#28-create-private-highly-available-container-registry)
    - [Concepts](#concepts-7)
    - [Tasks](#tasks-5)
    - [Tips](#tips-6)
    - [Resources](#resources-2)
- [DevOps Tasks](#devops-tasks)
  - [3.1 Continuous Integration and Continuous Delivery](#31-continuous-integration-and-continuous-delivery)
    - [Concepts](#concepts-8)
    - [Tasks](#tasks-6)
    - [Tips](#tips-7)
    - [Resources](#resources-3)
    - [Resources](#resources-4)
- [Misc Notes](#misc-notes)

<!-- /TOC -->

## 1.1 Prerequisites

- Azure subscription
  - The `az login...` command is not required if using Azure Shell. 
  - One can also log in with the Azure account Username/Password provided.
- Azure DevOps Organization account
- GitHub account 

## 1.2 Kubernetes basics

Also point users to:

- <https://kubernetes.io/docs/home/>
- <https://kubernetes.io/docs/tutorials/kubernetes-basics/>
- <https://azure.microsoft.com/en-us/topic/what-is-kubernetes/>

## 1.3 Application Overview

Nothing else to add.

## 1.4 Scoring

Optional, at the discretion of the Emcee. More practical when there is one or more proctors, in addition to the Emcee.

## 1.5 Tasks

# 2 Getting up and running

## 2.1 Deploy Kubernetes with Azure Kubernetes Service (AKS)

### Concepts

- Deployment methods, not just CLI and Portal (Azure Shell). Also, PowerShell, ARM, Terraform, Ansible
- K8S versions and AKS cadence update
  - AKS aims to certify and release new Kubernetes versions within 30 days of an upstream release, subject to the stability of the release.
- kubectl, the K8s CLI
- Regions, verify that AKS is available in a given region and what versions are supported
  - `az aks get-versions -l eastus`
- Service principal

### Solution notes

> Deploy Kubernetes to Azure, using CLI or Azure portal using the latest Kubernetes version available in AKS

- Visit https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough for step by step
- Note that solutions use Azure Shell. You can use Azure CLI on your lap top (install the Azure CLI). PowerShell is an alternative, if you want to be difficult ;>).
  - Open Azure Shell in a new browser tab using the URL <https://shell.azure.com>. When you connect for the first time, you will need to configure a storage account. This is needed for file persistence.

> Do not install with the cluster autoscaler, i.e. with the `--enable-cluster-autoscaler` parameter. It takes longer to install and may  increase the liklihood of cluster failure during the workshop.

|Step|Method|
|--|--|
|SSH Key|If using your own Azure subscription, you can use your own SSH key, and not use the `--generate-ssh-keys` option in the `az create aks` command. Instead, use the `--ssh-key-value /path` option.|
|Set env variables used in the lab|Resource Group, Location, Cluster name. See tip below.|
|Create resource group|`az group create --name $resource-group --location $region` **IMPORTANT**: If you have been given access to a subscription as part of a lab, an RG has probably been provisioned (e.g. ODL-k8s-68593). You will see this in Azure Portal. If so, use this RG instead of `akschallenge`. Note the location of the RG.|
|Get V8s versions for my region|az aks get-versions --location $region -o table|
|Create AKS cluster|`az aks create --resource-group $resource-group --name  $aksname --enable-addons monitoring --kubernetes-version 1.12.6 --generate-ssh-keys --location $region --service-principal $APP_ID --client-secret $APP_SECRET`|
|Get Cluster creds|`az aks get-credentials -g $resource-group -n $aksname`|
|Verify cluster access|`kubectl get nodes`|

> Cluster creds are stored in `~/.kube/config`. Open the files with `code ~/.kube/config`. Look for current context.

### Tips

1. Create a `aksworkshop` directory in your home dir (`~/`). Store code files here, e.g. K8s manifests.
2. Add env vars to `.bashrc`
```
code ~/.bashrc
// at the bottom of the .bashrc file, enter:
export APP_ID=<your sp client id>
export objectid=<your sp object id>
export subid=<your subscription id>
export APP_SECRET=<your sp password>
export resource-group=aksworkshop
export region=eastus
export aksname=aksworkshop
export KUBE_EDITOR="nano"
```

3. Add `alias k=kubectl` to `.bashrc`.
4. Refresh the shell: `source ~/.bashrc`
5. In Azure Shell use the `code` command to open a user-friendly  editor directly in Azure Shell (or use `vi` or `nano`, if you prefer).

## 2.2. Deploy MongoDB

### Concepts
- Helm
- ConfigMap as in
https://github.com/helm/charts/blob/master/stable/mongodb/templates/configmap.yaml
 - Secrets as in 
https://github.com/helm/charts/blob/master/stable/mongodb/templates/secrets.yaml 

### In Hints

2.2. Deploy MongoDB

> _Be careful with the authentication settings when creating MongoDB. It is recommended that you create a standalone username/password and database._

1. Why _Be careful_?
2. How does creating a standalone username/password and database help?
Note, if a database does not exist, MongoDB creates the database when you first store data for that database.
3. Why refer to this as a “standalone”? Why not use the language from the values file, “MongoDB custom user and database”. standalone is used in the helm chart, but not specifically where a custom database is defined. (Search on mongodbDatabase in https://github.com/helm/charts/blob/master/stable/mongodb/templates/deployment-standalone.yaml.)
4. Why use **stable.mongo** and not **mongodb-replicaset** as the Helm chart? See <https://github.com/helm/charts/issues/13186>.

### Tips

1. To Check if a cluster is RBAC enabled, visit https://resources.azure.com/ and drill down, e.g.  
https://resources.azure.com/subscriptions/[SUBSCRIPTIONID]/resourceGroups/[resource-group]/providers/Microsoft.ContainerService/managedClusters/[aksname]

    In my case, I see:

    ```
    "nodeResourceGroup": "MC_rgmichael28849_aksmichael28849_eastus",
        "enableRBAC": **true**,
        "networkProfile": {
        "networkPlugin": "kubenet",
    ```

1. to verify `helm`, run `helm version`, e.g.
    ```
    version.BuildInfo{Version:"v3.0.0", GitCommit:"e29ce2a54e96cd02ccfce88bee4f58bb6e2a28b6", GitTreeState:"clean", GoVersion:"go1.13.4"}
    ```

1. APPLIES TO HELM v2 ONLY! If helm is already installed, and you get error messages like:
   
    ```
    Error: release orders-mongo failed: namespaces "default" is forbidden: User "system:serviceaccount:kube-system:default" cannot get resource "namespaces" in API group "" in the namespace "default"
    ```
    run: `helm init --service-account tiller --upgrade`
    
    

1. APPLIES TO HELM v2 ONLY! If you get error:
    ```
    Error: could not find a ready tiller pod
    ```
    Then, the tiller pod is not ready. Give it some more time. Check with `helm version`.

1. When Mongo helm chart completes, there will be output such as that below. Copy and paste this into a notepad, in case you need to reference it later.

```
michael@Azure:~$ helm install orders-mongo stable/mongodb --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
NAME: orders-mongo
LAST DEPLOYED: Wed Dec  4 14:37:32 2019
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

MongoDB can be accessed via port 27017 on the following DNS name from within your cluster:

    orders-mongo-mongodb.default.svc.cluster.local

To get the root password run:

    export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default orders-mongo-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)

To get the password for "orders-user" run:

    export MONGODB_PASSWORD=$(kubectl get secret --namespace default orders-mongo-mongodb -o jsonpath="{.data.mongodb-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run --namespace default orders-mongo-mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb --command -- mongo admin --host orders-mongo-mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/orders-mongo-mongodb 27017:27017 &
    mongo --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD
```    

2. Verify Mongo by: 
- run `kubectl get po` to validate that the pod is running. You should get a results like:
```
NAME                                    READY   STATUS    RESTARTS   AGE
orders-mongo-mongodb-76c879dbc7-p2vlc   1/1     Running   0          7m36s
```
- Get a shell to the running Container, e.g. 
```
kubectl exec -it orders-mongo-mongodb-76c879dbc7-p2vlc -- /bin/bash
```
- Once in the shell, run `mongo`. You should get output like:

```
MongoDB shell version v4.0.3
connecting to: mongodb://127.0.0.1:27017
Implicit session: session { "id" : UUID("d2b7fba4-51ca-4115-b02c-388a2568fa4d") }
MongoDB server version: 4.0.3
Welcome to the MongoDB shell.
For interactive help, type "help".
<and so on>
```

### Notes

1. In https://github.com/helm/charts/blob/master/stable/mongodb/values.yaml, note this code that is commented out:
    ```
    ## MongoDB custom user and database
    ## ref: https://github.com/bitnami/bitnami-docker-mongodb/blob/master/README.md#creating-a-user-and-database-on-first-run
    ##
    # mongodbUsername: username
    # mongodbPassword: password
    # mongodbDatabase: database
    ```

    These values are consumed in https://github.com/helm/charts/blob/master/stable/mongodb/templates/deployment-standalone.yaml. 

    ... Which in turn are passed on docker run during helm chart installation. (See https://github.com/bitnami/bitnami-docker-mongodb/blob/master/README.md#creating-a-user-and-database-on-first-run) 

    This is why they are passed on the `helm install` command line.

    Alternatively, can a secret be used? How would that be done?

## 2.3 Deploy the Order Capture API

### Concepts

- Health checks in K8S - Liveness and readiness probes:
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/

- Services: Types, labels, selector (see Resource link)
    
### Tasks

- Provision the captureorder deployment and expose a public endpoint
    - Ensure orders are successfully written to MongoDB
    - The curl solution is not obvious. Let’s break it down:

    ```
    curl -d '{"EmailAddress": "mike@snp.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -X POST http://40.121.XXX.XXX/v1/order
    ```

    -d: (HTTP) Sends the specified data in a POST request to the HTTP server.
    
    -H: (HTTP) Extra header to include in the request when sending HTTP to a server.

    -X: (HTTP) Specifies a custom request method to use when communicating with the HTTP server. Default is GET. We are using POST

    POST http://40.121.XXX.XXX/v1/order

- Access the Swagger UI at http://[host]/swagger.
For me: http://40.121.XXX.XXX/swagger/

- With regard to the hint:

    > The Order Capture API exposes the following endpoint for health-checks: http://[PublicEndpoint]:[port]/healthz”...

    …note the /healthz path. This is coded in the application. Refer to:
<https://github.com/Azure/azch-captureorder/blob/1a4b34528882d9fb85a3d88246ddc53b74801f79/tests/default_test.go#L23>

  I found that I needed to leave out the port. The browser output should be:

  `i'm alive!`

### Tips

1. `k get svc` // to list k8s service resources

## 2.4 Deploy the frontend using Ingress

### Concepts

- Ingress controller
- Cluster IP service  

### Tasks

- Deployment of app is straight forward
- Deployment of Ingress controller is also straight forward, but note...

> Re: "ProTip: Place in the ingress controller in a different namespace". By following this practice, one can use the controller to route traffic across name spaces. There is a configurable setting in the Helm chart, controller.scope, that can limit the controller to a specific namespace, but it’s off by default. This tip is good to know, but not necessary.  
  

> Note: Show and discuss the **Architecture diagram** that appears at the end of the task.

## 2.5 Enable SSL/TLS on ingress

### Concepts

- cert-manager K8s add on
- ClusterIssuer

### Tasks

- Straight forward for frontend. Order capture is a challenge. See notes below.
- In cert-manager install there is a step to `Install the CustomResourceDefinition resources separately`. What is a CRD and why is it needed? [Custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) are extensions of the Kubernetes API. CustomResourceDefinition is an API resource allows you to define custom resources that can be served and handled by the K8s API. In the context of `cert-manager`, this is where the `cert-manager` API is defined. 

#### Order capture TLS Steps

- Repeat the work you did when creating the frontend ingress, but this time for the captureorder service, i.e. direct traffic via the ingress (create an second ingress object using YAML)
- The hostname will need to be different, but still point at your ingress controller IP, e.g. orders.{ingress-ip}.nip.io. You must set up TLS for it as you did with the frontend.

  `cp frontend-ingress-tls.yaml captureorder-ingress-tls.yaml`
  `code captureorder-ingress-tls.yaml`
  
  Update as follows, and save:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: captureorder
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - orders.23.96.4.79.nip.io
    secretName: orders-tls-secret
  rules:
  - host: orders.23.96.4.79.nip.io
    http:
      paths:
      - backend:
          serviceName: captureorder
          servicePort: 80
        path: /
```
- Modify the CAPTUREORDERSERVICEIP environmental variable in the frontend deployment YAML. This now needs to refer to the hostname of your orders ingress, re-deploy the frontend to make the changes live.

`cp frontend-deployment.yaml frontend-deployment-lb.yaml // backup the original deployment yaml` 

`code frontend-deployment.yaml`

```
env:
- name: CAPTUREORDERSERVICEIP
  value: "orders.23.96.4.79.nip.io"  # Replace with your hostname of your orders ingress
```
### Tips

Note: There is no need to update `captureorder-service.yaml`, but in the real-world you would delete the service since the use of the ingress controller is preferred. That said, you may not want to expose the service outside your cluster, in which case would a Cluster ID be sufficient?

## 2.6 Monitoring

### Concepts

- Azure Monitor for containers
- Azure Monitor
- Log Analytics
- ClusterRole & ClusterRoleBinding
- Prometheus

### Tasks

- A **Log Analytics workspace** is a unique environment for Azure Monitor log data. Each workspace has its own data repository and configuration, and data sources and solutions are configured to store their data in a particular workspace.
- Instructions around Log Analytics workspace set up are a bit confusing. If user is bringing their own subscription, they can check in Azure portal to see if there is an existing workspace they can use.
- If you create a new workspace, navigate to it in Azure Portal.
- We could have enabled the **AKS monitoring addon** when we created our cluster. You can [also enable](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-onboard) from  Azure Portal, ARM Template, etc.
- In the Insights blade there is a message "Enable fast alerting experience on basic metrics for this Azure Kubernetes Services cluster". Click the Enable button.

   

### Tips

- From [Troubleshooting link](https://github.com/Microsoft/OMS-docker/tree/ci_feature/Troubleshoot) in blade: 
  > It may take some time to establish monitoring data flow for newly created clusters. Please allow at least 10-15 minutes for data to appear for your cluster.

  **I found it took over 30 minutes to start seeing data!** Move user onto the Scaling task, and then come back to Monitoring.

### Resources

- Another lab: <https://github.com/Azure/kubernetes-hackfest/blob/master/labs/monitoring-logging/README.md>
- <https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/>

## 2.7 Scaling

### Concepts

- Azure Container Instances
- Horizontal Pod Autoscaler

### Tasks

- Run a baseline load test. Straight forward. My az command: 

    ```
    az container create -g $resource-group -n loadtest --image azch/loadtest --restart-policy Never -e SERVICE_ENDPOINT=https://orders.23.96.4.79.nip.io/v1/order
    ```
    ```
    az container logs -g $resource-group -n loadtest
    ```
    ```
    az container delete -g $resource-group -n loadtest
    ```
- Create Horizontal Pod Autoscaler.
  - The **Important** note in the solution requires edits to captureorder-deployment.yaml to remove the explicit `replicas: 2` count. There is also an instruction to define resource requests and resource limits. This is included in `captureorder-deployment.yaml`:
  ```
  resources:
    requests:
      memory: "128Mi"
      cpu: "100m"
    limits:
      memory: "256Mi"
      cpu: "500m"
  ```
  This requirement is mentioned in <https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-scale>. 
  

- Run a load test again after applying Horizontal Pod Autoscaler
  - Typo in `kubectl get pods -l`. Remove the `-l`. 
  - I recommend `kubectl get pods --watch`
  - Run `az container logs -g akschallenge -n loadtest`
  - After the load test ends, observe that the # of pods scales back down to 1.
- Check if your cluster nodes needs to scale/auto-scale
  - `az aks` command is given to scale cluster.
  - Important question is, "how do I know if I need more nodes?" 


### Tips

1. Run `k get deploy` to observe deployment resources:
    ```
    NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    captureorder           2         2         2            2           46h
    frontend               1         1         1            1           111m
    orders-mongo-mongodb   1         1         1            1           46h
    ```

### Resources

1. Another good resource is <https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/>. When you use the `az aks scale` command to scale down, AKS safely drains your nodes.

## 2.8 Create private highly available container registry

### Concepts

- Docker Image
- Azure Container Registry

### Tasks

- Create an Azure Container Registry (ACR)
  - Use Basic sku
- Use Azure Container Registry Build to push the container images to your new registry
  - `az acr login` can't be done from Azure shell. This command is only used if you are not using Azure shell.
  ```
  This command requires running the docker daemon, which is not supported in Azure Cloud Shell.
  ```
  - You can run `az acr build` from Azure shell, however...
  - This will take several minutes to complete. Note the following lines in the terminal output:
    ```
    - image:
    registry: <your unique reg name>.azurecr.io
    repository: captureorder
    tag: ca1
    ```
    These will be used in the next step.
  - If you are running the `az acr build` command in a terminal (not Azure Shell), you may get the following error:
    ```
    Some of the properties of 'buildRequest' are invalid. InnerErrors: QuickBuildRequest:Image name 'captureorder:{{.Run.ID}}' is invalid. \nThe acceptable patterns are 'repository' or 'repository:tag'.The repository and tag names follow the standardized docker repository and tag naming conventions. Accepted tag templates are '{{.Build.ID}},{{.Build.Date}}'
    ```
    If so, try this command instread:
    ```
    az acr build -t "captureorder:{{.Build.ID}}" -r <unique-acr-name> .
    ```
  - Explore the new ACR resource in Azure Portal.
- Configure your application to pull from your private registry
  - Grant AKS generated Service Principal to ACR
  - Create a Kubernetes Secret
- Now run `k apply -f captureorder-deployment.yaml`
  - Run `k get po --watch` to observe the captureorder pod restarting:
  ```
  michael@Azure:~/aksworkshop$ k get po --watch
    NAME                                    READY   STATUS    RESTARTS   AGE
    captureorder-57dc6cb5f6-w8bxd           0/1     Running   0          13s
    captureorder-69fd98b57b-d26vd           1/1     Running   0          3h51m
    frontend-55cf957d85-fx4l7               1/1     Running   0          3h14m
    orders-mongo-mongodb-6684cbf59f-wd49s   1/1     Running   0          3h56m
    captureorder-57dc6cb5f6-w8bxd   1/1   Running   0     15s
    captureorder-69fd98b57b-d26vd   1/1   Terminating   0     3h51m
    captureorder-69fd98b57b-d26vd   0/1   Terminating   0     3h51m
  ```
  - Run `k describe po captureorder-57dc6cb5f6-w8bxd` to confirm the use of the image in ACR:
  ```
  Image:          acrmike20190405.azurecr.io/captureorder:ca1
  ```

### Tips

Pending

### Resources

Pending

# DevOps Tasks

## 3.1 Continuous Integration and Continuous Delivery

### Concepts

- CI/CD
- Azure DevOps
- Azure DevOps Project

### Tasks

- Follow steps closely

- There is a line in the instructions **Inspect the Deploy stage and edit the `imageRepository` variable** that is not clear. There is no `imageRepository`.

- Verify everything works

### Tips

- To enable the Environments feature in Azure DevOps, turn on the **Multi-stage pipelines** Preview feature. This is mentioned later in the instructions, but it needs to be done first.
- In Azure Shell, you can see the new pods running in the `dev` namespace using `kubectl get po -n dev`. Likewise, you can see the deployment from `kubectl get deploy -n dev`.


### Resources


### Resources

1. https://hub.helm.sh/
2. Another lab <https://github.com/Azure/kubernetes-hackfest>
3. <https://www.katacoda.com/courses/kubernetes/helm-package-manager>

# Misc Notes

1. On trying to review live container logs for frontend container:
    ```
    Kube API Response
    pods "frontend-86bbcd8448-m7jfw" is forbidden: User "clusterUser" cannot get resource "pods/log" in API group "" in the namespace "default"
    ```
    I am able to see logs in logs analytics.
    No issues apparent from `k logs  frontend-86bbcd8448-m7jfw`.
    
2. I experienced a timeout error on Monday, April 8:
    ```
    michael@Azure:~$ k logs captureorder-69fd98b57b-bcnw2
    Error from server: Get https://aks-nodepool1-30479232-1:10250/containerLogs/default/captureorder-69fd98b57b-bcnw2/captureorder: dial tcp 10.240.0.4:10250: i/o timeout
    ```
    May be related to hot fix pushed on 4/4.
    
    Get list of upgrades:
    ```
    az aks get-upgrades -g $resource-group -n $aksname  --output table
    Name     ResourceGroup    MasterVersion    NodePoolVersion    Upgrades
    default  rgmike20190405   1.12.6           1.12.6             1.12.7
    ```
    ```
    michael@Azure:~$ az aks upgrade -g $resource-group -n $aksname --kubernetes-version 1.12.7
    Kubernetes may be unavailable during cluster upgrades.
    Are you sure you want to perform this operation? (y/n): y
    - Running ..
    ```




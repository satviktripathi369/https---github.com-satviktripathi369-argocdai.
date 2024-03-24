# Kubernetes deployment strategies
## Setup for Kubernetes, Docker and Argo CI CD (on Windows)

### Installing Kubernetes
```
curl.exe -LO "https://dl.k8s.io/release/v1.29.2/bin/windows/amd64/kubectl.exe"
```
Test to ensure the version of kubectl is the same as downloaded:
```
kubectl version --client
```
### Installing Docker
You can download docker from its ([official site](https://www.docker.com/products/docker-desktop/)) with the latest version 

### Installing Argo CI CD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
By default, the Argo CD API server is not exposed with an external IP. To access the API server follow below steps:
> - Port Forwarding
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access the UI for Argo CI CD through the IP mentioned while fetching the IP address and accessing it through a web browser http://127.0.0.1:8080/ (Proceed anyway since it is not HTTPS).

Username: admin
You can access the password through 
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
We have now finished the setup process. Now we Configure the process.

## Configuration of the Process
You need deployment yaml and service yaml files to deploy and expose your application. Make sure both files are configured properly. Make sure to have access to a Kubernetes cluster from any cloud provider. You can even use Minikube or Kind to create a cluster.

### Docker Containerization and using Kubernetes
Use the below commands to build and push the image:
```
docker build -t $your docker hub user name/$image name .
docker push $your docker hub user name/$image name .

```
Mine is: satviktripathi369/argocicd:v1

Manifest the deployment and service yaml file be ready for the further process of live deployment. Starting with deployment and then service yaml file.
```
kubectl apply -f canary-demo-v1-deployment.yml
```
likewise for the service

```
kubectl apply -f service.yaml
```
Verify the pods are running properly as expected after applying the kubectl apply commands.
```
kubectl get pods
```
> Cleaning up the pods
```
kubectl delete pods --all-namespaces
```

### Using ArgoCD via UI
Open a browser to the Argo CD external UI, and login by visiting the IP/hostname in a browser and use the credentials that you get in the previous steps.
After logging in, click the + New App button
Give your app the name guestbook, use the project default, and leave the sync policy as Manual
Connect the https://github.com/argoproj/argocd-example-apps.git (for sample purpose from ArgoCD) or [mine this repo](https://github.com/satviktripathi369/argocdai.git) repo to Argo CD by setting repository url to the github repo url, leave revision as HEAD, and set the path to guestbook
For Destination, set cluster URL to https://kubernetes.default.svc (or in-cluster for cluster name) and namespace to default.
After filling out the information above, click Create at the top of the UI to create the guestbook application.

### Sync (Deploy) The Application through the UI

In the Argo CD UI, you can monitor the status of the synchronization process for your application deployments. Here's how you can check if the syncing process is in progress or completed:

> Navigate to the Applications Tab: Open your web browser and go to the Argo CD UI.
> Click on the "Applications" tab in the left sidebar.
> Find Your Application:

> Locate your application from the list of applications displayed in the UI.
> Click on the name of your application to view its details.
> View Sync Status:

In the application details view, you'll see information about the synchronization status.
Look for the "Sync Status" section to see the current state of synchronization.
The possible states include:
Synced: Indicates that the application is in sync with the desired state defined in the Git repository. No changes need to be applied.
Syncing: Indicates that the synchronization process is currently in progress. Argo CD is comparing the current state of the application with the desired state defined in the Git repository and applying any necessary changes.
Out of Sync: Indicates that there are differences between the current state of the application and the desired state defined in the Git repository. Argo CD will attempt to reconcile these differences automatically.
View Sync Details:

You can click on the application name to view detailed information about the synchronization process.
This view provides information such as the sync status, sync revision, sync operation, sync health, and a timeline of sync events.
Monitor Sync Events:

Argo CD provides a timeline of sync events, which includes information about each synchronization operation, such as when it started, its duration, and its outcome (success or failure).
You can use this timeline to track the progress of synchronization and troubleshoot any issues that may arise.

> Cleaning up the app from ArgoCD UI: You can just delete the app which you have deployed.

## Challenges faced
> Summarizing the entire process, including challenges faced and solutions implemented, in a clear and concise manner may require careful organization and explanation to ensure it is understandable to both technical and non-technical audiences.
> Ensuring all resources created during the assignment are cleanly removed from the Kubernetes cluster without inadvertently deleting essential components or leaving behind any lingering artifacts can be challenging, requiring thorough verification and validation.
> Ensuring that the canary version is successfully deployed or not.
> Configuring Argo CD to correctly monitor the Git repository and automatically deploy changes to the Kubernetes cluster.
> Configuring the Dockerfile correctly to ensure all necessary components are included.

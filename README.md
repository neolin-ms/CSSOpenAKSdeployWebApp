# CSSOpen - Session 6 - Lab : Deploy a Web App on AKS

**Preequisites**

- Azure CLI version 2.0.53 or later.
- Need a local Docker development environment running Lnux containers.

## Deploy a web application on AKS

**1 - Prepare application for AKS**
1. Get application code.
> Command:<br>
> ```bash
> $ git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
> $ cd azure-voting-app-redis
> ```
2. Create container images 
> Command1:<br>
> ```bash
> $ docker-compose up -d
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/1_1.png "1_1")<br>
> Command2:<br>
> ```bash
> $ docker images
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/1_2.png "1_2")<br>
> Command3:<br>
> ```bash
> $ docker ps 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/1_3.png "1_3")<br>
3. Test application locally. To see the running application, enter `http://local:8080` in local web browser. The sample aplication, as shown in the following example.
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/1_4.png "1_4")<br>
4. Clean up resource.
> Command:<br>
> ```bash
> $ docker-compose down
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/1_5.png "1_5")<br>

**2 - Create container registry**
1. Create a Resource Group for Azure Container Registry.
> Command:<br>
> ```bash
> $ az group create --name myResourceGroup --location eastasia 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_1.png "2_1")<br>
2. Create an Azure Container Registry.
> Command:br>
> ```bash
> $ az acr create --resource-group myResourceGroup --name <acrName> --sku Basic 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_2.png "2_2")<br>
3. Log in to the container registry.
> Command:<br>
> ```bash
> $ az acr login --name <acrName>
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_3.png "2_3")<br>
4. See a list of your current local images.
> Command:<br>
> ```bash
> $ docker images
> ```  
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_4.png "2_4")<br>
5. Get the login server address.
> Command:<br>
> ```bash
> $ az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_5.png "2_5")<br>
6. Tag your local `azure-vote-front` image with the `acrLoginServer` address of the container registry. Add `:v1` to the end of the image name.
> Command:<br>
> ```bash
> $ docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v1
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_6.png "2_6")<br>
7. Verify the tags are applied.
> Command:<br>
> ```pash
> $ docker images
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_7.png "2_7")<br>
8. Puah the `azure-vote-front` image to your ACR instance.
> Command:<br>
> ```bash
> $ docker push <acrLoginServer>/azure-vote-front:v1
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_8.png "2_8")<br>
9. To return a list of images that have been pushed to your ACR instance. 
> Command:<br>
> ```bash
> $ az acr repository list --name <acrName> --output table 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_9.png "2_9")<br>
10. To see the tags for a specific image.
> Command:<br>
> ```bash
> $ az acr repository show-tags --name <acrName> --repository azure-vote-front --output table
> ``` 
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/2_10.png "2_10")<br>

**3 - Create Kubernetes cluster**
1. Create an AKS cluster.
> Command:<br>
> ```bash
> $ az aks create \
>      --resource-group myResourceGroup \
>      --name myAKSCluster \
>      --node-count 2 \
>      --generate-ssh-keys \
>      --attach-acr <acrName>
> ``` 
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/3_1.png "3_1")<br>
2. To connect to the Kubernetes cluster from your local compute, use `kubectl`. Install the Kubernetes CLI on Debian. 
> Command:<br>
> ```bash
> $ az aks install-cli 
> $ kubectl version --client
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/3_2.png "3_2")<br>
3. To configure `kubectl` to connect to your Kubernetes cluster. Gets credentials for the AKS cluster named `myAKSCluster` in the `myResourceGroup`.
> Command:<br>
> ```bash
> $ az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
> ``` 
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/3_3.png "3_3")<br>
4. To veify the connection to your cluster.
> Command:<br>
> ```bash
> $ kubectl get nodes
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/3_4.png "3_4")<br>

**4 - Run application**
1. Update the manifes file. Make sure that you're in the cloned `azure-voting-app-redis` directory, then open the manifest file with a text editor.
> Command:<br>
> ```bash
> $ vi azure-vote-all-in-one-redis.yaml  
> ```
> Replace `mcr.microsoft.com/azuredocs`_/azure-vote-front:v1_ with your ACR login server name.  
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/4_1.png "4_1")<br>
2. Deploy the application.
> Command:<br>
> ```bash
> $ kubectl apply -f azure-vote-all-in-one-redis.yaml
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/4_2.png "4_2")<br>
3. Test the application. Monitor progress and output shows a valid public IP address(EXTERNAL-IP) assigned to the service.
> Command:<br>
> ```bash
> $ kubectl get service azure-vote-front --watch
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/4_3.png "4_3")<br>
4. To see the application in action, open a web browser to the public IP address of your service.
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/4_4.png "4_4")<br>

**5 - Scale application**
1. See the number and state of pods in your cluster. A single replica was created. 
> Command:<br>
> ```bash
> $ kubectl get pods
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_1.png "5_1")<br>
2. Manually change the number of pods in `azure-vote-front` deployment, increases the number of front-end pods to 5. Af:
> Command:<br>
> ```bash
> $ kubectl scale --replicas=5 deployment/azure-vote-front
> $ kubectl get pods
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_2.png "5_2")<br>
3. Check the version of your AKS cluster. If your AKS cluster is less than `1.10`, the Metrics Server is not automatically installed. Kubernetes supports `horizontal pod autoscaling` to adjust the number of pods in a deployment depending on CPU utilization or other select metrics.
> Command:<br>
> ```bash
> $ az aks show --resource-group myResourceGroup --name myAKSCluster --query kubernetesVersion --output table
> ``` 
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_3.png "5_3")<br>
4. To use the autoscaler, all containers in your pods and your pods must have CPU requests and limits defined. In the azure-vote-front deployment, the front-end container already requests 0.25 CPU, with a limit of 0.5 CPU.
> Command1:<br>
> ```bash
> $ vi azure-vote-all-in-one-redis.yaml
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_4.png "5_4")<br>
> Command2:<br>
> ```bash
> $ kubectl describe deployment azure-vote-front
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_5.png "5_5")<br>
5. Create a manifest file to define the autoscaler behavior and resource limits.
> Command:<br>
> ```bash
> $ vi azure-vote-hpa.yaml
> ```
> YAML
> ```yaml
> apiVersion: autoscaling/v1
> kind: HorizontalPodAutoscaler
> metadata:
>   name: azure-vote-back-hpa
> spec:
>   maxReplicas: 10 # define max replica count
>   minReplicas: 3  # define min replica count
>   scaleTargetRef:
>     apiVersion: apps/v1
>     kind: Deployment
>     name: azure-vote-back
>   targetCPUUtilizationPercentage: 50 # target CPU utilization
>
> ---
>
> apiVersion: autoscaling/v1
> kind: HorizontalPodAutoscaler
> metadata:
>   name: azure-vote-front-hpa
> spec:
>   maxReplicas: 10 # define max replica count
>   minReplicas: 3  # define min replica count
>   scaleTargetRef:
>     apiVersion: apps/v1
>     kind: Deployment
>     name: azure-vote-front
>   targetCPUUtilizationPercentage: 50 # target CPU utilization
> ```
6. Apply the autoscaler defined in the `azure-vote-hpa.yaml` manifest file. Then see the status of the autoscaler.
> Command:<br>
> ```bash
> $ kubectl apply -f azure-vote-hpa.yaml
> $ kubectl get hpa
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_6.png "5_6")<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_7.png "5_7")<br>
7. Increases the number of nodes to three in the Kubernetes cluster named `myAKSCluster`.
> Command:<br>
> ```bash
> $ kubectl get nodes
> $ az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 3
> $ kubectl get nodes
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_8.png "5_8")<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/5_9.png "5_9")<br>

**6 - Update application** 
1. Open the `config_file.cfg` file with an editor.
> Command:<br>
> ```bash
> $ vi azure-vote/azure-vote/config_file.cfg
> ```
> Change the values for VOTE1VALUE and VOTE2VALUE to different values, such as colors.
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_1.png "6_1")<br>
2. Update the container image. Re-create the application image via Docker Compose.
> Command:<br>
> ```bash
> $ docker-compose up --build -d
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_2.png "6_2")<br>
3. Open a local web brpwser to `http://localhost:8080`. The updated values provided in the `config_file.cfg` file are displayed in your running application.
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_3.png "6_3")<br>
4. Clean up resource.
> Command:<br>
> ```bash
> $ docker-compose down
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_4.png "6_4")<br>
5. Update the image version to `v2`.
> Command:<br>
> ```bash
> $ docker images
> $ docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v2
> $ docker images
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_5.png "6_5")<br>
6. Upload the image to your registry.
> Command:<br>
> ```bash
> $ docker push <acrLoginServer>/azure-vote-front:v2
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_6.png "6_6")<br>
7. Return a list of images that have been pushed to your ACR instance.
> Command:<br>
> ```bash
> $ az acr repository show-tags --name <acrName> --repository azure-vote-front --output table 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_7.png "6_7")<br>
8. Deploy the updated application, specify the `v2` application version. 
> Command:<br>
> ```bash
> $ kubectl get pods
> $ kubectl set image deployment azure-vote-front azure-vote-front=<acrLoginServer>/azure-vote-front:v2
> $ kubectl get pods
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_8.png "6_8")<br>
9. Get the public IP(external IP) address of the `azure-vote-front` service.
> Command:<br>
> ```bash
> $ kubectl get service azure-vote-front
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_9.png "6_9")<br>
10. Open a local web browser to the public IP address of your service.
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenAKSdeployWebApp/blob/main/AKSImages/6_10.png "6_10")<br>

**Clean up the resources**
1. Delete the resource group.
> Command:<br>
> ```bash
> $ az group list -o table
> $ az group delete --name myResourceGroupVM --no-wait --yes  
> ```

## References

1. [AKS - Tutorials](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app)
2. [Kubernetesi - Install and Set Up kubectl](https://v1-18.docs.kubernetes.io/docs/tasks/tools/install-kubectl/) 

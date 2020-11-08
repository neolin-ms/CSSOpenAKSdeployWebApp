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
4. Clean ip resource.
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

**5 - Scale application**
**6 - Update application** 

## References

1. [AKS - Tutorials](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app)
2. [Kubernetesi - Install and Set Up kubectl](https://v1-18.docs.kubernetes.io/docs/tasks/tools/install-kubectl/) 

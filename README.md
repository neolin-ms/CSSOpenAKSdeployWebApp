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
> ![GITHUB](https://github.com/neolin-ms/CSSOpenTerraformAnsibleGithub/blob/main/AKSImages/1_1.png "1_1")<br>
> Command2:<br>
> ```bash
> $ docker images
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenTerraformAnsibleGithub/blob/main/AKSImages/1_2.png "1_2")<br>
> Command3:<br>
> ```bash
> $ docker ps 
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenTerraformAnsibleGithub/blob/main/AKSImages/1_3.png "1_3")<br>
3. Test application locally. To see the running application, enter `http://local:8080` in local web browser. The sample aplication, as shown in the following example.
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenTerraformAnsibleGithub/blob/main/AKSImages/1_4.png "1_4")<br>
4. Clean ip resource.
> Command:<br>
> ```bash
> $ docker-compose down
> ```
> Output:<br>
> ![GITHUB](https://github.com/neolin-ms/CSSOpenTerraformAnsibleGithub/blob/main/AKSImages/1_5.png "1_5")<br>

**2 - Create container registry**
**3 - Create Kubernetes cluster**
**4 - Run application**
**5 - Scale application**
**6 - Update application** 

## References

- [AKS - Tutorials](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app)

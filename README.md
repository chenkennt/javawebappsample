# A Simple Java Web App for Azure

## Build
```shell
mvn package
```

## Run Locally
```shell
mvn jetty:run
```
To run in a different port
```shell
mvn jetty:run -Djetty.port=<your port>
```

## Debug Locally
```shell
set MAVEN_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8000,suspend=n
mvn jetty:run
```

## Containerize Your Web App
1. Build a docker image using `Dockerfile`:
   ```
   docker build -t calculator .
   ```
2. Run docker image locally
   ```
   docker run --rm -p 8080:8080 calculator
   ```
3. Then you can access the web app at http://localhost:8080 in browser

## Deploy to Azure Web App

### Using FTP
Rename the `.war` file in `target` folder to `ROOT.war` and upload it to your Azure Web App through Git or FTP.

### Using Container Image
1. Create a Container Registry on Azure
2. Push your local image to ACR:
   ```
   docker login -u <client id> -p <client secret> <your ACR server>
   docker tag calculator <your ACR server>/calculator
   docker push <your ACR server>/calculator
   ```
3. Create a Web App in Linux on Azure
4. In Docker Container settings of Web App, fill in image name, server URL, username and password of your ACR.
5. Save the changes and you'll be able to access the web app in a few seconds.

## Deploy to Azure Container Service using Kubernetes
1. Create an Azure Container Service using Kubernetes as orchestrator.
2. Install kubectl and connect to ACS using kubectl (refer to [this doc](https://docs.microsoft.com/en-us/azure/container-service/container-service-tutorial-kubernetes-deploy-cluster)).
3. Create a secret for private container registry
   ```
   kubectl create secret docker-registry <secret name> \
     --docker-server=<your ACR server> \
     --docker-username=<client id> \
     --docker-password=<client secret> \
     --docker-email=<your email>
   ```
4. Build docker image and push to ACR (same as step 2 in previous section).
5. Update `image` and `imagePullSecrets` property in `deployment.yaml` with image and secret name you just created.
6. Create a deployment in Kubernetes:
   ```
   kubectl create -f deployment.yaml
   ```
7. Create a service in Kubernetes:
   ```
   kubectl create -f service.yaml
   ```
8. Open Kubernetes to verify the deployment result:
   ```
   kubectl proxy
   ```
   Open http://localhost:8001/ui, go to Services, check calculator service to get external endpoint.
   Open external endpoint to access the web app.

## Setup Continuous Deployment with Azure using Jenkins
General setup:
1. Go to Settings -> Integration & services, click Add service, choose Jenkins (GitHub plugin), fill in Jenkins hook url with `http://<your jenkins server>/github-webhook/`
2. Make sure your Jenkins has the following components installed:
   * JDK
   * Maven
   * Docker

   And the following plugins installed:
   * Azure credentials
   * Docker pipeline
   * Credentials binding
   * Azure app service
3. Create an Azure service principal via [portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal) or [CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json).
4. Open Jenkins, go to Credentials, add a new Microsoft Azure Service Principal using the credential information you just created.
5. If you want to deploy to Kubernetes, install `kubectl` and connect to ACS on your Jenkins instance (please be noted you have to do this using `jenkins` account).

### Deploy using Azure CLI
You can always use Azure CLI to deploy web app to Azure using a Jenkins pipeline, here're a few samples:
1. Azure Web App using [FTP approach](Jenkinsfile_ftp_azcli)
2. Azure Web App using [Container approach](Jenkinsfile_container_azcli).
3. ACS using [Kubernetes](Jenkinsfile_k8s_azcli).

### Deploy using Azure App Service Plugin
With Azure app service plugin, you can deploy to Azure web app more easily.

#### Use freestyle project
1. Create a freestyle project
2. Add a build step to invoke `clean package` maven goals.
3. Add a post-build action 'Publish an Azure Web App', select appropriate credential, resource group name and web app name.
4. To publish through FTP, select "Publish Files", fill in the `.war` file to deploy, source directory (`target`) and target directory (`webapps`). (If you need to deplo to ROOT, you need to add a step yourself to rename your war file to `ROOT.war`.)
5. To publish through container, select "Publish via Docker", fill in Dockerfile path, docker registry URL and registry credentials (you need to create a username/password credential first). (Docker image name is read from your web app configuration, you need to fill it in when creating a new web app or explicitly specify it in docker image in advanced properties.)

#### Use pipeline
Please refer to the sample pipeline of [FTP approach](Jenkinsfile_ftp_plugin) and [container approach](Jenkinsfile_container_plugin).

### Azure Container Registry Tasks with a docker starter template

#### Expectations
* the expecation here is to build and configure an end to end pipeline where whenever we add any changes to our codebase, the changes will be automatically pulled, created an image out of it and saved into the ACR and automatically deploy the same into the Azure web apps.
* have also included an optional workflow with Azure container instances and its better to go through it as well
* the actual codebase represented in this repo is a 'getting started' sample template, as it is meant for a fundamental walk through of the whole configurations rather than the codebase in itself

#### Resources Involved
* Azure Container Registry
* Azure Key Vault
* Azure Service Principles
* Azure Container Instances
* Azure App Service web apps
* GitHub 


#### Follow Along:
* Local Development
  * create the Dockerfile with the required base image
  * build the image with 'docker build -t acr-tasks:1.0
  * run the image as a container to test the same. 'docker run -p 8080:80 -d acr-tasks:1.0'
  * verify that the 'getting started' guide is now available at 'http://localhost:8080/'
  * commit and push the files to git remote

* Setting up the Azure Environment
  * create an Azure Container Registry instance
  * enable the 'admin user' in access keys section
  * build the image and push the same to the acr instance. `az acr build --registry {acrName} --resource-group {resouceGroupName} --image acr-tasks:1.0 .`
  * create a service principle to configure the authentication for the container instances to access the container registry. `az ad sp create-for-rbac --name acr-pull --scopes /subscriptions/{subscriptionId}/resourceGroups/experiments/providers/Microsoft.ContainerRegistry/registries/{acrName} --role acrpull --query password --output tsv`
  * create a key vault instance and store the username (appID) and the password of the created service principle as 2 secrets with the sample secret names ' acrexperiment-pull-user' and ' acrexperiment-pull-password'
  * build the image and create the container out of it. `az container create --name acrtaskscontainer --resource-group experiments --image {acrName}.azurecr.io/acr-tasks:1.0 --registry-login-server {acrName}.azurecr.io --registry-username $(az keyvault secret show --vault-name {keyVaultName} 
--name acrexperiment-pull-user --output tsv --query value) --registry-password $(az keyvault secret show --vault-name {keyVaultName} --name acrexperiment-pull-password --output tsv --query value) --dns-name-label acrtasks --query "{FQDN:ipAddress.fqdn}" --output table`
  * verify that the 'getting started' guide is now available at the domain which was printed as output (eg: acrtasks.westus.azurecontainer.io)

* create a GitHub PAT (Personal Access Token) to allow the ACR taks to authenticate
* create the ACR Task, `az acr task create --registry {acrName} --resource-group {resourceGroupName --name acrtaskgettingstarted --image acr-tasks:1.0 --context https://github.com/{userName}/acr-tasks.git --file Dockerfile --git-access-token {pat}`
* test the created build task, `az acr task run --registry {acrName} --resource-group {resourceGroupName} --name acrtaskgettingstarted`
* now test the full pipeline by perfomring a new commit to the repository, so that an automated build will be triggered

Now we have succesfully configured the ACR Task, so that whenever there we push any update to the GitHub, the changes will be pulled automatically and a new build will be generated out of it and saved, so that we can deploy the same in the next step of continuous deployment stage.

* Continuous Deployment Configurations
  * create an Azure App service webapp with the source as our image in the ACR
  * enable the 'continuous deployment' option in 'deployment center' 
  * ensure that a webhook has been added automatically in our created ACR instance
  * add a new commit in this repository, which would trigger a new task, which in turn would trigger a deployment to our web app
  * once we add a new commit here, ensure that the webhook has been triggered from ACR and also ensure that our new image from ACR has been pulled from the App service web app and was deployed automatically by going through the logs section in 'deployment center' section
  * hit the webapp URL to verify our 'getting started' page is up and running as required

#### Closure
So, we have configured our end to end pipeline where whenever a developer adds a new commit to the GitHub, the ACR Task builds the image and stores it in the ACR automatically and the configured webhook will pull the latest image and deploy the same in to the Azure App services web app. Hence, all a developer now needs to worry about is the code changes, and the rest will be taken care of automatically.

#### References
* ACR Tasks - https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task
* Azure web apps integration - https://azure.github.io/AppService/2021/11/01/how-to-setup-continuous-deployment-using-acr-tasks-with-windows-containers.html

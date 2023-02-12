### Azure Container Registry Tasks with a docker starter template

steps for follow along:
* create the Dockerfile with the required base image
* build the image with 'docker build -t acr-tasks:1.0
* run the image as a container to test the same. 'docker run -p 80:80 -d acr-tasks:1.0'
* verify that the 'getting started' guide is now available at 'http://localhost/'
* commit and push the files to git remote

* setup the Azure Environment
  * create an Azure Container Registry instance
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

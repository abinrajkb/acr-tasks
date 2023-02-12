### Azure Container Registry Tasks with a docker starter template

steps for follow along:
* create the Dockerfile with the required base image
* build the image with 'docker build -t acr-tasks:1.0
* run the image as a container to test the same. 'docker run -p 80:80 -d acr-tasks:1.0'
* verify that the 'getting started' guide is now available at 'http://localhost/'
* commit and push the files to git remote

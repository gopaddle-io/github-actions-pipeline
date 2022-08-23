# github-actions-pipeline
## Sample pipeline code for CI/CD pipelines in gopaddle using GitHub Actions
Gopaddle supports user to srtup the CI/DC pipeline through the Github Actions


  Here we are going to set up the pipeline to build the source code and push the image into the Azure Container Registry. Rolling update the image into the running application in the gopaddle.
  
 ## Pre-requisite

As a pre-requisite, an application must be deployed in gopaddle. Below flow chart gives the step by step process to be followed before creating a Github Actions workflow pipeline.

![](/assets/images/github-actions.png)

Since we are building a pipeline for an application deployed in gopaddle, we must first initialize and deploy an application in gopaddle before we move on to creating the pipeline in Github Actions.

+ Subscribe to gopaddle - If you do not have a gopaddle subscription yet, subscribe to the [gopaddle portal](https://portal.gopaddle.io/signUp)
+ [Provision K8s in gopaddle](https://help.gopaddle.io/en/articles/3942973-registering-a-cloud-account)
+ [Add a Container Registry](https://help.gopaddle.io/en/articles/3942974-adding-a-docker-registry) - Add a Container registry to gopaddle, to push or pull Docker images
+ Clone the project locally - Clone the GitHub project to be containerized. 
+ Initialize and deploy the project using gopaddle
    + [Download and install gpctl](https://help.gopaddle.io/en/articles/5116592-installing-and-configuring-gopaddle-command-line-utility) - Now, from your local desktop, download and install gpctl command line utility.
	+ [Perform gpctl init](https://help.gopaddle.io/en/articles/5056807-initializing-a-microservice-from-scratch) - Auto-generate the Dockerfile and Kubernetes YAML, build docker images, and deploy the application.
	+ capture the .gp file with the resource IDs - Once the application is onboarded using gopaddle, gpctl init creates a .gp file in the project folder which contains the ***apiToken***, ***containerID***, ***serviceID***, ***applicationID***, ***projectID***, ***releaseID*** and the ***distributionID***. Make a note of these IDs, as we will be using these in the Github Actions pipeline script.


## Getting Started

  First we have to create the workflow at the source code repository, for examble we are using spring-petclinic repository in this. fork the repository to your acclount and select the repository. create the **.github/workflows/main.yaml** file. copy the code from the same file from this repository.
  
  
    
  



  First part of the **main.yaml** is to name the Workflow pipeline 
 
 ```
 name: Java CI with Maven
 ```
 
 Then we have to specify when event have to triggered. here we are going to trigger the pipeline every time the source code is updated.
 
 ```
on:
  push:
    branches: [main]
  ```
  
  This part is actually a Actions Part that contains three steps: 
  - **Building the Source Code**
  - **Create the Image and push it into the Azure Registry**
  - **Rolling Update the Image at the gopaddle**

## Building the Source Code

  Create the build job using the follwing command, the job will run on the Ubuntu machine. you can specify the os as per your requirement.
  
```
jobs:
  build:
    runs-on: ubuntu-latest
```    
  Now we are going to add the steps to this job to build the spring-petclinic source code. we are going to check out the repository for workflow to access the source code.
  
```
steps:
  - uses: actions/checkout@v2
```

 Install and setup the Java JDK to build the source code
 
 ```
 - name: Set up JDK 1.8
   uses: actions/setup-java@v1
   with:
    java-version: 1.8
 ```
 
 Build the source code with maven
 ```
 - name: Build with Maven
   run: mvn -B package --file pom.xml
 ```
 Now the build process is complete. next we are going to push the image to registry.
 
 ## Create the Image and push it into the Azure Registry
 
 Add the credentials for your Azure Registry in this repository.
- REGISTRY_LOGIN_SERVER
- REGISTRY_USERNAME
- REGISTRY_PASSWORD

create those secrets in the repository. Goto the **Settings > Secrets > Actions > New secret repository**

![New Secret Repository](/assets/images/githubsecret.png)

Login into AWS Registry

```
- uses: actions/checkout@v2
- uses: docker://ghcr.io/kciter/aws-ecr-action:latest
  with:
    access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    account_id: ${{ secrets.AWS_ACCOUNT_ID }}
    repo: <repo_name>
    region: <region>
    tags: ${{ github.sha }}
    create_repo: true
    image_scanning_configuration: true
  env:
    REPOSITORY: <repo_name>
    REGION: <region>
  run: |
    docker build -t ${{ secrets.AWS_ACCOUNT_ID }}.dkr.$REGION.amazonaws.com/$REPOSITORY:$IMAGE_TAG .
    docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.$REGION.amazonaws.com/$REPOSITORY:$IMAGE_TAG
```

Login into the Google Registry 
```

```

Login into Docker Registry

```
- name: Login to DockerHub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```
	Prepare the docker image using the given Dockerfile. you can change the contents of the Dockerfile as per your project. push the image into the registry

```
 - run: |
    docker build . -t petclinic:${{ github.sha }}
    docker push petclinic:${{ github.sha }}
 ```
        

Login into Azure Container Registry

```
- name: "Build and push image"
  uses: azure/docker-login@v1
  with:
    login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
```

  Prepare the docker image using the given Dockerfile. you can change the contents of the Dockerfile as per your project. push the image into the registry
  
 ```
 - run: |
    docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/petclinic:${{ github.sha }}
    docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/petclinic:${{ github.sha }}
 ```
 
  Get the Commit information for rolling update description.
  
  ```
  - run: echo "DESC=$(echo -n ${{ github.event.head_commit.message }} | base64)" >> $GITHUB_ENV
  ```
  
  ## Rolling Update the Image at the gopaddle
  
  Now we are going to rolling update the application in gopaddle using API.
  we need following information from the gopaddle.
- Container ID
- Service ID
- Application ID
- API Token
- Project ID
- ServiceGroup ID

using the information replace the values in the following API.

```
- name: Deploy Stage
  uses: fjogeleit/http-request-action@master
  with:
    url: "https://portal.gopaddle.io/gateway/v1/{Project ID}/application/{Application ID}"
    method: "PUT"
    bearerToken: ${{ secrets.BEARER_TOKEN }}
    data: '{"updateType":"buildUpdate","deploymentTemplateVersion":"draft","serviceGroups":[{"name":"spring-petclinic","id":"{ServiceGroup ID}","version":"draft","services":[{"id":"{Service ID}","serviceVersion":"draft","releaseConfig":{"image":"${{ secrets.REGISTRY_LOGIN_SERVER }}/petclinic:${{ github.sha }}"}}],"description":"${{ env.DESC }}"}]}'
```



  
  
  
        
 
 
  
 

    
  
  









# github-actions-pipeline
## Sample pipeline code for CI/CD pipelines in gopaddle using GitHub Actions
Gopaddle supports user to srtup the CI/DC pipeline through the Github Actions


Here are some simple steps to follow:

- **Create Github Actions**
- **Register Docker Registry in gopaddle**
- **Create an Image based container in gopaddle**
- **Create necessary artifacts in gopaddle and deploy the service on a Kubernetes environment**
- **Generate gopaddle API token**
- **Update github actions to invoke gopaddle API**
## Create Github Actions

As the first step, we will set the github actions to build a docker container. You can use github secrets to store sensitive information like credentials and tokens. Then create a github workflow using those secrets. We will be using Azure Docker Registry to push the docker image. Hence we will use the **azure/docker-login@v1** plugin in the workflow.

- Let us create the secrets in github repository. Under settings option in github respository, edit Secrets and Add a New respository secret. We will add the Azure credentials in the github secret for the time being. But as we progress through the remaining steps, we will need to add the secrets for gopaddle API token as well. You can follow the steps documented under Azure ACR Registry section here to create an Azure Registry. Here is the list of secrets we need in order to use Azure Registry. You will need these information while registering the Azure Registry in gopaddle as well.
  - **REGISTRY_LOGIN_SERVER**
  - **REGISTRY_USERNAME**
  - **REGISTRY_PASSWORD**
  
  ![](/assets/images/githubsecret.png)
  
  
- Now let us create the .github/workflows folder in the repository. Go to the workflow tab and create the yaml file to build your source code. Your github actions task would look like this to build and push the image to the docker registry. petclinic is the name of the docker image and we will use the github commit ID as the docker image tag. By using the commit ID as the tag, we can relate the docker image to the commit that triggered the build for future debugging.

```
    - name: 'Build and push image'
      uses: azure/docker-login@v1
      with:
        login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - run: |
            docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/petclinic:${{ github.sha }}
            docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/petclinic:${{ github.sha }}
 ```
 
- Execute github actions once, so that a docker image is created and pushed to docker registry. We will be using this image in the next steps.


# Register Docker Registry in gopaddle
We will be using the image created in earlier step to deploy an application in gopaddle. Once an application is deployed, we can perform further rolling updates on that application. We can automate this initial deployment process as well using gopaddle APIs. But for the time being, we will manually deploy an application using gopaddle UI. To create the gopaddle artifacts, Docker Registry has to be registered with gopaddle, so that gopaddle knows the source from which a docker image can be pulled and deployed.

- Register the Azure Registry in gopaddle as described [here](https://help.gopaddle.io/en/articles/3942974-adding-a-docker-registry).

# Create necessary artifacts in gopaddle and deploy the service on a Kubernetes environment

- We need to create 3 resources in gopaddle.
  - **Container:** Create an image based container in gopaddle using the image created in the earlier step as described here. Once the container is created, note down the container ID by viewing the container and extracting the ID from the browser URL.
  - **Service:** Create a Service and add the container to the service as described here. Once the service is created, note down the service ID by viewing the service and extracting the ID from the browser URL.
  - **Deployment Template:** Create a Deployment Template and add the Service to the template.
 

- Create a Kubernetes cluster as described here and deploy the template on the Kubernetes cluster to create a running application.

- Once the application is launched, view the application and gather the application ID from the browser URL.


Using the container ID, service ID and the application ID, we can trigger the rolling update on the specific container within an application as soon as the build is complete.

# Generate gopaddle API token

gopaddle APIs can be securely invoked by using an API token. A user can generate one or more API tokens. A role defines the permissions for a user. A role can be assigned to a user to restrict actions within gopaddle. Let us first create a role, a user and then assign the role to the user. Once the user is created, we will generate an API token.

- In the gopaddle portal, under user profile, navigate to **Teams -> Access Control Lists** and create a new role with access to All Services and All Permissions for the time being.

# gopaddle – create a role with All Permissions

- Under the Users tab, create a user and assign the role.

- Click on User actions and Add a token


# Update github actions to invoke gopaddle API

Now we have the necessary details to invoke gopaddle APIs.

- Container ID
- Service ID
- Application ID
- API Token

  - Add this API token in the github secrets.

using **fjogeleit/http-request-action@master** github plugin to trigger an API call.

Note that, In the above YAML, **BEARER_TOKEN** is the API token, the URL contains the application ID, id under services array is the container ID,        id under serviceGroups is the service ID. Also note that the commit description is encoded in base64 format and then used as the description in          rolling update.

CI/CD pipeline is now ready.

Let us now commit a few changes to the github code and this triggers the workflow.


In the gopaddle portal, you can check the version history under the application. We can see that a new rolling update is triggered.


We can also extend this pipeline by triggering a Jenkins pipeline instead of triggering gopaddle APIs, so that a few integration tests can be run before performing a rolling update. We can use the payload described in the YAML file to trigger gopaddle API from Jenkins pipeline as well. 





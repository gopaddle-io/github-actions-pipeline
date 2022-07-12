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



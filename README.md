# GitHub Actions Workflow for Azure Deployment

This repository contains a GitHub Actions workflow to automate the deployment of a Dockerized application to Azure App Service.

## Workflow Overview

The workflow defined in `workflow.yml` performs the following steps:

1. **Checkout the repository**: Uses the `actions/checkout@v2` action to checkout the repository.
2. **Login to Azure Container Registry**: Uses the `docker/login-action@v1` action to login to the Azure Container Registry.
3. **Build and push Docker image**: Uses the `docker/build-push-action@v2` action to build and push the Docker image to the Azure Container Registry.
4. **Login via Azure CLI**: Uses the `azure/login@v1` action to login to Azure using the Azure CLI.
5. **Deploy to Azure App Service**: Uses the `azure/webapps-deploy@v2` action to deploy the Docker image to Azure App Service.
6. **Restart Azure App Service**: Uses the Azure CLI to restart the Azure App Service.

## Secrets

The following secrets are required for the workflow to run:

- `ACR_USERNAME`: The username for the Azure Container Registry.
- `ACR_PASSWORD`: The password for the Azure Container Registry.
- `AZURE_CREDENTIALS`: The credentials for Azure login.
- `AZURE_WEBAPP_PUBLISH_PROFILE`: The publish profile for the Azure App Service.
- `AZURE_CLIENT_ID`: The client ID for the Azure service principal.
- `AZURE_CLIENT_SECRET`: The client secret for the Azure service principal.
- `AZURE_TENANT_ID`: The tenant ID for the Azure service principal.

## Usage

To use this workflow, ensure that the required secrets are added to your GitHub repository. You can add secrets by navigating to the repository settings and selecting "Secrets".

The workflow will automatically run on each push to the repository. You can also manually trigger the workflow from the GitHub Actions tab.

## Example

Here is an example of the `workflow.yml` file:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Login to Azure Container Registry
        uses: docker/login-action@v1
        with:
          registry: acr-name.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: acr-name.azurecr.io/image-name:v1

      - name: 'Login via Azure CLI'
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: 'Deploy to Azure App Service'
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'app-service-name'
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          images: 'acr-name.azurecr.io/image-name:v1:v1'
      
      - name: Restart Azure App Service
        run: |
          az login --service-principal --username ${{ secrets.AZURE_CLIENT_ID }} --password ${{ secrets.AZURE_CLIENT_SECRET }} --tenant ${{ secrets.AZURE_TENANT_ID }}
          az webapp restart --name app-service-name --resource-group resource-group-name
        shell: bash

License
This project is licensed under the MIT License.
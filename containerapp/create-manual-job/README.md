# Create a manual job

[![containerapp/create-manual-job/README.md](https://github.com/Azure-Samples/java-on-azure-examples/actions/workflows/containerapp_create-manual-job_README_md.yml/badge.svg)](https://github.com/Azure-Samples/java-on-azure-examples/actions/workflows/containerapp_create-manual-job_README_md.yml)

## Prerequisites

<!-- 

  if [[ -z $REGION ]]; then
    export REGION=westus
  fi

  -->
<!-- workflow.cron(0 12 * * 3) -->
<!-- workflow.include(../../acr/helloworldjob/README.md) -->
<!-- workflow.include(../create-environment/README.md) -->

This example assumes you have previously completed the following examples:

1. [Create an Azure Resource Group](../../group/create/README.md)
1. [Create an Azure Container Registry](../../acr/create/README.md)
1. [Build and push a Hello World Job application to Azure Container Registry](../../acr/helloworldjob/README.md)
1. [Create an Azure Container Apps environment](../create-environment/README.md)

## Create the manual job

<!-- workflow.skip() -->
```shell
  export ACA_JOB_NAME=joazca-job-$RANDOM
```

<!-- workflow.run()

if [[ -z $ACA_JOB_NAME ]]; then
  export ACA_JOB_NAME=aca-job-$RANDOM
  sleep 60
fi

-->

To create the manual job use the command line below to create the job.

```shell
  az containerapp job create \
    --name $ACA_JOB_NAME \
    --resource-group $RESOURCE_GROUP \
    --environment $ACA_ENVIRONMENT_NAME \
    --trigger-type Manual \
    --replica-timeout 1800 \
    --replica-retry-limit 1 \
    --replica-completion-count 1 \
    --parallelism 1 \
    --registry-identity system \
    --registry-server $ACR_NAME.azurecr.io \
    --image $ACR_NAME.azurecr.io/$ACR_HELLOWORLDJOB_IMAGE || true
```

<!-- workflow.run()

  sleep 60

 -->

Then create a role assigment so the job can be pulled from your Azure Container Registry:

```shell
  az role assignment create \
    --assignee $(az containerapp job identity show --name $ACA_JOB_NAME --resource-group $RESOURCE_GROUP --query principalId --output tsv) \
    --role acrpull \
    --scope $(az acr show --name $ACR_NAME --query id --output tsv)
```

<!-- workflow.run()

  sleep 240

 -->

And then issue the create command again so the job can be successfully created:

```shell
  az containerapp job create \
    --name $ACA_JOB_NAME \
    --resource-group $RESOURCE_GROUP \
    --environment $ACA_ENVIRONMENT_NAME \
    --trigger-type Manual \
    --replica-timeout 1800 \
    --replica-retry-limit 1 \
    --replica-completion-count 1 \
    --parallelism 1 \
    --registry-identity system \
    --registry-server $ACR_NAME.azurecr.io \
    --image $ACR_NAME.azurecr.io/$ACR_HELLOWORLDJOB_IMAGE 
```

<!-- workflow.directOnly()

  export RESULT=$(az containerapp job show --name $ACA_JOB_NAME --resource-group $RESOURCE_GROUP --output tsv --query properties.provisioningState)
  az group delete --name $RESOURCE_GROUP --yes || true
  if [[ "$RESULT" != Succeeded ]]; then
    echo "Azure Container Apps job $ACA_JOB_NAME was not provisioned properly"
    exit 1
  fi

  -->

## Cleanup

Do NOT forget to remove the resources once you are done running the example.

1m

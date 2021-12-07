# Gettinging started

## Azure CLI

``` bash
# run Azure CLI
docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh mcr.microsoft.com/azure-cli
```

## Login to Azure

``` bash
# Login and follow prompts
az login

# Grab the appropriate tentant id from de response and set it as environment variable

TENANT_ID=<your-tenant-id>

# view and select you subscription account
az account list -o table

#grap the SubscriptionId and set it as environment variable

SUBSCRIPTION=<subscriptionId>

#Set the subscription as Default

az account set --subscription $SUBSCRIPTION
```

## Create Service Principal

Kubernetes needs a service account to manage our Kubernetes cluster

``` bash
SERVICE_PRINCIPAL_JSON=$(az ad sp create-for-rbac --skip-assignment --name aks-hypotheek-sp -o json)

# Keep the `appId` and `password` for later use!

SERVICE_PRINCIPAL=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.appId')
SERVICE_PRINCIPAL_SECRET=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.password')

#note: reset the credential if you have any sinlge or double quote on password
az ad sp credential reset --name "aks-hypotheek-sp"

# Grant contributor role over the subscription to our service principal

az role assignment create --assignee $SERVICE_PRINCIPAL \
--scope "/subscriptions/$SUBSCRIPTION" \
--role Contributor

```
For extra reference you can also take a look at the Microsoft Docs: [here](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/aks/kubernetes-service-principal.md)

## Terraform CLI

Get Terraform

``` bash
# download the cli
curl -o /tmp/terraform.zip -LO https://releases.hashicorp.com/terraform/1.0.11/terraform_1.0.11_linux_amd64.zip

# unzip the cli
unzip /tmp/terraform.zip
chmod +x terraform && mv terraform /usr/local/bin/

cd deploy/azure

```

## Generate SSH Key

``` bash

ssh-keygen -t rsa -b 4096 -N "XzyNUi6Ae6Q1Nh9" -C "p.evers@tjip.com" -q -f  ~/.ssh/id_rsa

SSH_KEY=$(cat ~/.ssh/id_rsa.pub)

```

## Terraform Azure Kubernetes Provider

``` bash
terraform init

terraform plan -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"

terraform apply -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"


terraform refresh -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"

```

## Lets see what we deployed

``` bash
# grab our AKS config

az aks get-credentials -n aks-hypotheek -g aks-hypotheek

# Get kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl

kubectl get svc

```

## Clean up

``` bash
terraform destroy -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"
```
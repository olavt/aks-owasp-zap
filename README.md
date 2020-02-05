# Running OWASP ZAP in Azure Kubernetes Service

## Introduction

This article shows how to run OWASP ZAP security scanning in Azure Kubernetes Service (AKS).

## Prerequsites

- Azure CLI needs to be installed on the local computer in order to issue az command locally. Alternatively you can use the Cloud Shell from the Azure Portal.

## Login to your Azure Account
```
$ az login
```

## List the Azure Subscriptions for your account

```
$ az account list --output table
```

## Select the right subscription

```
$ az account set --subscription "<Subscription Id>"
```

## Create a new Resource Group
```
$ az group create --name OWASPZap --location westeurope
```

## Create an AKS cluster

```
$ az aks create --resource-group OWASPZap --name OWASPZapCluster --node-count 1 --node-vm-size Standard_B2s --generate-ssh-keys --kubernetes-version 1.14.6
```

The above command will create a 1-node cluster using the VM-size "Standard_B2s". To see what VM-sizes are available in a given location issue the following command:

```
$ az vm list-sizes -l westeurope --output table
```

## Install kubectl on your local computer
```
$ az aks install-cli
```

## Configure kubectl to connect to your Kubernetes cluster,
```
$ az aks get-credentials --resource-group OWASPZap --name OWASPZapCluster
```
If you re-run the process, you may need to add the --overwrite-existing option. 

## Verify the connection to your cluster
```
$ kubectl get nodes
```

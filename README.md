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

### Create a Virtual Network

```
$ az network vnet create -g OWASPZap -n ZapNetwork --address-prefix 10.0.0.0/16 \
    --subnet-name aks --subnet-prefix 10.0.0.0/24
```

### Get subnet ID for aks

```
$ az network vnet subnet list \
    --resource-group OWASPZap \
    --vnet-name ZapNetwork \
    --output tsv
```

## Create an AKS cluster

```
$ az aks create \
--resource-group OWASPZap \
--name OWASPZapCluster \
--node-count 1 \
--node-vm-size Standard_B2s \
--network-plugin azure \
--vnet-subnet-id <subnet-id> \
--generate-ssh-keys \
--kubernetes-version 1.15.7
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

## Create a Kubernetes .yaml file for running OWASP ZAP as a CronJOb

Create a file names owasp-zap-cronjob.yaml with the following content:

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: owasp-zap
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: owasp-zap
            image: owasp/zap2docker-weekly
            command: ["/bin/sh", "-c"]
            args: ["quick-scan -sc -o '-config api.disablekey=true' -s xss,sqli http://Your Url"]
          restartPolicy: Never
```

This defines a CronJob to run run a scan every 5 minutes.

Deploy the CronJob to the Kubernetes cluster:

```
$ kubectl apply -f owasp-zap-cronjob.yaml
```


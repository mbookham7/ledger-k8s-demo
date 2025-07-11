# Cloud Deployment Tutorial

Deployment tutorial for setting up a full-stack demo using multi-region cloud deployment.

## Prerequisites

- A multi region CockroachDB cluster running in Kubernetes.
- A local KUBECONFIG with access to all Kubernetes clusters.

## Create CockroachDB Cluster

First create a CockroachDB cluster with a cloud provider of choice.

- There is an example available [here](https://github.com/mbookham7/mb-crdb-multi-region-aks) to create such a cluster in Azure AKS.

- There are step by step instructions for AWS EKS [here](https://www.cockroachlabs.com/docs/stable/orchestrate-cockroachdb-with-kubernetes-multi-cluster.html).

- There are step by step instructions for GCP GKE [here](https://www.cockroachlabs.com/docs/stable/orchestrate-cockroachdb-with-kubernetes-multi-cluster.html?filters=eks).

## Create Ledger Database

First to allow for ease of use of these instructions we need to set some variables. These variables need to set the context of each of the Kubernetes cluster
```
clus1="mb-crdb-mr-k8s-uksouth"
clus2="mb-crdb-mr-k8s-ukwest"
clus3="mb-crdb-mr-k8s-northeurope"
loc1="uksouth"
loc2="ukwest"
loc3="northeurope"
```

Once you have a multi region CockroachDB cluster, connect to the cluster and create a database called `ledger`

To do this `exec` into the pod using the CockraochDB SQL client.
```
kubectl exec -it cockroachdb-client-secure -n $loc1 --context $clus1 -- ./cockroach sql --certs-dir=/cockroach-certs --host=cockroachdb-public
```
Create a user, make that user an admin and create the `ledger` database.
```
CREATE USER craig WITH PASSWORD 'cockroach';
GRANT admin TO craig;
CREATE DATABASE ledger; 
\q
```

### Deploy Ledger

Bank Server needs to be deployed into each region. This is done by applying a simple Kubernetes manifest that contains a deployment and a service.
To make our lives easier lets set our Kubernetes contexts as environment variables.
> If you followed my AKS guide and used the same regions then you can use the values below. If you didn't make sure you update them to the correct values for your environment.
```
clus1="mb-crdb-mr-k8s-uksouth"
clus2="mb-crdb-mr-k8s-ukwest"
clus3="mb-crdb-mr-k8s-northeurope"
loc1="uksouth"
loc2="ukwest"
loc3="northeurope"
```

Create a new namespace for Ledger to be deployed into in each region.
```
kubectl create namespace ledger --context $clus1
kubectl create namespace ledger --context $clus2
kubectl create namespace ledger --context $clus3
```

First you need to create an account plan and setup multiregion for the ledger database. The command are help in a text file which is mounted as a configmap and passed as a arg at runtime.
```
kubectl apply -f scalability/client-configmap-account-plan.yaml -n ledger --context $clus1
kubectl apply -f ./manifest/ledger-account-plan-job.yaml -n ledger --context $clus1
```

To monitor to progress we can tail the logs of the pod. Once you see this entry below the pod can be removed.
```
```

This command grabs the name of name of the pod and store it as an environment variable the passes it in the command below to tail the log.
```
export POD_NAME=$(kubectl get pods -n ledger --context $clus1 --selector=run=ledger-account-plan -o jsonpath="{.items[0].metadata.name}")
kubectl logs -f --tail 100 $POD_NAME -n ledger --context $clus1
```

Once you see `Setting survival goal: REGION` then you should be good to go. You can delete the pod.
```
kubectl delete -f ./manifest/ledger-account-plan-job.yaml -n ledger --context $clus1
```

Now that the database has been setup with the account plan and multi-region we can start the application in each region. Again the configuration for the app is held in a text file passed at runtime. So let create the configmap.

Apply the config map.
```
kubectl apply -f scalability/ledger-config.yaml -n ledger --context $clus1
kubectl apply -f scalability/ledger-config.yaml -n ledger --context $clus2
kubectl apply -f scalability/ledger-config.yaml -n ledger --context $clus3
```

Deploy Ledger into each of the clusters and create a service to expose this to the outside world.
```
kubectl apply -f ./manifest/uksouth-deployment.yaml -n ledger --context $clus1
kubectl apply -f ./manifest/ukwest-deployment.yaml -n ledger --context $clus2
kubectl apply -f ./manifest/northeurope-deployment.yaml -n ledger --context $clus3
```

Check each cluster to ensure the pods are running.
```
kubectl get po -n ledger --context $clus1
kubectl get po -n ledger --context $clus2
kubectl get po -n ledger --context $clus3
```

If all the pods are running and the service has been able to create a load balancer successfully then you can move to the next stage.

## Demo Instructions

Once the cluster is setup and the bank is deployed you can access the UI via its external IP `http://EXTERNALIP:9090`. You will see your workload running.

## Useful Commands

Delete deployment..
```
kubectl delete -f ./manifest/uksouth-deployment.yaml -n ledger --context $clus1
kubectl delete -f ./manifest/ukwest-deployment.yaml -n ledger --context $clus2
kubectl delete -f ./manifest/northeurope-deployment.yaml -n ledger --context $clus3
```

Delete account plan job and configmap
```
kubectl delete -f scalability/client-configmap-account-plan.yaml -n ledger --context $clus1
kubectl delete -f ./manifest/ledger-account-plan-job.yaml -n ledger --context $clus1
```
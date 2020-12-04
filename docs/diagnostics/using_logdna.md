---
layout: default
title: Using LogDNA with IBM Blockchain Platform
parent: Diagnostics
---

# Using LogDNA with IBM Blockchain Platform

Using LogDNA to access logs from an IBP Kubernetes cluster can help troubleshoot problems.

Here's how!


## You will need

- An existing [IBP Blockchain Platfrom](https://cloud.ibm.com/catalog/services/blockchain-platform) service

- The [IBM Cloud CLI and Kubernetes CLI plug-in](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli)

- The [jq](https://stedolan.github.io/jq/) command-line JSON processor

Make sure you're logged in using `ibmcloud login -a cloud.ibm.com` (you may need to use the `--sso` option)


## Set up the LogDNA service

- Create an [IBM Log Analysis with LogDNA](https://cloud.ibm.com/catalog/services/ibm-log-analysis-with-logdna) service

- Get the ingestion key (you can view this in the UI, or use the commands below)

Start by finding the LogDNA service you just created:

```
ibmcloud resource service-instances
```

Use the LogDNA service name to create an environment for the ingestion key which we'll use later:

```
export LOGDNA_KEY=$(ibmcloud resource service-keys --instance-name "IBM Log Analysis with LogDNA-demo" --output json | jq -r '.[0].credentials.ingestion_key')
```

## Configure the IBP Kubernetes cluster

- Use your LogDNA service ingestion key to [configure your Kubernetes cluster to send logs to your LogDNA instance](https://cloud.ibm.com/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-kube)

Start by finding your IBP Kubernetes cluster:

```
ibmcloud ks cluster ls
```

Use the cluster ID to download the Kubernetes configuration files and certificates required for kubectl commands to connect to your IBP cluster:

```
ibmcloud ks cluster config --cluster bv56eljd05dlt2gm5lng
```

Verify that the kubeconfig file was updated successfully:

```
kubectl get nodes
```

**Note:** The following commands are also shown on the "Add agents to desired log sources" page of the "IBM Log Analysis with LogDNA" service.

Create a new namespace:

```
kubectl create namespace ibm-observe
```

Create a Kubernetes secret to store your LogDNA ingestion key:

```
kubectl create secret generic logdna-agent-key -n ibm-observe --from-literal=logdna-agent-key=$LOGDNA_KEY
```

Create a Kubernetes daemon set to deploy the LogDNA agent on every worker node of your Kubernetes cluster:

```
kubectl create -f https://assets.us-south.logging.cloud.ibm.com/clients/agent-resources.yaml
```

Verify that the LogDNA agent is deployed successfully

```
kubectl get pods -n ibm-observe
```

Now you're ready to start [viewing IBM Blockchain Platform logs](https://cloud.ibm.com/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-view_logs#view_logs)!

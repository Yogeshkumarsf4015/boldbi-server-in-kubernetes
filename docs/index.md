# Deployment Prerequisites

* [Install Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) to deploy Bold BI using kubectl.
* [File Storage](pre-requisites.md#file-storage)
* [Create and connect a cluster](pre-requisites.md#create-a-cluster)
* [Load Balancing](pre-requisites.md#load-balancing)

  

# Deploy Bold BI using kubectl

[Bold BI](https://www.boldbi.com/) can be deployed manually on Kubernetes cluster. You can create Kubernetes cluster on cloud cluster providers(GKE,AKS and EKS). After completing cluster creation, connect to it and you can download the configuration files [here](../deploy/). This directory includes configuration YAML files, which contains all the configuration settings needed to deploy Bold BI on Kubernetes cluster. The following links explain Bold BI Kubernetes deployment in a specific cloud environments.
    
* [Google Kubernetes Engine (GKE)](google-gke.md)
* [Amazon Elastic Kubernetes Service (EKS)](amazon-eks.md)
* [Azure Kubernetes Service (AKS)](microsoft-aks.md)
* [Oracle Kubernetes Engine (OKE)](oracle-oke.md)


# Enable Upgrade Center

After deploying Bold BI, you can optionally enable the **Upgrade Center** feature to manage in-application upgrades directly from the Bold BI administration panel. Refer to the [Upgrade Center configuration guide](upgrade-center.md) for deployment steps using kubectl or Helm.

# Upgrade Bold BI

If you are upgrading Bold BI to latest version, please follow the steps in this [link](upgrade.md).

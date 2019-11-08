**Important Note:** HobbyFarm will tear down your provisioned resources within 5 minutes if your laptop goes to sleep or you navigate off of the HobbyFarm page. Please ensure that you do not do this, for example, during lunch or you will need to restart your scenario.

# Introduction

The goal of this training module is help to developers to onboard their apps on Rancher. This modules defines some agile development concepts and practical exercises to be used as a basic example to onboard apps on Rancher.

## Requirements
    
- Github user
- Docker hub user
- git, docker and make installed 
- kubectl and helm installed

## Lab preparation

### Rancher server

In this step, we will be deploying Rancher as a single-node container. Note that this is not a suitable deployment model for anything near production, and is entirely designed for running in Lab scenarios (which this happens to be).

Rancher runs as a single Docker container in this scenario, with an internal, embedded etcd and Kubernetes API Server.

To get started, execute the below docker command on `Rancher01`

```ctr:Rancher01
sudo docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /opt/rancher:/var/lib/rancher \
rancher/rancher:stable
```

Rancher may not immediately be available at the link below, as it may be starting up still. Please continue to refresh until Rancher is available.

1. Access Rancher Server at <a href="https://${vminfo:Rancher01:public_ip}" target="_blank">https://${vminfo:Rancher01:public_ip}</a>
2. Enter a password for the default `admin` user when prompted.
3. When prompted, the **Rancher Server URL** should be `${vminfo:Rancher01:public_ip}`, which is the IP you used to access the server.

### K8s cluster

In this step, we will be creating a Kubernetes Lab environment within Rancher. Normally, in a production case, you would create a Kubernetes Cluster with multiple nodes with k8s sepparated planes; however, with this lab environment, we will only be using 3 virtual machines for the cluster with overlapped planes.

1. Hover over the top left dropdown, then click **Global**
1. Click **Add Cluster**
    - *The current context is shown in the upper left, and should say 'Global'*
    - Note the multiple types of Kubernetes cluster Rancher supports. We will be using **Custom** for this lab, but there are a lot of possibilities with Rancher.
1. Click on the **Custom** Cluster box
1. Enter a name in the **Cluster Name** box and click **Next** at the bottom.
1. Make sure the boxes **etcd**, **Control Plane**, and **Worker** are all ticked.
1. Click **Show advanced options** to the bottom right of the **Worker** checkbox
1. Repeat these steps for `Cluster01`, `Cluster02` and `Cluster03`
    1. Browse back to HobbyFarm (where you are reading these steps from) and click the `Cluster01` tab. Note the bar of VM Information, including **Public Address** and **Private Address**. Make sure that **Public Address* is NOT the address that you are accessing Rancher with.
    1. Enter the **Public Address** (`${vminfo:Cluster01:public_ip}`) and **Internal Address** (`${vminfo:Cluster01:private_ip}`) from the `Cluster01` virtual machine. This information can also be found at the top of the `cluster01` tab.
        - **IMPORTANT:** It is VERY important that you use the correct External and Internal addresses from the **Cluster01** machine for this step, and run it on the correct machine. Failure to do this will cause the future steps to fail.
    1. Click the clipboard to **Copy to Clipboard** the `docker run` command
    1. Take the copied docker command and run it on `Cluster01`
1. Click **Done**
1. Within the Rancher UI click on `<YOUR_CLUSTER_NAME>` which is the name you entered during cluster creation.
1. You can watch the state of the cluster as your Kubernetes node `Cluster01` registers with Rancher here as well as the **Nodes** tab
1. Your cluster state on the Global page will change to **Active**
1. Once your cluster has gone to **Active** you can click on it and start exploring.
    - *You can find the docker command again by editing the cluster in the UI if needed*

## CI/CD/CD concepts
  
Let's start whith basic Continuous Integration, Continuous Delivery and Continuous Deployment (CI/CD/CD) introduction. CI/CD/CD is a concept which is implemented as a pipeline that facilitates the lifecycle management of an app, from committing new generated code to deploy new version of it. 

CI/CD/CD is essential to software development using Agile methodologies which recommends the use of automated testing to get working software into the hands of real users as quickly as possible. This allows stakeholders and users to access newly created features and provide feedback as soon as possible, so features can be iteratively improved upon.

### CI: defines what to deploy for every app version

Continuous Integration pipeline stage, at least, should define these steps:
- how to build a new version of the app, when new code is commited
- how to test new builded version of the app
- how to package a new tested version of the app
- how to publish a new packaged version of the app

### CD: defines how to deploy/upgrade every app version builded

 Continuous Delivery pipeline stage at least, should define these processes:
- how to deploy/upgrade a new published version of the app
- how to test new deployment version of the app
- how to publish a new tested deployment version of the app

### CD: defines where to deploy/upgrade every app version delivered

 Continuous Deployment pipeline stage at least, should define these processes:
- where to deploy/upgrade a new deployment version of the app
- how to to test new deployment version of the app


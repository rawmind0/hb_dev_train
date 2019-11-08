# Lab preparation

Before start, we need to prepare the lab for proper working.

The lab provides 4 vm's: 
- 1 vm for rancher server, `Rancher01`.
- 3 vm's for k8s cluster, `Cluster01`, `Cluster02`, `Cluster03`.

## Rancher server

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

## K8s cluster

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

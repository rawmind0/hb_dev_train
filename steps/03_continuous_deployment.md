# Continuous Deployment

On this chapter we are going to start defining the CD steps needed for our app.Basically we need to define scenario to where to deploy or upgrade our app. 

It's needed to define: 

## Rancher

Rancher is used to manage k8s clusters and workloads deployments.

### Workloads (manifest files)

To deploy and manage workloads lifecycle, we need k8s manifest files that we got previous stage. 

#### Deploying workload

To deploy it, first configure kubectl. Get kubectl config file from Rancher UI and export KUBECONFIG=&lt;kubeconfig_file&gt; env variable or execute command from the kubectl windows from rancher UI.

```
# DOCKER_USER=&lt;docker_user&gt; make k8s_deployment
# kubectl apply -f k8s/hello-world-app-deployment.yaml
```

### Apps (Helm charts)

Installed helm charts are called `apps` on Rancher. Helm repos are called `catalogs` on Rancher.

#### Adding catalog

Our helm repo needs to be added on Rancher to manage our app

1. Access Rancher Server at <a href="https://${vminfo:Rancher01:public_ip}" target="_blank">https://${vminfo:Rancher01:public_ip}</a>
1. Hover over the top left dropdown, then click **Global**
1. Hover over the top **Tools** dropdown and click **Catalogs**
1. Click **Add Catalog**
1. Enter a name in the **Name** box, a helm repo url `https://github.com/&lt;user&gt;/hello-helm` in the **Catalog URL** box and click **Create** at the bottom.

It can also be done using Rancher CLI

```
# rancher catalog add my-repo https://github.com/&lt;user&gt;/hello-helm
```

#### Deploying app

Once the catalog is added to Rancher, our app could be deployed from it.

1. Hover over the top left dropdown, **`&lt;YOUR_CLUSTER_NAME&gt;`** then click **`Default`** project
1. Click **Apps** on the top menu
1. Click **Launch** 

It can also be done using Rancher CLI

```
# rancher app install hello-world-app hello-world-app
```

#### Updating app

1. Hover over the top left dropdown, **`&lt;YOUR_CLUSTER_NAME&gt;`** then click **`Default`** project
1. Click **Apps** on the top menu
1. Select `hello-world-app` app and click upgrade

It can also be done using Rancher CLI

```
# rancher app upgrade hello-world-app 0.0.2
```

### Creating pipeline

We are going to use rancher pipeline facility to configure our full pipeline.

1. Hover over the top left dropdown, **`&lt;YOUR_CLUSTER_NAME&gt;`** then click **`Default`** project
1. Hover over the top **Tools** dropdown and click **Pipeline**
1. Configure github integration following onsite instructions

Once pipeline is configured click on **configure repositories** button and enable your repo.

To see and manage pipeline go to

1. Hover over the top left dropdown, **`&lt;YOUR_CLUSTER_NAME&gt;`** then click **`Default`** project
1. Hover over the top **Resources** dropdown and click **Pipeline**

Pipeline is defined in the repo, https://github.com/rawmind0/hello-world-app/blob/master/.rancher-pipeline.yml

The pipeline should run on every time we pushed a tag in the git repo 



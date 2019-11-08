# Continuous Deployment

## Rancher

### Apps (helm chart)

Installed helm charts are called `apps` on Rancher. Helm repos are called `catalogs` on Rancher.

#### Adding catalog

Our helm repo needs to be added on Rancher to manage our app

1. Access Rancher Server at <a href="https://${vminfo:Rancher01:public_ip}" target="_blank">https://${vminfo:Rancher01:public_ip}</a>
1. Hover over the top left dropdown, then click **Global**
1. Hover over the top **Tools** dropdown and click **Catalogs**
1. Click **Add Catalog**
1. Enter a name in the **Name** box, a helm repo url `https://github.com/<user>/hello-helm` in the **Catalog URL** box and click **Create** at the bottom.

It can also be done using Rancher CLI

```
# rancher catalog add my-repo https://github.com/&lt;user&gt;/hello-helm
```

#### Deploying app

Once the catalog is added to Rancher, our app could be deployed from it.

1. Hover over the top left dropdown, **`<YOUR_CLUSTER_NAME>`** then click **`Default`** project
1. Click **Apps** on the top menu
1. Click **Launch** 

It can also be done using Rancher CLI

```
# rancher app install hello-world-app hello-world-app
```

#### Updating app

1. Hover over the top left dropdown, **`<YOUR_CLUSTER_NAME>`** then click **`Default`** project
1. Click **Apps** on the top menu
1. Select `hello-world-app` app and click upgrade

It can also be done using Rancher CLI

```
# rancher app upgrade hello-world-app 0.0.2
```

### Creating pipeline

We are going to use rancher pipeline facility to configure our full pipeline.

1. Hover over the top left dropdown, **`<YOUR_CLUSTER_NAME>`** then click **`Default`** project
1. Hover over the top **Tools** dropdown and click **Pipeline**
1. Configure github integration following onsite instructions

Once pipeline is configured click on **configure repositories** button and enable your repo.

To see and manage pipeline go to

1. Hover over the top left dropdown, **`<YOUR_CLUSTER_NAME>`** then click **`Default`** project
1. Hover over the top **Resources** dropdown and click **Pipeline**

Pipeline is defined in the repo, https://github.com/rawmind0/hello-world-app/blob/master/.rancher-pipeline.yml

The pipeline should run on every time we pushed a tag in the git repo 

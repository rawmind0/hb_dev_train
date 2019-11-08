# Continuous Deployment

On this chapter we are going to start defining the CD steps needed for our app.Basically we need to define scenario to where to deploy or upgrade our app. 

## Rancher

Rancher is used to manage k8s clusters and workloads deployments.

### Workloads (k8s manifests)

To deploy and manage workloads lifecycle, we need k8s manifest files that we got previous stage and a k8s cluster where deploy to.

#### Deploying workload

To deploy it, first configure kubectl. Get kubectl config file from Rancher UI and export KUBECONFIG=&lt;kubeconfig_file&gt; env variable or execute command from the kubectl windows from rancher UI.

```
# DOCKER_USER=&lt;docker_user&gt; make k8s_deployment
# kubectl apply -f k8s/hello-world-app-deployment.yaml
```

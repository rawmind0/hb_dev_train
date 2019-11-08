# Continuous Delivery

On this chapter we are going to start defining the CD steps needed for our app.Basically we need to generate manifest files to define how to deploy or upgrade our app. 

It's needed to define: 
- k8s manifests: how to deploy our app from k8s
- helm chart: how to deploy our app from helm repo 
- upgrade strategy: how to upgrade our app with a new release

If you don't have kubectl nor helm running in your computer, please install it into one of the ClusterX vm

* helm: `curl -LO https://get.helm.sh/helm-v2.16.0-linux-amd64.tar.gz`
* kubectl: `curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl`
  
## Kubernetes manifests

It's time to define our app k8s manifest files to be able to deploy it into our k8s cluster.

### Upgrade strategy

App upgrade will be managed by k8s. Depending of the app need, we can choose distinct [upgrade strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)

Basically there are 2 strategies: 
- Recreate: All existing Pods are killed before new ones are created when `.spec.strategy.type==Recreate`.

- Rolling Update: The Deployment updates Pods in a rolling update fashion when `.spec.strategy.type==RollingUpdate`. You can specify [`maxUnavailable`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) and [`maxSurge`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) to control the rolling update process.

We are choosing rolling update.

```
...
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
...
```

In addition to rolling upgrade strategy, another topic to take care is the number of the [replicas](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#replicas) of the app. It's important to define at least replica=2 to get rolling upgrade with zero downtime on stateless services.


### deployment file

We are going to use a [k8s deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to run our app on a k8s cluster.

```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    app: hello-world-app
  name: hello-world-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: hello-world-app
    spec:
      containers:
      - image: rawmind/hello-world-app
        imagePullPolicy: Always
        name: hello-world-app
        ports:
        - name: http-port
          containerPort: 8080
          protocol: TCP
        env:
        - name: MY_NODE_IP
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: status.hostIP
        livenessProbe:
          tcpSocket:
            port: http-port
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /
            port: http-port
          initialDelaySeconds: 15
          periodSeconds: 20
```

To deploy it, first configure kubectl. Get kubectl config file and export KUBECONFIG=&lt;kubeconfig_file&gt; env variable or execute command from the kubectl windows from rancher UI.

```
# DOCKER_USER=&lt;docker_user&gt; make k8s_deployment
# kubectl apply -f k8s/hello-world-app-deployment.yaml
```

### service file

Once our deployment is defined, we need to expose the app to others. We need to define a service.

```
apiVersion: v1
kind: Service
metadata:
  name: hello-world-app-service
  namespace: default
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-world-app
  type: NodePort
```
To deploy it

```
# DOCKER_USER=&lt;docker_user&gt; make k8s_service
# kubectl apply -f k8s/hello-world-app-service.yaml
```

### ingress file

Optionally, may be we want to expose the app to others using http/https. We could define a ingress.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-app-ingress
spec:
  rules:
  - host: hello-world-app.test
    http:
      paths:
      - backend:
          serviceName: hello-world-app-service
          servicePort: http
        path: /
```

To deploy it

```
# DOCKER_USER=&lt;docker_user&gt; make k8s_ingress
# kubectl apply -f k8s/hello-world-app-ingress.yaml
```

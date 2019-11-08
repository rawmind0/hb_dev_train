# Continuous Delivery

## Helm chart

Additionally to generate k8s manifests, helm chart could be used to share and deploy our app.

### Creating repo

First we need to create an empy github repo from the web browser. For the practice we have created https://github.com/rawmind0/hello-helm

Create your own repo using your github user, https://github.com/&lt;user&gt;/hello-helm We are going to use `master` branch for charts definitions and `gh-pages` branch for helm index file.

```
# cd hello-helm
# echo "Hello helm" > README.md
# mkdir charts
# git add README.md charts
# git commit -m "Initial commit"
# git push origin master
# git checkout -b gh-pages
# git push origin gh-pages
```

For use github repo as helm chart you should follow [this steps](https://helm.sh/docs/chart_repository/#github-pages-example) to configure github pages to serve helm index files from `gh-pages` branch. At the end helm repo should be avilable by https://rawmind0.github.io/hello-helm/

### Creating app chart

Once the repo is created, let's go to create our app chart.

```
# mkdir -p charts/hello-world-app
# helm create hello-world-app    # Create chart folder for the app
# mv hello-world-app charts/hello-world-app/${VERSION}
```

Enter hello-world-app folder and edit files to configure our app deployment, service and ingress. Take a look to https://github.com/rawmind0/hello-helm/tree/master/hello-world-app 

Once chart is edited commit changes to git repo

```
# git add charts/hello-world-app
# git commit -m "Initial commit"
# git push origin master
```

Once chart is pushed to `master`, we need to generate helm chart template and index, and push them to `gh-pages` branch

```
# helm package charts/hello-world-app/${VERSION}
# git checkout gh-pages
# helm repo index --url https://&lt;user&gt;.github.io/hello-helm/ .
# git add hello-world-app-${VERSION}.tgz index.yaml
# git commit -m "Added helm index file"
# git push origin gh-pages
```

## Testing deployment

For testing, generate k8s manifests files using helm.

```
# helm template hello-world-app > hello-world-app-helm-template.yaml
# kubectl apply -f hello-world-app-helm-template.yaml
```

## Creating pipeline

Minimum steps needed to CD pipeline are already seen. A pipeline is needed to be able to automate all the steps and make them reproducible. 

A Makefile is prepared on the app repo to define and execute all stages. Take a look at [Makefile](https://github.com/rawmind0/hello-world-app/blob/master/Makefile)

Execute make command to check that all steps are executed automatically. kubectl should be configured in order deploy works fine.

```
# DOCKER_USER=&lt;docker_user&gt; k8s_deploy  # Should generate k8s manifest and deploy
```

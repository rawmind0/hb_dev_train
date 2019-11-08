# Continuous Integration

## Publishing app
  
Once the image is builded, it should be scanned and published to get it available for others.

### Pushing app image

To publish the image a docker registry is needed. We are going to use docker hub in the practice. Login to docker hub and generate a [personal token](https://www.docker.com/blog/docker-hub-new-personal-access-tokens/) to be used during the practice, 

Login into docker hub and push the image to the registry

```
# docker login -u &lt;docker_user&gt; -p &lt;token&gt;
# docker push &lt;docker_user&gt;/hello-world:dev
```

## Creating pipeline

Minimum steps needed to CI pipeline are already seen. A pipeline is needed to be able to automate all the steps and make them reproducible. 

A Makefile is prepared on the app repo to define and execute all stages. Take a look at [Makefile](https://github.com/rawmind0/hello-world-app/blob/master/Makefile)

Execute make command to check that all steps are executed automatically.

```
# DOCKER_USER=&lt;docker_user&gt; make app  # Should build, package and test the code
# DOCKER_USER=&lt;docker_user&gt; DOCKER_PASS=&lt;token&gt; make publish  # Should test and publish the image
```

It's important here pay attention on the version you are getting. It should be something like &lt;commit_id&gt;-dirty. It's more than recommended to set a git tag before publish a new version of the app, to match docker, app and git tags.

Set a git tag, build and publish a new app docker using make

```
# git tag 0.0.1
# git push origin master --tags
# DOCKER_USER=&lt;docker_user&gt; make app
# DOCKER_USER=<docker_user> DOCKER_PASS=&lt;token&gt; make publish
```

If all is working fine, `rawmind/hello-world:0.0.1` should be builded, tested and published.

If we want to automate it, on every git tag we need to execute

```
# DOCKER_USER=&lt;docker_user&gt; make app
# DOCKER_USER=<docker_user> DOCKER_PASS=&lt;token&gt; make publish
```
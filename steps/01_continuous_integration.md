# Continuous Integration

On this chapter we are going to start defining the CI steps needed for an example app. The app is on [hello-world-app repo](https://github.com/rawmind0/hello-world-app). It's a simple web service coded in golang saying hello.

First of all, please fork the github repo [hello-world-app repo](https://github.com/rawmind0/hello-world-app) to you gihtub user from the web browser. [How to fork a repo](https://help.github.com/en/github/getting-started-with-github/fork-a-repo#fork-an-example-repository)

Once github repo is forked, please clone your repo and go to the repo root folder.

```
# git clone https://github.com/&lt;user&gt;/hello-world-app
# cd hello-world-app
```

## Dockerizing app
  
Dockerize the app is the first step needed at this stage. This step needs  to accoplish 2 goals, build the app and package it into a container for a runtime portability.

Docker images are used for both, building (builder) and execute (executor) the app. 

### building docker image

A Dockerfile is needed to build a docker image: 
* It's the script used to build a Docker image. 
* Every Dockerfile has a FROM section, to specify what the "base" image is. 
* Many popular applications (ex: mongodb, mysql, apache, etc) already have Docker images published on the Docker Hub. Any image can be used as the base image (using FROM) and customized for user needs.We could copy files, install software, add users, expose port, define entrypoint,...
* ENTRYPOINT and CMD, define default execution command when the container runs.

At this point we can choose between 2 strategies: 
* single-stage building: build and package the app in one single docker. Builder and executor are the same image.
* multi-stage building: build the app in one docker and package it in another docker. Builder and executor are distinct images. This is the recommended option.

#### layers

One of the best practices in a container build is to do it as smaller as possible. To get it layers should be understood. 

* Every single command in a Dockerfile generates a layer.
* Layers are assembled by runtime in sequence to created a "flattened" view of the filesystem
* Deleting a file in a layer creates a mask over the file but does not remove it.

```
FROM ubuntu:18.04 
COPY . /app 
RUN apt-get -y install golang
RUN /app/build/build.sh
RUN apt-get –y remove golang-go
RUN chmod a+x /app/bin/myapp
ENTRYPOINT ["/app/bin/myapp"]
```

* To really delete files should be done in the same layer.

```
FROM ubuntu:18.04 
COPY . /app 
RUN apt-get -y install golang && \
    /app/build/build.sh && \
    apt-get –y remove golang-go && \
    chmod a+x /app/bin/myapp
ENTRYPOINT ["/app/bin/myapp"]
```

#### single-stage building
      
This method is the simplest one. A Dockerfile is created with info about how to build the app and how to execute it in one docker. 

This requires basic steps:
- install build tools and dependencies.
- build the app. 
- copy the binary to a concrete location
- cleanup build tools and dependencies to get docker as smaller as possible.

This is the Dockerfile for our example app.

```
# Set FROM image as starting point
FROM rawmind/alpine-base:3.10-1

# Set some maintainer info
MAINTAINER Raul Sanchez &lt;raul@rancher.com&gt;

#Set arguments for building
ARG SERVICE_NAME=hello-world-app
ARG SERVICE_HOME=/opt/hello-world-app
ARG SERVICE_REPO=rawmind0
ARG SERVICE_VERSION=dev

#Set environment
ENV SERVICE_NAME=${SERVICE_NAME} \
    SERVICE_HOME=${SERVICE_HOME} \
    SERVICE_USER=rancher \
    SERVICE_UID=10012 \
    SERVICE_GROUP=rancher \
    SERVICE_GID=10012 \
    VERSION=${SERVICE_VERSION} \
    GOMAXPROCS=2 \
    GOROOT=/usr/lib/go \
    GOPATH=/opt/src \
    GOBIN=/gopath/bin
    PATH=${PATH}:${SERVICE_HOME}

# Add files
ADD . /opt/src/src/github.com/rawmind0/hello-world-app/

# Build the app
RUN apk add --no-cache git mercurial bzr make go musl-dev && \
    cd /opt/src/src/github.com/rawmind0/hello-world-app && \
    make test && \
    make build && \
    mkdir ${SERVICE_HOME} && \
    cp -Rp ${SERVICE_NAME} img/* ${SERVICE_HOME}/ && \
    cd ${SERVICE_HOME} && \ 
    rm -rf /opt/src /gopath && \
    apk del --no-cache git mercurial bzr make go musl-dev && \
    addgroup -g ${SERVICE_GID} ${SERVICE_GROUP} && \
    adduser -g "${SERVICE_NAME} user" -D -h ${SERVICE_HOME} -G ${SERVICE_GROUP} -s /sbin/nologin -u ${SERVICE_UID} ${SERVICE_USER} && \
    chown -R ${SERVICE_USER}:${SERVICE_GROUP} ${SERVICE_HOME} && \
    setcap 'cap_net_bind_service=+ep' ${SERVICE_HOME}/${SERVICE_NAME}

# Set user to execute the app
USER $SERVICE_USER

# Set app workdir
WORKDIR $SERVICE_HOME

# Set app entrypoint
ENTRYPOINT ["/opt/hello-world-app/hello-world-app"]
```

To test it, execute the following

```
# docker build -f package/Dockerfile .
```

#### multi-stage building

This method is cleaner than the single-stage. A Dockerfile is created with info about how to build the app in a builder docker and how to copy and execute it in an executor docker. 

This requires basic steps:
- define builder image and install build tools and dependencies, golang in our example.
- build the app into the builder docker.
- define executor image and copy the binary to a concrete location

This would be a Dockerfile for our example app.

```
# Set FROM image as starting point for builder
FROM golang:1.12.13-alpine as builder

#Set arguments for building
ARG SERVICE_NAME=hello-world-app
ARG SERVICE_HOME=/opt/${SERVICE_NAME}
ARG SERVICE_REPO=rawmind0
ARG SERVICE_SRC=/go/src/github.com/${SERVICE_REPO}/${SERVICE_NAME}
ARG SERVICE_VERSION=dev

ENV VERSION ${SERVICE_VERSION}
WORKDIR $SERVICE_SRC/
COPY . .
RUN env && \
    apk add --upgrade --no-cache git make gcc musl-dev && \
    make test && \
    make

# Set FROM image as starting point for executor
FROM rawmind/alpine-base:3.10-1

# Set some maintainer info
MAINTAINER Raul Sanchez &lt;raul@rancher.com&gt;

#Setting arguments for execution
ARG SERVICE_NAME=hello-world-app
ARG SERVICE_HOME=/opt/${SERVICE_NAME}
ARG SERVICE_REPO=rawmind0

#Set environment
ENV SERVICE_NAME=${SERVICE_NAME} \
    SERVICE_HOME=${SERVICE_HOME} \
    SERVICE_USER=rancher \
    SERVICE_UID=10012 \
    SERVICE_GROUP=rancher \
    SERVICE_GID=10012 \
    PATH=${PATH}:${SERVICE_HOME}

# Set app workdir
WORKDIR $SERVICE_HOME/

# Copy app binary from builder image
COPY --from=builder /go/src/github.com/${SERVICE_REPO}/${SERVICE_NAME}/${SERVICE_NAME} .
COPY img/ ./

# Create the user to execute app
RUN addgroup -g ${SERVICE_GID} ${SERVICE_GROUP} && \
    adduser -g "${SERVICE_NAME} user" -D -h ${SERVICE_HOME} -G ${SERVICE_GROUP} -s /sbin/nologin -u ${SERVICE_UID} ${SERVICE_USER} && \
    chown -R ${SERVICE_USER}:${SERVICE_GROUP} ${SERVICE_HOME} && \
    setcap 'cap_net_bind_service=+ep' ${SERVICE_HOME}/${SERVICE_NAME}

# Set user to execute the app
USER $SERVICE_USER

# Set app entrypoint
ENTRYPOINT ["/opt/hello-world-app/hello-world-app"]
```

To test it, execute the following

```
# docker build -f package/Dockerfile.multistage .
```

### tagging docker image

Once docker image is builded, it should be tagged to get identified before it get pushed into a docker registry. The tag should be on the form &lt;docker_registry_url&gt;/&lt;docker_registry_url&gt;/<docker_image&gt;:&lt;docker_image_version&gt; . If &lt;docker_registry_url&gt; is not specified, docker hub is set by default. 

```
# docker tag SOURCE_IMAGE &lt;docker_user&gt;/hello-world:dev
```

Tag can also be done at building time, 

```
# docker build -t &lt;docker_user&gt;/hello-world:dev -f package/Dockerfile.multistage .
```

### testing docker image

Test is the most important and the hardest part to implement on some developments. On our app, we are defining a quick and dirty test to show the concept. 

This test just run the docker image and test that is able to connect to the service and the version is correct.

```
# docker run -td --name hello-world-dev  &lt;docker_user&gt;/hello-world:dev
# docker exec -it ${PKG_NAME}-${VERSION} curl -s localhost:8080/version
# docker rm -fv hello-world-dev
```

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

**Important Note:** HobbyFarm will tear down your provisioned resources within 5 minutes if your laptop goes to sleep or you navigate off of the HobbyFarm page. Please ensure that you do not do this, for example, during lunch or you will need to restart your scenario.

# Introduction

The goal of this training module is help to developers to onboard their apps on Rancher. This modules defines some agile development concepts and practical exercises to be used as a basic example to onboard apps on Rancher.

## Requirements

- Github user
- Docker hub user
- git, docker and make installed 
- kubectl and helm installed

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


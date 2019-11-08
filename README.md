# hb_dev_train

This the hobbyfarm scenario for dev train.

## Requirements

4 vm's are required to execute this hobbifarm scenario, 1 vm for rancher server and 3 vm for k8s cluster.

## Index

* Intoduction
  * CI/CD/CD concepts to onboard apps
    * CI: What to deploy..
    * CD: How to deploy/upgrade..
    * CD: Where to deploy/upgrade..
* Continuous Integration 
  * Dockerizing app
    * building docker image
      * single-stage building
      * multi-stage building
      * layers
    * tagging docker image
    * testing docker image
  * Publishing app
    * Pushing app image
  * Creating pipeline
* Continuous Delivery
  * Upgrade strategy
  * Kubernetes manifest
	* deployment/daemonset file
	* service file
	* ingress file
  * Helm chart
    * Creating repo
    * Creating app chart
  * Testing deployment
  * Creating pipeline
* Continuous Deployment
  * Rancher
    * Workloads (manifest files)
      * Deploying workload
    * Apps (Helm charts)
      * Adding catalog
      * Deploying app
      * Updating app
  * Creating pipeline

## Scenario steps

Every file under `steps` folder is a mardown file. Its content should be added as step to hobbyfarm scenario.

## License
Copyright (c) 2014-2018 [Rancher Labs, Inc.](http://rancher.com)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

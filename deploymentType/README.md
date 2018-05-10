# minishift-ci-cd
Deploying different deployment type - Minishift

## Requirements
- [minishift](https://docs.openshift.org/latest/minishift/getting-started/installing.html) 1.5+
- Make sure minishift is available in your PATH

## Build

### Execute step by step
```
./minishift-start 
./minishift-deploy-rolling-canary 
./minishift-deploy-recreate 
./minishift-deploy-bluegreen 
./minishift-change-route-bluegreen 
./minishift-start-pipeline
```

* minishift-start
```
minishift start --vm-driver=virtualbox && \
minishift oc-env
```
Deploy new app with deployment config rolling canary
* minishift-deploy-rolling-canary
```
oc login -u developer
oc new-app openshift/deployment-example && \
oc expose svc/deployment-example && \
oc scale dc/deployment-example --replicas=3 && \
oc tag --source=docker openshift/deployment-example:v2 deployment-example:latest
oc deploy deployment-example
```
This operation configure a recreate deployment by updating a deployment config
  
* minishift-deploy-recreate
```
eval $(minishift oc-env)
oc login -u developer
oc create -f ../templates/recreate-example.yaml
oc tag recreate-example:v2 recreate-example:latest
oc deploy recreate-example
```
This operation configure a bluegreen deployment by creating two new app
  
* minishift-deploy-bluegreen
```
eval $(minishift oc-env)
oc login -u developer
oc new-app openshift/deployment-example:v1 --name=bluegreen-example-old
oc new-app openshift/deployment-example:v2 --name=bluegreen-example-new
oc expose svc/bluegreen-example-old --name=bluegreen-example
```
This operation edit the route and change the service to bluegreen-example-new
  
* minishift-change-route-bluegreen
```
eval $(minishift oc-env)
oc login -u developer
oc edit route/bluegreen-example
```
Change spec.to.name to bluegreen-example-new and save and exit the editor.

## Create project minishift within ci platform to execute different deployment strategies

### All-in bootstrapDeployment
```
./minishift-start && \
./minishift-create-project && \
./minishift-create-app && \
./minishift-start-pipeline
```

### Manual step by step 
* Start with clean installation
```
minishift delete --clear-cache
```
* minishift-start
```
minishift start --memory=10240 --vm-driver=virtualbox && \
minishift oc-env
```
This will create a new project 
* minishift-create-project 
```
eval $(minishift oc-env)
oc login -u developer
oc new-project testbluegreen --display-name="test blue green"
```
This operation will create a new app from yaml templates
* minishift-create-app
eval $(minishift oc-env)
oc login -u developer
oc new-app -n testbluegreen -f ../templates/bluegreen-example.yaml
```
This operation will label the last version of the image used, 
then the implementation of the deployment element will be triggered to later balance the routing element to the service where the latest version was deployed.
* minishift-start-pipeline
eval $(minishift oc-env)
oc login -u developer
oc tag blue-green-deployment-example:v1 blue-green-deployment-example:latest
oc start-build blue-green-deployment-example
```
If we want to re-launch the pipeline we generate a new version image tag and execute the pipeline. We consult results now


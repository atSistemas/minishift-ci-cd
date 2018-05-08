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
# minishift-ci-cd
CI/CD Demo - Minishift

Provision minishift and setup required CI components (Gogs, Jenkins, SonarQube). 
Deploy a demo app to showcase a CI/CD pipeline.

## Requirements
- [minishift](https://docs.openshift.org/latest/minishift/getting-started/installing.html) 1.5+
- Make sure minishift is available in your PATH

## Build

### All-in bootstrap
* bootstrap
```
./minishift-start && \
./minishift-warmup && \
./minishift-create-projects && \
./minishift-admin-setup && \
./minishift-deploy-demo && \
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
This will prompt the web console URL and logging details
* minishift-warmup
```
oc login -u developer
minishift ssh docker pull openshiftdemos/gogs:0.11.29
minishift ssh docker pull openshiftdemos/sonarqube:6.7
minishift ssh docker pull sonatype/nexus3:3.6.1
minishift ssh docker pull registry.access.redhat.com/openshift3/jenkins-2-rhel7
minishift ssh docker pull registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7 
minishift ssh docker pull registry.access.redhat.com/jboss-eap-7/eap70-openshift
```
This operation will take some time as it will download some docker images!!!
  
* minishift-create-projects
```
eval $(minishift oc-env)
oc login -u developer
# Create Projects
echo "Creating DEV namespace ..."
oc new-project dev --display-name="Dev Environment" && \
echo "Creating STAGE namespace ..."
oc new-project stage --display-name="Stage Environment" && \
echo "Creating CI/CD namespace ..."
oc new-project cicd --display-name="CI/CD"

# Grant Jenkins Access to Projects
echo "Adding roles to admin user ..."
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n stage
```
* minishift-admin-setup
```
eval $(minishift oc-env)
oc login -u system:admin
# Make sure latest Jenkins image is installed
oc import-image jenkins:v3.7 --from="registry.access.redhat.com/openshift3/jenkins-2-rhel7" --confirm -n openshift
oc tag jenkins:v3.7 jenkins:latest -n openshift

# Manage policy on the cluster
oc adm policy add-role-to-user admin system -n dev 
oc adm policy add-role-to-user admin system -n stage 
oc adm policy add-role-to-user admin system -n cicd
 
# Update the annotations on namespace resources    
oc annotate --overwrite namespace dev demo=openshift-cd 
oc annotate --overwrite namespace stage demo=openshift-cd
oc annotate --overwrite namespace cicd  demo=openshift-cd

oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
```

* minishift-deploy-demo 
```
eval $(minishift oc-env)
oc login -u developer
# Download ci-cd template and create app from it
oc new-app -n cicd -f https://raw.githubusercontent.com/OpenShiftDemos/openshift-cd-demo/ocp-3.7/cicd-template.yaml
```
* minishift-start-pipeline
```
eval $(minishift oc-env)
oc start-build tasks-pipeline
```
This operation will take some time as it will have to download all artifacts from nexus 
and promote the demo app across all the pipeline stages.
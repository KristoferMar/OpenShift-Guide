## Deploying and Managing Applications on an OpenShift Cluster
#### Create new project
```
oc new-project my-app --display-name="My Application"
```
View project
```
oc get project my-app
```

#### Image registry
View the image registry
```
oc get routes -n openshift-image-registry
```

#### Put image into registry
```
docker tag my-local-image:latest the-openshift-image-registry/my-project/my-app:latest
```
Login to image registry
```
docker login -u $(oc whoami) -p $(oc whoami -t) the-openshift-image-registry
```
Push image to cluster
```
docker push the-openshift-image-registry/my-project/my-app:latest
```

#### Deploy application
```
oc new-app my-project/my-app:latest
```


## Deploying Containerized Applications for OpenShift
## Enterprise container images
- ImageStreams

## Managing Builds on OpenShift 
## Customizing Source-to-Image Builds

## Deploying multi container applications
- helm
- templates## Managing Application Deployments 

## Managing Applications Deployments
- Deployment Strategies

## Building Applications for OpenShift

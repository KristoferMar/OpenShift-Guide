# Customizing a Deployment with Kustomize

Kustomize is a tool that customizes Kubernetes resources for different environments or needs. Kustomize is a template free solution that helps reuse configurations and provides an easy way to patch any resource.

### Resource files 
The resource section of the kustomization.yaml file is a list of files thta combined create all the resources needed for this specific environment.

<pre>
resources:
- deployment.yaml
- secrets.yaml
- service.yaml
</pre>

The layout for a base definition as represented with these three resources would look like
<pre>
myapp
└── base
  ├── deployment.yaml
  ├── kustomization.yaml
  ├── secrets.yaml
  └── service.yaml
</pre>

## Applying customization
Standalone way
<pre>
[user@host ~]$ kustomize build myapp/base | oc apply -f -
</pre>

Integration in the cluster management tools
<pre>
[user@host ~]$ oc apply -k myapp/base
</pre>

Applying the overlay modifications
<pre>
[user@host ~]$ oc apply -k myapp/overlays/staging
</pre>

Kustomize is only available as part of the kubectl apply or oc apply commands because it is an extension of the declarative management of Kubernetes objects, which is designed to work in combination with source version control systems

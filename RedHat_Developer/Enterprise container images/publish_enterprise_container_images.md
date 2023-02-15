# Publish Enterprise Container Images

### Public registries
Registries that allow anyone to consume container images directly from the internet without any authentication. Docker Hub, Quay.io, and the Red Hat Registry are examples of public container registries.

### Private registries
Registries that are available only to selected consumers and usually require authentication. The Red Hat terms-based registry is an example of a private container registry.

### External registries
Registries that your organization does not control. They are usually managed by a cloud provider or a software vendor. Quay.io is an example of an external container registry.

### Enterprise registries
Registry servers that your organization manages. They are usually available only to the organization's employees and contractors.

### OpenShift internal registries
A registry server managed internally by an OpenShift cluster to store container images. Create those images using OpenShift's build configurations and the S2I process or a Containerfile, or import them from other registries.

## Skopeo
You can use it in various ways.
- You can manage container registires with skopeo
- Copy images back and forth
- Delete images from a registry
- Inspect to view metadata about an image
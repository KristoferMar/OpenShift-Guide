# Creating Image Streams

An "ImageStream" is basically the same as simply an image - they call it "stream" becuase there is a mechanism that both updates the Image when uploading new tags but it also keeps the old tags available allowing the developer to publish both newest and older versions. 

Image streams provide a stable, short name to reference a container image that is independent of any registry server and container runtime configuration.

An image stream represents one or more sets of container images. Each set, or stream, is identified by an image stream tag. Unlike container images in a registry server, which have multiple tags from the same image repository (or user or organization), an image stream can have multiple image stream tags that reference container images from different registry servers and from different image repositories.

An image stream provides default configurations for a set of image stream tags. Each image stream tag references one stream of container images, and can override most configurations from its associated image stream.

An image stream tag stores a copy of the metadata about its current container image and can optionally store a copy of its current and past container image layers. Storing metadata allows faster search and inspection of container images, because you do not need to reach its source registry server.

## Import imagestream into openshift

Adding image to local registry
<pre>
oc import-image default-route-openshift-image-registry.apps-crc.testing/openshift/system-db:v1 --confirm
</pre>
## Granting Access to Images in an Internal Registry

Any user with access to an OpenShift project can push and pull images to and from that project, according to their access level. If a user has either the admin or edit roles on the project, they can both pull and push images to that project. If instead they have only the view role on the project, they can only pull images from that project.

OpenShift also offers a few specialized roles for when you want to grant access only to images inside a project, and not grant access to perform other development tasks such as building and deploying applications inside the project. The most common of these roles are:

### registry-viewer and system:image-puller
These roles allow users to pull and inspect images from the internal registry.

### registry-editor and system:image-pusher
These roles allow users to push and tag images to the internal registry.

The system:* roles provide the minimum capabilities required to pull and push images to the internal registry. As stated before, OpenShift users who already have admin or edit roles in a project do not need these system:* roles.

The registry-* roles provide more comprehensive capabilities around registry management for organizations that want to use the internal registry as their enterprise registry. These roles grant additional rights such as creating new projects but do not grant other rights, such as building and deploying applications. The OCI standards do not specify how to manage an image registry, so whoever manages an OpenShift internal registry needs to know about OpenShift administration concepts and the oc command. This makes the registry-* less useful.

The following example allows a user to pull images from the internal registry in a given project. You need to have either project or cluster-wide administrator access to use the oc policy command.
<pre>
[user@host ~] oc policy add-role-to-user system:image-puller \
user_name -n project_name
</pre>
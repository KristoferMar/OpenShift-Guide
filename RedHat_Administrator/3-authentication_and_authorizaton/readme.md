# Configuring Authentication and Authorization
    
- A newly-installed OpenShift cluster provides two authentication methods that grant administrative access: the kubeconfig file and the kubeadmin virtual user.

- The HTPasswd identity provider authenticates users against credentials stored in a secret. The name of the secret, and other settings for the identity provider, are stored inside the OAuth custom resource.

- To manage user credentials using the HTPasswd identity provider, you must extract data from the secret, change that data using the htpasswd command, and then apply the data back to the secret.

- Creating OpenShift users requires valid credentials, managed by an identity provider, plus user and identity resources.

- Deleting OpenShift users requires deleting their credentials from the identity provider, and also deleting their user and identity resources.

- OpenShift uses role-based access control (RBAC) to control user actions. A role is a collection of rules that govern interaction with OpenShift resources. Default roles exist for cluster administrators, developers, and auditors.

- To control user interaction, assign a user to one or more roles. A role binding contains all of the associations of a role to users and groups.

- To grant a user cluster administrator privileges, assign the cluster-admin role to that user.
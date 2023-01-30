## Role-based Access Control (RBAC)

## Authorization Process
The authorization process is managed by rules, roles, and bindings.
- Rule: Allowed actions for objects or groups of objects.
- Role: Sets of rules. Users and groups can be associated with multiple roles.
- Binding: Assignment of users or groups to a role.

## RBAC Scope
Red Hat OpenShift Container Platform (RHOCP) defines two groups of roles and bindings
depending on the user's scope and responsibility: cluster roles and local roles.
- Cluster Role: Users or groups with this role level can manage the OpenShift cluster.
- Local Role: Users or groups with this role level can only manage elements at a
project level.

## Managing RBAC Using the CLI
Cluster administrators can use the oc adm policy command to both add and remove cluster roles and namespace roles.

To add a cluster role to a user, use the add-cluster-role-to-user subcommand
<pre>
[user@host ~]$ oc adm policy add-cluster-role-to-user cluster-role username
</pre>

Change a regular user to a cluster administrator
<pre>
[user@host ~]$ oc adm policy add-cluster-role-to-user cluster-admin username
</pre>

To remove a cluster role from a user, use the remove-cluster-role-from-user
subcommand
<pre>
[user@host ~]$ oc adm policy remove-cluster-role-from-user cluster-role username
</pre>

change a cluster administrator to a regular user
<pre>
[user@host ~]$ oc adm policy remove-cluster-role-from-user cluster-admin username
</pre>

You can use the oc adm policy who-can command to determine if a user can execute an
action on a resource
<pre>
[user@host ~]$ oc adm policy who-can delete user
</pre>

## Default Roles
OpenShift ships with a set of default cluster roles that can be assigned locally or to the entire
cluster. You can modify these roles for fine-grained access control to OpenShift resources, but
additional steps are required that are outside the scope of this course.

- admin: Users with this role can manage all project resources, including granting
access to other users to access the project.

- basic-user: Users with this role have read access to the project.

- cluster-admin: Users with this role have superuser access to the cluster resources. These
users can perform any action on the cluster, and have full control of all
projects.

- cluster-status: Users with this role can get cluster status information.

- edit: Users with this role can create, change, and delete common application
resources from the project, such as services and deployments. These users
cannot act on management resources such as limit ranges and quotas, and
cannot manage access permissions to the project.

- self-provisioner: Users with this role can create new projects. This is a cluster role, not a
project role.

- view: Users with this role can view project resources, but cannot modify project
resources.

Project administrators can use the oc policy command to add and remove namespace roles.

Add a specified role to a user with the add-role-to-user subcommand.
<pre>
[user@host ~]$ oc policy add-role-to-user role-name username -n project
</pre>

To add the user dev to the role basic-user in the wordpress project
<pre>
[user@host ~]$ oc policy add-role-to-user basic-user dev -n wordpress
</pre>

## User Types

### Regular users
Most interactive OpenShift Container Platform users are regular users, represented with the User object.

### System users
Many system users are created automatically when the infrastructure is defined, mainly
for the purpose of enabling the infrastructure to securely interact with the API.

### Service accounts
These are special system users associated with projects.
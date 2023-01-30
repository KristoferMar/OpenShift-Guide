# Introducing OpenShift Dynamic Storage

## Persistent Storage Overview
Containers have ephemeral storage by default. For example, when a container is deleted, all the
files and data inside it are deleted also.

To preserve the files, containers offer two main ways of
maintaining persistent storage: volumes and bind mounts.

## Persistent Volume and Persistent Volume Claim Life Cycle

## Verifying the Dynamic Provisioned Storage

## Deploying Dynamically Provisioned Storage

## Deleting Persistent Volume Claims
To delete a volume, use the oc delete command to delete the persistent volume claim. The
storage class will reclaim the volume after the PVC is removed.
<pre>
[user@host ~]$ oc delete pvc/example-pvc-storage
</pre>

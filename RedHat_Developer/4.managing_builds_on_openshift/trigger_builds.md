# Trigger builds

Two kinds of build triggers

## Image change triggers 
An image change trigger rebuilds an application container image to incorporate changes made by its parent image.

## Webhook triggers
Red Hat OpenShift webhook triggers are HTTP API endpoints that start a new build. Use a webhook to integrate OpenShift with your Version Control System, such as Github or BitBucket, to trigger new builds on code changes.

See triggers associated with a build configuration
<pre>
[user@host ~]$ oc describe bc/name
</pre>

Add an image change trigger to build configuration.
<pre>
[user@host ~]$ oc set triggers bc/name --from-image=project/image:tag
</pre>

Remove image change trigger from a build configuration
<pre>
[user@host ~]$ oc set triggers bc/name --from-image=project/image:tag --remove
</pre>

The oc new-app command creates a generic and a Git webhook. To add other types of webhook to a build configuration, use the oc set triggers command. For example, to add a GitLab webhook to a build configuration
<pre>
[user@host ~]$ oc set triggers bc/name --from-gitlab
</pre>

To remove an existing webhook from a build configuration, use the oc set triggers command with the --remove option. For example, to remove a GitLab webhook from a build configuration
<pre>
[user@host ~]$ oc set triggers bc/name --from-gitlab --remove
</pre>

## Updaing image stream to see triggers 
When you have your application deployed with the following triggers
<pre>
[kris@workstation trigger-builds]$ oc describe bc/trigger | grep Triggered
Triggered by: Config, ImageChange
</pre>

You can update the image stream to start a new build
<pre>
[kris@workstation trigger-builds]$ skopeo copy --format v2s1 \
oci-archive:php-73-ubi8-newer.tar.gz \
docker://quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8:latest
Getting image source signatures
...output omitted...
Writing manifest to image destination
Storing signatures
</pre>

Now update the php image stream in this case to fetch the metadata for the new container image
<pre>
[kris@workstation trigger-builds]$ oc import-image php
imagestream.image.openshift.io/php-73-ubi8 imported

Name:                   php
...output omitted...
latest
  tagged from quay.io/yourquayuser/php-73-ubi8

  * quay.io/yourquayuser/php-73-ubi8@sha256:3f5e...6ab7
      Less than a second ago
    quay.io/yourquayuser/php-73-ubi8@sha256:c919...8cb2
      2 minutes ago
...output omitted...
</pre>

Check the build 
<pre>
[kris@workstation trigger-builds]$ oc get builds
NAME        TYPE      FROM          STATUS     STARTED          DURATION
trigger-1   Source    Git@da010df   Complete   13 minutes ago   58s
trigger-2   Source    Git@da010df   Complete   28 seconds ago   15s
</pre>
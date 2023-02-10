# Managing Applcation Builds

### oc new-app
When you use oc new-app command there will be created a build configuration behind the scenes based on the arguments specified. For example, if a Git repository is defined, then build configuration using the source stratigy is created. Also, if a template is the argument, and the template has a build configuration defined, it creates a build configuration based on template parameters.

### Custom build configuration
Using this method we create a custom build configuration using YAML or JSON syntax and import it to RedHat OpenShift using the oc create -f command.

## Managing application Builds Using CLI 

### oc start-build
<pre>
[user@host ~]$ oc start-build name
</pre>

###  oc cancel-build 
<pre>
[user@host ~]$ oc cancel-build name
</pre>

### oc delete
<pre>
[user@host ~]$ oc delete bc/name
</pre>

This deletes the a build (not build configuration) to reclaim the space used bt the build.
<pre>
[user@host ~]$ oc delete build/name-1
</pre>

### oc logs
Display the build logs from the most recent build.
<pre>
[user@host ~]$ oc logs -f bc/name
</pre>
The -f option follows the log until you terminate the command with Ctrl+C.

Display the build logs from a specific build
<pre>
[user@host ~]$ oc logs build/name-1
</pre>

## Pruning Builds
By default, the five most recent builds that have completed are persisted. You can limit the number of previous builds that are retained using the successfulBuildsHistoryLimit attribute, and the failedBuildsHistoryLimit attribute, as shown in the following YAML snippet of a build configuration
<pre>
apiVersion: "v1"
kind: "BuildConfig"
metadata:
  name: "sample-build"
spec:
  successfulBuildsHistoryLimit: 2
  failedBuildsHistoryLimit: 2
  ...contents omitted...
</pre>
Failed builds include builds with a status of failed, canceled or error.

The following command is used for prune
<pre>
oc adm prune
</pre>
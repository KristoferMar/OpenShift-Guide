# Implementing Post-commit Build Hooks
Why use it?
- To integrate a build with an external application through an HTTP API
- To validate a non-functional requirement such as application performance, secutiry, usability or compatibility.
- To send an email to the developer's team informing them of the new build

There are multiple ways to implement post-commit build hooks
## Command
A command is executed using the exec system call. 
<pre>
[user@host ~]$ oc set build-hook bc/name --post-commit \
--command -- bundle exec rake test --verbose
</pre>

## Shell script
Runs a build hook with the /bin/sh -ic command. This is more convenient since it has all the features provided by a shell, such as argument expansion, redirection, and so on. It only works if the base image has the sh shell.
<pre>
[user@host ~]$ oc set build-hook bc/name --post-commit \
--script="curl http://api.com/user/${USER}"
</pre>


## Example
Example on how you can set a build-hook using command.
<pre>
oc set build-hook bc/hook --post-commit --command -- \
    bash -c "curl -s -S -i -X POST http://builds-for-managers-${RHT_OCP4_DEV_USER}-post-commit.${RHT_OCP4_WILDCARD_DOMAIN}/api/builds -f -d 'developer=\${DEVELOPER}&git=\${OPENSHIFT_BUILD_SOURCE}&project=\${OPENSHIFT_BUILD_NAMESPACE}'"
</pre>
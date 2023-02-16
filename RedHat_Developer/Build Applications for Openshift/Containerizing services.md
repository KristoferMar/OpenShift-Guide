# Containerizing Services

## Containerizing Nexus as a Service

We start off by inspecting our Dockerfileand verify Nexus persists ints files in /nexus-data 
<pre>
[student@workstation nexus3]$ grep VOLUME Dockerfile
VOLUME ${NEXUS_DATA}
[student@workstation nexus3]$ grep NEXUS_DATA Dockerfile
    NEXUS_DATA=/nexus-data \
...output omitted...
</pre>

Inspect the Dockerfile file and observe that it defines an environment variable that sets Java Virtual Machine (JVM) heap sizes. Later, we must override this environment variable so that the OpenShift deployment configuration controls the pod memory settings.
<pre>
[student@workstation nexus3]$ grep ENV Dockerfile
...output omitted...
ENV INSTALL4J_ADD_VM_PARAMS="`-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m` -Djava.util.prefs.userRoot=${NEXUS_DATA}/javaprefs"
</pre>

Build the image and push it to a registry
<pre>
[student@workstation nexus3]$ podman build -t nexus3 .
...output omitted...
STEP 33: COMMIT nexus3
[student@workstation nexus3]$ podman login -u ${RHT_OCP4_QUAY_USER} quay.io
Password:
Login Succeeded!
[student@workstation nexus3]$ skopeo copy \
--format v2s1 containers-storage:localhost/nexus3 \
docker://quay.io/${RHT_OCP4_QUAY_USER}/nexus3
...output omitted...
Writing manifest to image destination
Storing signatures
</pre>

WILL GET BACK TO THIS.
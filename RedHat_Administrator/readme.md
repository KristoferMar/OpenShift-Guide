## Create a Security Context Constraints (SCC) for Privileged Containers
First create the file
```
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-privileged-scc
allowPrivilegedContainer: true
allowedCapabilities:
- '*'
allowHostDirVolumePlugin: true
allowHostIPC: true
allowHostNetwork: true
allowHostPID: true
allowHostPorts: true
allowHostPath: true
allowRunAsUser: true
defaultAddCapabilities:
- '*'
fsGroup:
  type: RunAsAny
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
users:
- system:serviceaccount:<your-namespace>:default
```

Apply the SCC
```
oc apply -f custom-privileged-scc.yaml
```

Assign the SCC to Service Account
```
oc get sa -n <your-namespace>
and then
oc adm policy add-scc-to-user custom-privileged-scc -z default -n <your-namespace>
```

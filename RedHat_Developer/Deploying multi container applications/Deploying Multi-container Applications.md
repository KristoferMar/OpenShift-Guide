# Deploying Multi-container Applications

Templates define parameters that are used to customize the resource configuration. 

## Template Syntax
<pre>
apiVersion: template.openshift.io/v1
kind: Template (1)
metadata:
  name: mytemplate
  annotations:
    description: "Description" (2)
objects: (3)
- apiVersion: v1
  kind: Pod
  metadata:
    name: myapp
  spec:
    containers:
    - env:
      - name: MYAPP_CONFIGURATION
        value: ${MYPARAMETER} (4)
      image: myorganization/myapplication
      name: myapp
      ports:
      - containerPort: 80
        protocol: TCP
parameters: (5)
- description: Myapp configuration data
  name: MYPARAMETER
  required: true
labels: (6)
  mylabel: myapp
</pre>

1. Template Resource type
2. Optional annotations for use by RHOCP tools
3. Resource list
4. Reference to a template parameter
5. Parameter list
6. Label list

## Create an Application from a Template
Deploy application from template file
<pre>
[user@host ~]$ oc new-app --file mytemplate.yaml -p PARAM1=value1 \
-p PARAM2=value2
</pre>

This applies values to a tmplate and stores the results in a local file
<pre>
[user@host ~]$ oc process -f mytemplate.yaml -p PARAM1=value1 \
-p PARAM2=value2 > myresourcelist.yaml
</pre>

The file generated above is given to the oc create command
<pre>
[user@host ~]$ oc create -f myresourcelist.yaml
</pre>

The above can be piped together
<pre>
[user@host ~]$ oc process -f mytemplate.yaml -p PARAM1=value1 \
-p PARAM2=value2 | oc create -f -
</pre>

It's recommended to use "new-app" instead of "process".

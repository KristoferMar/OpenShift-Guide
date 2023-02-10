# How to create a Helm Chart

Create a new Helm Chart.
<pre>
[kris@workstation multicontainer-helm]$ helm create famouschart
Creating famouschart
[kris@workstation multicontainer-helm]$ cd famouschart
</pre>

The folder structure will look like the following 
<pre>
[kris@workstation famouschart]$ tree .
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
</pre>

Configure application deployment.
Use the values.yaml file to configure the resulting deployment for the application.

Modify the repository property of the image section to match our needs.
<pre>
image:
  repository: quay.io/redhattraining/famous-quotes
  pullPolicy: IfNotPresent
  tag: "2.1"
</pre>

Modify the container connection to use the right port, this is changed in the "templates/deployment.yml" file. 
<pre>
ports:
  - name: http
    containerPort: 8000
    protocol: TCP
</pre>

Add the database dependency.

To do this, add the following snippet at the end of the Chart.yaml file:
<pre>
dependencies:
- name: mariadb
  version: 11.0.13
  repository: https://charts.bitnami.com/bitnami
</pre>

Update the dependencies for the chart.
This downloads the charts added as dependencies and locks its versions.
<pre>
[kris@workstation famouschart]$ helm dependency update
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://charts.bitnami.com/bitnami" chart repository
Saving 1 charts
Downloading mariadb from repo https://charts.bitnami.com/bitnami
Deleting outdated charts
</pre>

Set up the database to use custom values for authentication and security.

it's good to be able to control the values instead of letting the database helm chart create ones at random.

Add the following lines at the end of the values.yaml file in this example
<pre>
mariadb:
  auth:
    username: quotes
    password: quotespwd
    database: quotesdb
  primary:
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
</pre>

Configure the application's database access by using environmental variables.

The default deployment template does not pass any environmental variable to the deployed applications. Modify the templates/deployment.yaml template to pass the environmental variables defined in the values.yaml file to the application's container, add the following snippet after the imagePullPolicy value in the containers section
<pre>
imagePullPolicy: {{ .Values.image.pullPolicy }}
env:
  {{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
  {{- end }}
</pre>

Add the appropriate environmental variables at the end of the values.yaml file:
<pre>
env:
  - name: "QUOTES_HOSTNAME"
    value: "famousapp-mariadb"
  - name: "QUOTES_DATABASE"
    value: "quotesdb"
  - name: "QUOTES_USER"
    value: "quotes"
  - name: "QUOTES_PASSWORD"
    value: "quotespwd"
</pre>

## Deploy the application using the Helm chart.

<pre>
[kris@workstation famouschart]$ oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
[kris@workstation famouschart]$ oc new-project \
${RHT_OCP4_DEV_USER}-multicontainer-helm
</pre>

Use the helm install command to deploy the application in the RHOCP cluster
<pre>
[kris@workstation famouschart]$ helm install famousapp .
NAME: famousapp
LAST DEPLOYED: Thu May 20 19:12:09 2021
NAMESPACE: yourdevuser-multicontainer-helm
STATUS: deployed
REVISION: 1
NOTES:
...output omitted...
</pre>

Verify
<pre>
[kris@workstation famouschart]$ oc get deployments
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
famousapp-famouschart      0/1     1            1           10s
</pre>
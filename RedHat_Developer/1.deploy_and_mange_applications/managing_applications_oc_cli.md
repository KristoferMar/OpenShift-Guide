# Managing Applications with the CLI

Example of a build template

```
{
    "kind": "Template",
    "apiVersion": ""template.openshift.io/v1",
    "metadata": {
        "name": "php-mysql-ephemeral",  1
...output omitted...
    "objects": [
        {
            "apiVersion": "v1",
            "kind": "Secret",  2
...output omitted...
            },
            "stringData": {
                "database-password": "${DATABASE_PASSWORD}",
                "database-user": "${DATABASE_USER}"
...output omitted...
        {
            "apiVersion": "route.openshift.io/v1",
            "kind": "Route",  3
...output omitted...
            "spec": {
                "host": "${APPLICATION_DOMAIN}",
                "to": {
                    "kind": "Service",
                    "name": "${NAME}"
...output omitted...
```

1. The template is a copy of the standard cakephp-mysql-example template, with resources and parameters specific to that framework deleted. The custom template is suitable for any simple PHP application that uses a MySQL database.

2. A secret stores database login credentials and populates environment variables in both the application and the database pods. OpenShift secrets are explained later in this book.

3. A route provides external access to the application.

Get avaiable tempaltes

```
[kris@workstation DO288-apps]$ oc get templates -n openshift | grep php \
| grep mysql
cakephp-mysql-example       An example CakePHP application ...
cakephp-mysql-persistent    An example CakePHP application ...
```

Create template from json file located on local workstation.

```
[kris@workstation DO288-apps]$ oc create -f \
~/DO288/labs/build-template/php-mysql-ephemeral.json
template.template.openshift.io/php-mysql-ephemeral created
```

Review the template parameters 

```
[kris@workstation DO288-apps]$ oc describe template php-mysql-ephemeral \
-n ${RHT_OCP4_DEV_USER}-common
Name:		php-mysql-ephemeral
...output omitted...
Parameters:
    Name:           NAME
    Display Name:   Name
    Description:    The name assigned to all of the app objects defined in this template.
    Required:       true
    Value:          php-app
...output omitted...
```

After creating an app using the template you can check the logs

```
[kris@workstation DO288-apps]$ oc logs -f bc/quotesapi
Cloning "https://github.com/your-GitHub-user/DO288-apps" ...
...output omitted...
Push successful
```

Get the ip andress and endpoint of pod

```
[kris@workstation DO288-apps]$ oc describe svc quotesdb | grep Endpoints
Endpoints:		10.8.0.71:3306
[kris@workstation ~]$ oc describe pod quotesdb-6b7ffcc649-dslpq | grep IP
IP:			10.8.0.71
...output omitted...
```

Verify the database connection parameters in application pod

```
[kris@workstation DO288-apps]$ oc describe pod quotesapi-7d76ff58f8-6j2gx \
| grep -A 5 Environment
    Environment:
      DATABASE_SERVICE_NAME:    quotesdb
      DATABASE_NAME:            phpapp
      DATABASE_USER:            <set to the key 'database-user' in secret 'quotesapi'>
      DATABASE_PASSWORD:        <set to the key 'database-password' in secret 'quotesapi'>
    Mounts:
```

Verify that application pod can reach the database pod

```
[kris@workstation DO288-apps]$ oc rsh quotesapi-7d76ff58f8-6j2gx bash -c \
'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'
OK
```

Review application logs

```
[kris@workstation DO288-apps]$ oc logs quotesapi-7d76ff58f8-6j2gx
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.129.0.143. Set the 'ServerName' directive globally to suppress this message
...output omitted...
[Mon May 27 14:54:56.187516 2019] [php7:notice] [pid 52] [client 10.128.2.3:43952] SQL error: Table 'phpapp.quote' doesn't exist\n
10.128.2.3 - - [27/May/2019:14:54:51 +0000] "GET /get.php HTTP/1.1" 500 - "-" "curl/7.29.0"
...output omitted...
```

Copy SQL script to database pod

```
[kris@workstation DO288-apps]$ oc cp ~/DO288/labs/build-template/quote.sql \
quotesdb-6b7ffcc649-dslpq:/tmp/quote.sql
```

Run SQL script inside the database pod

```
[kris@workstation DO288-apps]$ oc rsh -t quotesdb-6b7ffcc649-dslpq
sh-4.2$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE < /tmp/quote.sql
...output omitted...
sh-4.2$ exit
[kris@workstation DO288-apps]$
```

You can now access application to verify that it works

```
[kris@workstation DO288-apps]$ curl -si \
http://quote-$RHT_OCP4_DEV_USER.$RHT_OCP4_WILDCARD_DOMAIN/get.php
HTTP/1.1 200 OK
...output omitted...
Always remember that you are absolutely unique. Just like everyone else.
```

# Managing Builds on OpenShift

The following are the available build strategies in Red Hat OpenShift
- Source-to-image (S2I) build
- Docker build
- Custom build

## Build Input Sources
A build input source provides source content for builds. Red Hat OpenShift supports the following six types of input sources, listed in order of precedence
-  Containerfile: Specifies the Containerfile inline to build an image.
-  Image: You can provide additional files to the build process when you build from images.
-  Git: Red Hat OpenShift clones the input application source code from a Git repository. It is possible to configure the default location inside the repository where the build looks for application source code.
-  Binary: Allows streaming binary content from a local file system to the builder.
-  Input secrets: You can use input secrets to allow creating credentials for the build that are not available in the final application image.
-  External artifacts: Allow copying binary files to the build process.

It's possible to combine multiple inputs in a single build.

## BuildConfig Resource
It defines how the build process happens. The BuildConfig resource defines a single build configuration and a set of triggers for when Red Hat OpenShift must create a new build.

Red Hat OpenShift generates the BuildConfig using the oc new-app command. It can also be generated using the Add to Project button from the web console or by creating an application from a template.

The following example builds a PHP application using the Source strategy and a Git input source:
<pre>
{
    "kind": "BuildConfig",
    "apiVersion": "v1",
    "metadata": {
        "name": "php-example", 
        ...output omitted...
    },
    "spec": {
        "triggers": [ 2
            {
                "type": "GitHub",
                "github": {
                    "secret": "gukAWHzq1On4AJlMjvjS"  
                }
            },
            ...output omitted...
        ],
        "runPolicy": "Serial", 
        "source": { 
            "type": "Git",
            "git": {
                "uri": "http://services.lab.example.com/php-helloworld"
            }
        },
        "strategy": { 
            "type": "Source",
            "sourceStrategy": {
                "from": {
                    "kind": "ImageStreamTag",
                    "namespace": "openshift",
                    "name": "php:7.0"
                }
            }
        },
        "output": { 
            "to": {
                "kind": "ImageStreamTag",
                "name": "php-example:latest"
            }
        },
        ...output omitted...
    },
    ...output omitted...
}
</pre>

- The runPolicy attribute defines whether a build can start simultanseously. The Serial value represents that it is not possible to build simultaneously. 

- The output attribute defines where to push the new container image after a successful build.
# Deploying Cloud-Native Applications with JKube

Cloud-native is a very general term that aims to describe technologies. These technologies are designed to build and run scalable applications in public, private, and hybrid clouds, where containers, microservices, Kubernetes, and other modern technologies serve as examples.

## Eclipse JKube
Eclipse JKube is a set of open source plugins and libraries that can build container images via different strategies, and generates and deploys Java applications to Kubernetes and OpenShift, with little to no configuration, making your applications cloud-native.

JKube, through its Maven plugins, supports several Java frameworks, such as:

### Quarkus
A modern full-stack, cloud-native framework, with a small memory footprint and reduced boot time.

### Spring Boot
A cloud-native development framework based on the popular Spring Framework and auto configuration.

### Vert.x
A reactive, low-latency development framework based on asynchronous I/O and event streams.

### Micronaut
A modern full-stack toolkit for building modular, easily testable microservices and serverless applications.

### Open Liberty
A flexible server runtime for Java applications.

## Kubernetes Maven Plug-in
JKube's Kubernetes Maven Plug-in allows the developer to generate container images, deployment descriptors and configuring deployments.

## OpenShift Maven Plug-in
Built on top of the Kubernetes Maven Plug-in, the JKube OpenShift Maven Plug-in is one of the core components of the open source Eclipse JKube project.

The configuration that follows is a minimal configuration to enable the JKube OpenShift Maven plug-in for a Java application
<pre>
...output omitted...
  <build>
    <plugins>
      <plugin> (1)
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>openshift-maven-plugin</artifactId>
        <version>1.2.0</version>
        <executions> (2)
          <execution>
            <goals>
              <goal>resource</goal>
              <goal>build</goal>
              <goal>apply</goal>
            </goals>
          </execution>
        </executions>
        <configuration> (3)
          <!-- additional configuration here -->
        </configuration>
      </plugin>
    </plugins>
  </build>
</pre>

1. Provides Maven with the required information for it to locate, download, and use the plug-in.
2. Configures the standard install goal for Maven. This is entirely optional.
3. Additional configuration, which can be more specific, allowing you to configure images, resources, volumes for the replica sets, services and config maps to fine-grain details.


## OpenShift Maven Plug-in Goals
A Maven plug-in goal represents a well-defined task in the software development life cycle process. You execute a Maven plug-in goal by using the mvn command

```
[user@host sample-app]$ mvn <plug-in goal name>
```

For OpenShift builds, the plug-in creates an application build configuration and image stream resource on the OpenShift cluster. For both Source and Docker build strategies, the plug-in updates the image stream with the new container image.

### oc:apply
Applies the generated resources to a connected OpenShift cluster.

### oc:deploy
Similar to oc:apply, except it runs in the background.

### oc:undeploy 
Removes resources from the OpenShift cluster.

### oc:watch
Watches files for changes, which then trigger a rebuild and redeployment. This is useful during development.
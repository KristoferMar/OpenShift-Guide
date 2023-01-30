### Load Balancing
Clusters provide three types of load balancers.
- An external load balancer, which manages access to the OpenShift API
- the HAProxy load balancer, for external access to applications
- The internal load balancer, which uses Netfilter rules for internal access to applications and services.

### Logging and Monitoring
RHOCP ships with an advanced monitoring solution, based on Prometheus.

### Services Discovery
RHOCP runs an internal DNS service on the cluster, and configures all containers to use that internal DNS for name resolution.


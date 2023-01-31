# Configuring Application Security

## Special topic notes
- Secret resources allow you to separate sensitive information from application pods. You expose secrets to an application pod either as environment variables or as ordinary files.

- OpenShift uses security context constraints (SCCs) to define allowed pod interactions with system resources. By default, pods operate under the restricted context which limits access to node resources.
# Docker Exercise

## Requirements
 - Google Cloud Platform account with billing enabled
 - Unix-like OS to run the shell scripts
 - kubectl installed + ~/.kube/config configured with access to a Kubernetes cluster OR Kubernetes cluster setup below
 - jq if connect to pods shell is desired
 - openssl and docker to generate a self-signed HTTPS certificate and key


## Create Kubernetes Cluster and Setup Authentication (Provider Setup)
 - Create new project at https://console.cloud.google.com/kubernetes/
 - Download gcloud SDK at https://cloud.google.com/sdk/
 - Decompress and install gcloud sdk (platform dependent)
 - Initialize the SDK (platform dependent)
   - Select to install kubectl as part of installation
 - Create a cluster for this test: gcloud container clusters create hello-ruby
 - Download credentials for use by kubectl: gcloud container clusters get-credentials hello-ruby
 - Set default authentication: gcloud beta auth application-default login

## Create Project
 - Inside project directory
 - ./create.sh
 - ./generate_https_certificate.sh
 - Get IP address of LB: kubectl describe service hello-ruby | grep Ingress | cut -d ':' -f 2
 - Connect to LB:
   - curl http://IP
   - curl -k https://IP

## Cleanup

### Project
 - Inside project directory
 - ./delete.sh

### Kubernetes Cluster and Authentication
 - gcloud container clusters delete hello-ruby
 - Delete project at https://console.cloud.google.com/kubernetes/
 - Remove gcloud SDK from local computer
 - Remove/cleanup ~/.kube/config
 - Remove permission grants for Google Cloud SDK and Google Auth Library at https://security.google.com/settings/security/permissions


## Changing Docker Prefix
In order to change the docker prefix (so the docker images can be generated using a different account), the following files need to be updated:
 - deployment.yaml
 - docker_build_image.sh
 - generate_https_certificate.sh
 - All the Dockerfile files under docker/


## Further Improvements
 - Use a real HTTPS certificate
 - Run httpd container as non-root
 - Logging
 - Metrics
 - Alerting
 - Kubernetes security http://blog.kubernetes.io/2016/08/security-best-practices-kubernetes-deployment.html
 - Redis security http://redis.io/topics/security
 - Docker security
 - Annual HTTPS certificate renewal
 - Self-building make_secret container instead of a statically generated one in case make_secret.go changes

## PS. Further Improvements
 - It would be great to have health checks on the Load balancers, not only on the pods
 - It's possible to also do SSL offloading on the LB instead of inside nginx on the pod (depending on LB used), then puma could be run listening on an HTTP socket and get rid of nginx and supervisord completely
 - If serving static files is necessary the nginx and ruby pods could be split as well, didn't have a preference and given the task description it looked more appropriate to have them in the same pod
 - Supervisord could be replaced by djb's daemontools or an alternative
 - The way I created the docker images for the pods is a bit like subclassing although maybe overkill in this situation the idea is that you might have a class Nginx with subclass HelloRuby, tagging should also be used if shared between projects

# Kubernetes, ArgoCD y Helm - FormadoresIT - Ayesa

# Kubernetes

### microK8S

Installation: https://microk8s.io/docs/install-alternatives

```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s ubuntu
newgrp microk8s
microk8s status --wait-ready
```

### kubectl

Installation: https://kubernetes.io/docs/tasks/tools/#kubectl

Kubeconfig: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

Generate and add kubeconfig for microk8s: `microk8s config > ~/.kube/config`

Create an alias for `k` to be `kubectl`: `echo "alias k=\'kubectl\'" >> .bash_aliases && source .bash_aliases`

**kubectl quick reference**: https://kubernetes.io/docs/reference/kubectl/quick-reference/



### General commands

Create / update objects using YAML file or folder: `kubectl apply -f <FILE>`

List objects `kubectl get <OBJECT>`
    
Options:

- List more details `-o wide`
- Print the YAML `-o yaml`
- Show labels `--show-labels`

Describe a specific object `kubectl describe <OBJECT> <OBJECT_NAME>`

### Pods

Pods are the basic workloads in Kubernetes. They contain one or more containers that share the same network space, meaning they share the same private IP and can communicate between containers using localhost.

https://kubernetes.io/docs/concepts/workloads/pods/

List pods: `kubectl get pod`

List with IP: `kubectl get pod -o wide`

Describe details and events: `kubectl describe pod <POD>`

Get the whole YAML file: `kubectl get pod -o yaml`

Print logs: `kubectl logs <POD> -c <CONTAINER>`

Open terminal into a container: `kubectl exec -ti <POD> -c <CONTAINER> -- bash`

Run a specific command inside container: `kubectl exec <POD> -c <CONTAINER> -- <COMMAND>`

**Add features to containers**:

- Configure probes: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

- Set resource requests and limits: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

- Set environment variables: https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

- Set command and args (entrypoint and command in Docker): https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

### Deployments (deploy)

Deployments creates and manages pods that are 100% similar. You can configure how it manages changes.

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

**Rollout commands**:

- See status and wait for completion: `kubectl rollout status deployment <DEPLOYMENT>`

- Restart all pods: `kubectl rollout restart deployment <DEPLOYMENT>`

- Print past versions: `kubectl rollout history deployment <DEPLOYMENT>`

- Rollback to a version: `kubectl rollout undo deployment <DEPLOYMENT> --to-revision <VERSION>`

ReplicaSets: https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/

### Services (svc)

Services provides a private IP that balances traffic among similar pods.

There can also open a 

https://kubernetes.io/docs/concepts/services-networking/service/

List endpoints for a service: `kubectl get endpoint <SERVICE> -o yaml>`

DNS address for internal communication with a service: `<SERVICE>.<NAMESPACE>.svc.cluster.local`


### Ingress (ing)

**Ingress Controller** is just a reverse proxy that reads all ingresses in the cluster and configure itself to route HTTP and HTTPS traffic to the services accourding to the ingresses in the cluster. It acts as a sigle entry point for all HTTP traffic to all services in the cluster:

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

There are many different options to deploy as ingress controller for the cluster


**Ingress** is the resource that contains the rules to route traffic to a given service. Usually you have one ingress per service:

https://kubernetes.io/docs/concepts/services-networking/ingress/


Everytime you create an ingress, the ingress controller reads it and configure itself to route traffic using those rules.

**Nginx Ingress Controller** is the official ingress controller for Kubernetes:

https://kubernetes.github.io/ingress-nginx/

Deploy Nginx Ingress Controller in microk8s: `microk8s enable ingress`

You can set advanced configuration in the ingresses using annotations: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/

### Configmaps (cm)

https://kubernetes.io/docs/concepts/configuration/configmap/

You can use a configmap to mount files into pods and set environment variables.

### Secrets

https://kubernetes.io/docs/concepts/configuration/secret/

Similar to configmap but it obfuscates the content. Meant to be acccessible only to admins and pods.

Special uses for secrets:

- Docker authentication: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  
- TLS certificates for ingresses: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/


### Persistence

#### Persistent Volumes (pv)

It represents a disk or storage server to be used by Kubernetes. Usually they're created atuomatically by a storage provisioner.

https://kubernetes.io/docs/concepts/storage/persistent-volumes/

#### Storage Class (sc)

It represents a storage provisioner that can create persistent volumes and provision the corresponding disk. There are provisioners for all cloud platforms and the most common storage systems.

https://kubernetes.io/docs/concepts/storage/storage-classes/

Easiest example is the storage class for a NFS server: https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs

Microk8s has its own provisioner for OpenEBS that's enabled with this script:

```
sudo apt update && sudo apt install open-iscsi
sudo systemctl enable --now iscsid

microk8s enable community
microk8s enable openebs
```

#### Persistent Volume Claim (pvc)

PVCs are a request for a persistent disk. PVCs are the resource to be used by a pod to mount a persistent volume. They specify the storage class that should provision the persistent volume for the container that mounts it. PVCs survive when the associated pod or pods are deleted.

https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims

Mount a PVC to a pod: https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

### StatefulSets (sts)

Are similar to deployments. They creates and manages pods but in this case the pods are unique. Statefulsets create pods with an unique identifier. They have an associated service that can route traffic into each pod, not only balance between them. 

Statefulsets can also contain a template to create a PVC for each pod.

Fianally, Statefulsets deploy pods one after one always.

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/


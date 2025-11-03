# Kubernetes, ArgoCD y Helm - FormadoresIT - Ayesa

- [Kubernetes, ArgoCD y Helm - FormadoresIT - Ayesa](#kubernetes-argocd-y-helm---formadoresit---ayesa)
- [Kubernetes](#kubernetes)
    - [microK8S](#microk8s)
    - [kubectl](#kubectl)
    - [General commands](#general-commands)
    - [Pods](#pods)
    - [Deployments (deploy)](#deployments-deploy)
    - [DeamonSets (ds)](#deamonsets-ds)
    - [Jobs](#jobs)


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

### DeamonSets (ds)

Deploys 1 pod on all nodes in the cluster

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

### Jobs

Run a pod that runs a task meant to have an end. Once the tasks inide the containers ends with success, it's marked as completed and does not run anymore. If the task fails, it retries with a configurable policy.

https://kubernetes.io/docs/concepts/workloads/controllers/job/

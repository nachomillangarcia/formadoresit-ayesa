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
    - [Cronjobs](#cronjobs)
    - [Services (svc)](#services-svc)
    - [Ingress (ing)](#ingress-ing)
    - [Configmaps (cm)](#configmaps-cm)
    - [Secrets](#secrets)
    - [StatefulSets (sts)](#statefulsets-sts)
    - [Persistence](#persistence)
      - [Persistent Volumes (pv)](#persistent-volumes-pv)
      - [Storage Class (sc)](#storage-class-sc)
      - [Persistent Volume Claim (pvc)](#persistent-volume-claim-pvc)
    - [RBAC (Role Based Access Control)](#rbac-role-based-access-control)
      - [Service Accounts (sa)](#service-accounts-sa)
      - [Roles and Clusterroles](#roles-and-clusterroles)
      - [RoleBindings and ClusterRoleBindings](#rolebindings-and-clusterrolebindings)
- [Helm](#helm)
    - [Installation](#installation)
    - [Core concepts](#core-concepts)
    - [Commands](#commands)
    - [Helm Charts](#helm-charts)
      - [Template syntax](#template-syntax)
      - [Hooks](#hooks)
    - [Managing Repositories](#managing-repositories)
      - [Classic repositories](#classic-repositories)
      - [OCI repositories](#oci-repositories)
      - [Helm chart tests](#helm-chart-tests)
    - [Helm CLI plugins](#helm-cli-plugins)


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

### Cronjobs

A cronjob creates jobs according to a specific schedule.

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

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


### StatefulSets (sts)

Are similar to deployments. They creates and manages pods but in this case the pods are unique. Statefulsets create pods with an unique identifier. They have an associated service that can route traffic into each pod, not only balance between them. 

Statefulsets can also contain a template to create a PVC for each pod.

Fianally, Statefulsets deploy pods one after one always.

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/



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

### RBAC (Role Based Access Control)

#### Service Accounts (sa)

A service account is just a token a pod (or many) can use to communicate with Kubernetes API

https://kubernetes.io/docs/concepts/security/service-accounts/

To define a new service account:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: my-namespace
```

To use the service account in a deployment, for example:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        ...
      volumes:
        ...
      serviceAccountName: nginx
```

The token from the service account is mounted on the path `/var/run/kubernetes.io/`  along with the root certificate (`ca.crt`) to communicate with the Kubernetes API. Containers within the Kubernetes cluster can user the address `kubernetes.default.svc.cluster.local` as the endpoint.

#### Roles and Clusterroles

Are a list of permissions. Roles are namespaced while clusterroles are cluster-wide

https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]          # Same as apiVersion field for the object we want to grant permissions, without the version. "" means v1
  resources: ["secrets"]   # Object names over which grant permissions
  verbs: ["get", "watch", "list"]  # List of verbs
```

#### RoleBindings and ClusterRoleBindings

Grant a single role to one or more users/groups/service accounts. Rolebindings are namespaced while clusterRoleBindings are cluster-wide

https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-binding-examples

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: ServiceAccount       # this must be User, Group or ServiceAccount
  name: my-serviceaccount 
  apiGroup: rbac.authorization.k8s.io
roleRef: 
  kind: Role                 # this must be Role or ClusterRole
  name: secret-reader        # name of the Role of ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

# Helm

### Installation

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

https://helm.sh/docs/intro/install/

### Core concepts

- **Chart**: An application to be deployed using Helm
- **Release**: A chart deployed to Kubernetes
- **Values**: Variables used to customize a release
- **Repository**: An online source for charts. Always use official repositories

### Commands

Reference: https://helm.sh/docs/intro/using_helm/


Repositories:

- Add a new repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`
- Update all repositories: `helm repo update`

Install / Upgrade:

- `helm install <RELEASE NAME> <REPOSITORY>/<CHART>`
- Upgrade: `helm upgrade <RELEASE NAME> <REPOSITORY>/<CHART>`
- Use a local chart `helm install/upgrade <RELEASE NAME> <CHART FOLDER>`
- Specify namespace `-n namespace`
- Specify values file `--values <FILE>`

Print redered YAMLs without actually deploying:
- `helm template <RELEASE NAME> <REPOSITORY>/<CHART>`

List releases:

- `helm ls -n <NAMESPACE>`

List revisions for a release:

- `helm history -n <NAMESPACE> <RELEASE NAME>`

Print values:

- `helm get values -n <NAMESPACE> <RELEASE NAME>`
- Print all values including defaults `--all`
- Specific revision `--revision <REVISION NUMBER>`

Print Kubernetes YAMLs:

- `helm get manifest -n <NAMESPACE> <RELEASE NAME>`
- Print all values including defaults `--all`
- Specific revision `--revision <REVISION NUMBER>`


### Helm Charts

Chart structure and common files: https://helm.sh/docs/topics/charts/

Kickstart a new chart: https://helm.sh/docs/chart_template_guide/getting_started/

Dependencies and sub-charts: https://helm.sh/docs/chart_best_practices/dependencies/

#### Template syntax

Generealities: https://helm.sh/docs/chart_best_practices/templates/

Using built-in variables as `.Release` and `.Chart`: https://helm.sh/docs/chart_template_guide/builtin_objects/

Using Values `{{ .Values.... }}`: https://helm.sh/docs/chart_template_guide/values_files/

Conditionals (`if`): https://helm.sh/docs/chart_template_guide/control_structures/#ifelse

```
# Condition for Value being true

{{- if .Values.... }}

{{- end }}

# Compare a Value. In this example, to be equal to other part.
# You can use the operators [eq, ne, lt, gt, and, or] into conditional statements

{{- if eq .Values... <EQUAL VALUE>}}

{{- end }}
```

Looping (`range`): https://helm.sh/docs/chart_template_guide/control_structures/#looping-with-the-range-action

```
# Over a list:

{{- range .Values.deployment.volumeMounts }}
  - name: {{ .name | quote }}
    mountPath: {{ .path | quote }}
{{- end }}

# Over a dictionary

{{- range $name, $value := .Values.deployment.annotations }}
{{ $name }}: {{ $value }}
{{- end }}
```

Modifying the scope (`with`): https://helm.sh/docs/chart_template_guide/control_structures/#modifying-scope-using-with

```
# Set the new scope to .Values.Service
  {{- with .Values.service}}
  type: {{ .type | default "ClusterIP" }}
    {{- if eq .type "NodePort" }}
    nodePort: {{ .nodePort }}
    {{- end }}
    targetPort: http
  {{- end }}
{{- end }}
```

Note that you can always access to the root scope using the reserved variable `$`: `$.Release.Name`

Define variables (`{{- $relname := .Release.Name -}}`): https://helm.sh/docs/chart_template_guide/variables/

Functions: https://helm.sh/docs/chart_template_guide/functions_and_pipelines/

```
# Use a functions with parameters

{{ quote .Values.favorite.drink }}  # This adds quotes in the extremes of the value

# Use a function like pipeline

{{ .Values.favorite.food | upper | quote }} # This makes the value uppercase, and then adds quotes
```

Full functions list: https://helm.sh/docs/chart_template_guide/function_list/

#### Hooks

Hooks allow to deploy some objects before or after all the others.

We define hooks adding annotations to the objects we want Helm to deploy before or after:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade   # Deploy this configmap before other resources when installing or upgrading
    "helm.sh/hook-weight": "-5"               # Set a weight to control the order among all the other objects with the same hook
```

Default deploy order in Helm:  https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package

All different hooks and reference: https://helm.sh/docs/topics/charts_hooks/#helm

### Managing Repositories

#### Classic repositories

https://helm.sh/docs/topics/chart_repository/

Helm repositories are merely web servers with the chart files and an index.yaml file that Helm understands to retrieve the charts.

To create a helm repository:

```
helm repo package <PATH TO CHART>     # Create a .tgz file with the chart 
mv <TGZ FILE> ./<REPOSITORY FOLDER>   # Move it to the folder for the repo
helm repo index ./<REPOSITORY FOLDER> # Generate or update index file
<upload the repository folder to a file server>
```

#### OCI repositories

https://helm.sh/docs/topics/registries/

New OCI repositories doesn't require all the steps before. We can just push our chart to a OCI-compliant registry (DockerHub, GitHub, etc.)

First we need to log in to the registry using `helm login`

The we can just push the chart to the registry:

``` helm push <.tgz file>```

#### Helm chart tests

https://helm.sh/docs/topics/chart_tests/

We can set up pods that runs tests for our helm charts. For example, to check if connection to service is up. To do that, we create a new YAML in the /templates folder with a pod resource that run the tests, and add the annotation `"helm.sh/hook": test`.

To run the test pods in a realese, just run the command `helm test <REALEASE>`. Tests are meant to run during development.

### Helm CLI plugins

There are plugins that add funcionalities to the `helm` CLI. To install a plugin, use the command `helm plugins install <GITHUB REPO FOR THE PLUGIN>`.

Here is a full list of useful helm plugins: https://helm.sh/docs/community/related/#helm-plugins

In the course we installed the plugin helm diff: https://github.com/databus23/helm-diff


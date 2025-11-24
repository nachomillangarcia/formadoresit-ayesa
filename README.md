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
- [Log management with ElasticSearch stack](#log-management-with-elasticsearch-stack)
    - [Installing](#installing)
      - [ElasticSearch cluster](#elasticsearch-cluster)
      - [Kibana](#kibana)
    - [Filebeat](#filebeat)
      - [Deploying](#deploying)
      - [Config example](#config-example)
      - [Annotations](#annotations)
      - [References](#references)
    - [Kibana](#kibana-1)
    - [Metricbeat](#metricbeat)
      - [Deploying](#deploying-1)
- [Monitoring with Prometheus and Grafana](#monitoring-with-prometheus-and-grafana)
    - [Deploying](#deploying-2)
    - [Queries](#queries)
      - [Selectors](#selectors)
      - [Functions](#functions)
      - [Operators](#operators)
    - [Metric types](#metric-types)
      - [Gauge](#gauge)
      - [Counter](#counter)
      - [Histogram](#histogram)
      - [Summary](#summary)
    - [Configuration](#configuration)
    - [Metrics sources (jobs and ServiceMonitor)](#metrics-sources-jobs-and-servicemonitor)
    - [Rules (PrometheusRule)](#rules-prometheusrule)
      - [Recording rules](#recording-rules)
      - [Alerting rules](#alerting-rules)
    - [Alert Mamager](#alert-mamager)
      - [Alert Manager Discovery](#alert-manager-discovery)
      - [Configuration](#configuration-1)
    - [Exporters](#exporters)
    - [Thanos](#thanos)
    - [Grafana](#grafana)
      - [Connect with Prometheus](#connect-with-prometheus)
      - [Import dashboards for node-exporter and kube-state-metrics](#import-dashboards-for-node-exporter-and-kube-state-metrics)
      - [Create new dashboards](#create-new-dashboards)
      - [Variables](#variables)
- [Autoscaling](#autoscaling)
    - [Node Autoscaling](#node-autoscaling)
    - [Pod autoscaling](#pod-autoscaling)
      - [Horizontal Pod Autoscaler (hpa)](#horizontal-pod-autoscaler-hpa)
        - [Basic autoscaling with CPU and memory metrics](#basic-autoscaling-with-cpu-and-memory-metrics)
      - [KEDA](#keda)
    - [kube-green](#kube-green)
- [Cert Manager](#cert-manager)
    - [Installing](#installing-1)
    - [Creating a ClsterIssuer (or namespaced Issuers)](#creating-a-clsterissuer-or-namespaced-issuers)
- [External Secrets Operator](#external-secrets-operator)
    - [Installing](#installing-2)
    - [SecretsStore and ClusterSecretsStore](#secretsstore-and-clustersecretsstore)
    - [ExternalSecret and ClusterExternalSecrets](#externalsecret-and-clusterexternalsecrets)
    - [Alternatives](#alternatives)


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

# Log management with ElasticSearch stack

### Installing

#### ElasticSearch cluster

First install the ECK operator (https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s) :

```
helm repo add elastic https://helm.elastic.co
helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace
```

Then we create the elastic namespace and a secret with the user and password:

```
kubectl create ns elastic
kubectl -n elastic create secret generic elasticsearch-es-elastic-user  --from-literal=elastic=elastic
```

And deploy the YAML with the file `Elasticsearch` object:

```
kubectl apply -f elasticsearch/elasticsearch.yaml
```


#### Kibana

```
kubectl apply -f elasticsearch/kibana.yaml
```

### Filebeat

Filebeat is an agent that reads log files from plenty of different sources, process them line by line, and push the processed objects to ElsticSearch for storage and querying.

Filebeat comes with pod autodiscovery to be used in Kubernetes easily:

https://www.elastic.co/docs/reference/beats/filebeat/configuration-autodiscover

#### Deploying

```
kubectl apply -f elasticsearch/filebeat.yaml
```

This deploys filebeat in a daemonset with the default configuration for Kubernetes.

#### Config example

```      
  config:
    http:                   # Enable HTTP port for metrics
      enabled: true
      host: 0.0.0.0
      port: 5066
    filebeat:
      autodiscover:                 # Configures Kubernetes hints-based autodiscovery
        providers:
        - type: kubernetes
          node: ${NODE_NAME}
          hints:
            enabled: true
            default_config:         # Default config if no other templates matches the pod
              type: filestream
              id: container-${data.container.id}
              paths:
              - /var/log/containers/*${data.kubernetes.container.id}.log       # Path to the logs folder for the node
              parsers:
              - container: {}               # Parse the content of the logs files with the container module
              close.inactive: 5m
              close.removed: true
              close.on_state_change.removed: false
              ignore_older: 30m
              clean_inactive: 48h
              prospector:
                scanner:
                  fingerprint.enabled: false
                  symlinks: true
                  check_interval: 10s        # how often to rescan
                  exclude_files: ['\.gz$']   # skip rotated files
```

#### Annotations

Hints-based autodiscovery allows a set of annotations to be added to the pods so we can modify the behaviour of filebeat for those pods: https://www.elastic.co/docs/reference/beats/filebeat/configuration-autodiscover-hints

To monitor Nginx using filebeat Nginx module:

```
        co.elastic.logs/module: "nginx"
        co.elastic.logs/fileset.stdout: "access"
```

To parse logs from CoreDNS pod using the disect processor:

```
    co.elastic.logs/processors.dissect.tokenizer: '[%{log.level}] %{client_ip}:%{client_port} - %{connection_id} "%{query.class} IN %{query.name} %{query.type} %{qid} false %{length}" %{rcode} %{flags} %{number} %{response_time}s'
    co.elastic.logs/processors.dissect.field: "message"
    co.elastic.logs/processors.dissect.ignore_failure: "true"
    co.elastic.logs/processors.dissect.target_prefix: "coredns"
```

#### References

- Filebeat: https://www.elastic.co/docs/reference/beats/filebeat/
- Inputs: https://www.elastic.co/docs/reference/beats/filebeat/configuration-filebeat-options
  - Container input: https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-container
- Modules: https://www.elastic.co/docs/reference/beats/filebeat/filebeat-modules
  - Nginx module: https://www.elastic.co/docs/reference/beats/filebeat/filebeat-module-nginx
- Processors: https://www.elastic.co/docs/reference/beats/filebeat/filtering-enhancing-data
  - Dissect processor: https://www.elastic.co/docs/reference/beats/filebeat/dissect


### Kibana

Kibana is the web dashboard to use with ElasticSearch.

Once we've deployed filebeat to push the first documents to ElasticSearch, we can enter Kibana, go to `discover`  and create a new **data view** to start querying the documents: https://www.elastic.co/docs/explore-analyze/find-and-organize/data-views

Then we can start using the **Discover** page to explore the data: https://www.elastic.co/docs/explore-analyze/discover/discover-get-started

A very useful feature briefly explored in the course are Dashboards, in which you can create visualizations from queries: https://www.elastic.co/docs/explore-analyze/dashboards

### Metricbeat

Metricbear is an agent that pushes metrics from different sources to ElasticSearch.

#### Deploying

```
kubectl apply -f elasticsearch/filebeat.yaml
```

This install the default sources for Kubernetes:
-  A DaemonSet that pushes metrics about nodes and container status (CPU, memory, network, disk, etc.)
-  kube-state-metrics, which is a generic agent for Kubernetes that exposes metrics about different objects and their status (deployments, jobs, statefulsets, pods, etc.)
-  A deployment that reads metrics from kube-state-metrics and pushs them to elasticsearch

Reference: https://www.elastic.co/docs/reference/beats/metricbeat/metricbeat-module-kubernetes

Personally, I'm not a fan of metrics in ElasticSearch, it's a better approach to use Prometheus and Grafana for everything about metrics in Kubernetes.

# Monitoring with Prometheus and Grafana

Prometheus is the default metrics server in Kubernetes. All components in a Kubernetes cluster expose metrics in the format defined for Prometheus. 

Grafana is the web dashboard that integrates easily with Prometheu and many other data sources.

### Deploying

Using Prometheus Operator (https://prometheus-operator.dev/) and the helm chart https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

```
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --values helm/prometheus-values.yaml
```

This will install:
- Prometheus operator and the necessary CustomResourceDefinitions
- It creates an object `kind: Prometheus` with the passed specification. The operator then deploys the Prometheus cluster according to the spec.
- Same for `kind: Alertmanager` for the alertmanager
- A daemonset for nodeexporter (https://github.com/prometheus/node_exporter) which exposes metrics about nodes health
- A deployment for kube-state-metrics which is a generic agent for Kubernetes that exposes metrics about different objects and their status (deployments, jobs, statefulsets, pods, etc.)
- A deployment for Grafana
- A bunch of default monitors to tell Prometheus to monitor all those components by default.

### Queries

#### Selectors

https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors

Prometheus metrics can be selected using its labels

```
container_memory_working_set_bytes{image!="registry.k8s.io/pause:3.10", container!="", namespace="default",pod=~"nginx-deployment-.*"}
```

This query excludes images equal to "registry.k8s.io/pause:3.10", excludes results where container label is empty, selects only the results with namespace label = default and with pod label similar to nginx-deployment

#### Functions

https://prometheus.io/docs/prometheus/latest/querying/functions/

We can apply functions to metrics to make calculations, averages, etc

```
rate(container_cpu_usage_seconds_total{image!="registry.k8s.io/pause:3.10", container!="", namespace="default",pod=~"nginx-deployment-.*"} [5m])
```

Calculates the rate of change for the metric container_cpu_usage_seconds_total

#### Operators

https://prometheus.io/docs/prometheus/latest/querying/operators

There are arithmetic operators (+, -, *, /, etc) and comparison operators (>, <, ==, etc)

```
container_memory_working_set_bytes{image!="registry.k8s.io/pause:3.10", container!="", namespace="default",pod=~"nginx-deployment-.*"} / (1024*1024)
```

Divides the metric by (1024*1024) to convert it to Mebibytes

We can add modifiers to operators to help matching labels between metrics:

https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching

```
container_memory_working_set_bytes{image!="registry.k8s.io/pause:3.10", container!="", namespace="default",pod=~"nginx-deployment-.*"} / on(pod, namespace, container) kube_pod_container_resource_limits{resource="memory"}
```

Divides memory usage for containers by the memory limit for the container, and does the matching using only the labels (pod, namespace, container)


### Metric types

#### Gauge

https://prometheus.io/docs/concepts/metric_types/#gauge

Represents the exact value at the time when the metric was scraped. i.e. memory usage at that time, disk usage at that time, etc

It can go up and down.

We don't need to do any other operations to make use of the value. It already represents the information we're seeking

#### Counter

https://prometheus.io/docs/concepts/metric_types/#counter

Counter only goes up. It's meant to represent a metric that's always increasing, as CPU usage or requests. Metrics are created as counters to give information not only about the moment like the gauges for memory usage, but to give information about how it has increased over time.

We use rate functions like rate, irate, delta, idelta to extract information about how the counter metric has changed, so we get requests per second or core usage per second, instead of total requests or total cpu usage.

```
rate(container_cpu_usage_seconds_total{image!="registry.k8s.io/pause:3.10", container!="", namespace="default",pod=~"java-.*"} [5m])
```

`container_cpu_usage_seconds_total` is the total cpu seconds since the pod started. rate() transforms it in the average of seconds used in the last 5 min


#### Histogram

https://prometheus.io/docs/concepts/metric_types/#histogram

https://prometheus.io/docs/practices/histograms/

Are counters that are disgregated in different buckets, under the label `le` 

```
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.00025"} 257
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.0005"} 464
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.001"} 495
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.002"} 495
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.004"} 548
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.008"} 561
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.016"} 610
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.032"} 613
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.064"} 614
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.128"} 614
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.256"} 614
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="0.512"} 723
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="1.024"} 729
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="2.048"} 729
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="4.096"} 730
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="8.192"} 730
coredns_dns_request_duration_seconds_bucket{server="dns://:53",view="",zone=".",le="+Inf"} 730
coredns_dns_request_duration_seconds_sum{server="dns://:53",view="",zone="."} 61.89406280700003
coredns_dns_request_duration_seconds_count{server="dns://:53",view="",zone="."} 730
```

In `coredns_dns_request_duration_seconds_bucket` each one of these metrics represents how many queries has been served below `<le>` seconds.

Histograms always contain +Inf le meaning the total of requests served. They include also a sum and count.

We use this metrics along with the function histogram_quantile to get percentiles.

```
histogram_quantile(0.95, sum(idelta(coredns_dns_request_duration_seconds_bucket[5m])) by (le))
```

#### Summary

https://prometheus.io/docs/concepts/metric_types/#summary

Similar to histogram, but it already provides the percentiles so we don't need to apply the histogram_quantile function

### Configuration

Prometheus config file contains the parameters to tell the server. We can provide a config file, but the Prometheus Operator defines custom object that will add configs to the server automatically.

https://prometheus.io/docs/prometheus/latest/configuration/configuration/

https://prometheus-operator.dev/docs/api-reference/api/


### Metrics sources (jobs and ServiceMonitor)

Prometheus scrape metrics from the configured targets every 1 minute (by default). A job defines a target to be scraped.

In a setup with Proemtheus Operator as ours, we configure jobs creating a resource of kind ServiceMonitor: https://prometheus-operator.dev/docs/api-reference/api/#monitoring.coreos.com/v1.ServiceMonitor

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  endpoints:
  - port: metrics

  selector:
    matchLabels:
      app: nginx
  namespaceSelector:
    matchNames:
    - default
```

A ServiceMonitor configures Prometheus to scrape metrics from endpoints (pods in Ready state that receives traffic from a service) that match the selector values.

We can see the configured targets, with their URLs and health status in the we UI, in the targets section.

The label `job` tells what serviceMonitor or job has provided the metric.

### Rules (PrometheusRule)

Rules are pre-defined queries that run periodically in the server. We can store the result in a new metric to be used (Recording Rules) or use the rule to trigger an alert (Alerting Rules).

#### Recording rules

https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/

```
    groups:
      - name: node-requests.rules
        rules:
        - record: node:memory_requests:ratio
          expr: |-
            (sum by (node) (kube_pod_container_resource_requests{resource="memory"})/ 1024 / 1024) 
            /
            (sum by (node) (kube_node_status_allocatable{resource="memory"})/ 1024 / 1024) * 100
```

Recording rules contain a `expr` with the query to be run periodically and a `record` to store the result. We can then use the record as a metric anywhere.

#### Alerting rules

https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/

```      
        rules:
        - alert: NodeMemoryRequestsHigh
          annotations:
            description: |
              {{ $labels.node }} node memory requests are over 3%, currently at {{$value }}%
            summary: Node has high memory requests
          expr: node:memory_requests:ratio > 3
          for: 5m
          labels:
            severity: warning
            node: |-
              {{ $labels.node }}
```

Alerting rules contain a `expr` field with the query to be run. Also

- `for` is how long it'll wait to trigger the alert
- labels and annotations to add info to the alert triggered
  
This alert will triger if the `expr` returns any value for `5m`

If we have a Prometheus operator, we add rules by creating a file of kind `prometheusRule`: https://prometheus-operator.dev/docs/api-reference/api/#monitoring.coreos.com/v1.PrometheusRule


### Alert Mamager

AlertManager is the component of Prometheus that receives triggering alerts from one or many Prometheus servers and notify configured receivers.

#### Alert Manager Discovery

Promtheus can auto-discover alert managers to interact with, by using this section in Prometheus configuration: https://prometheus.io/docs/prometheus/latest/configuration/configuration/#alertmanager_config

#### Configuration

https://prometheus.io/docs/alerting/latest/configuration

All features in alert manager are set in the configuration file. Sections

- global: global settings and endpoints for different receivers
- routes: list of rules to match triggering alerts with the coresponding receiver
- receivers: list of receivers to be used in routes
- inhibitors: rules to suspend alerts if there are other alerts that meet the conditions (i.e. if there are critical alerts, don't send warnings)

### Exporters

As not every software is made to expose metrics for Prometheus, there are exporters that make this work for us. Here is a full list of available exporters: https://prometheus.io/docs/instrumenting/exporters/

We deployed Nginx exporter (https://hub.docker.com/r/nginx/nginx-prometheus-exporter) in the `deploment-nginx.yaml` file and added the corresponding service monitor.

### Thanos

Thanos is a layer on top of Prometheus that allows to use many Prometheus clusters at the same time, for example to connect clusters in different regions. It also allows to store matrics data in object storage systems as S3 or Blob Storage to keep metrics long-term.

Componenets:

https://thanos.io/tip/thanos/quick-tutorial.md/#components

- Sidecar: it's a container that runs in the same pod as every Prometheus server we want to connect. This component read all metrics from the Prometheus server and pushes it to our long-term storage. It also servers metrics or other components.
- Querier: Similar to the Prometheus UI for queries, but Thanos querier runs the queries against all connected Prometheus servers, connecting with the sidecar.
- Ruler: Same as querier but for rules. It runs rules agains all Prometheus servers and store the result on the long-term storage, or connects to an alert manager for alerting.
- Store Gateway: Allow all components to make queries agains the long-term storage bucket
- Compact: Optimise the objects in the long-term storage bucket.

To deploy Thanos we start with the sidecar, which we can deploy using the Helm chart we used for Prometheus, configuring the values.

Then we can deploy the ruler in the same way since it's supported by Prometheus operator. We also deployed querier by using a YAML file with all it's components

All other components must be deployed using regular YAML files: https://github.com/thanos-io/kube-thanos/tree/main/examples/all/manifests

### Grafana

#### Connect with Prometheus

1st create a data source for Prometheus, connect with the deployed cluster using `http://prometheus-kube-prometheus-prometheus.svc.cluster.local:9090`

#### Import dashboards for node-exporter and kube-state-metrics

We can jump into dashboards by importing existing ones available in https://grafana.com/grafana/dashboards/

One example is https://grafana.com/grafana/dashboards/1860-node-exporter-full/, we can import it in New > Import and use the ID for he dashboard.

#### Create new dashboards

https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/

We can add new visualizations for our queries. Each visualization can contain more than one query. It's important to:

* Configure the units of the visualization
* Configure the legend for each query so it's readeable. We can us labels with `{{ label }}` in the legend.

#### Variables

A central feature of dashboards is setting up variables to select what resources are plotted in the dashboard.

https://grafana.com/docs/grafana/latest/dashboards/variables/

Variables can get the list of options dinamically using prometheus queries. Usually we will want to use the Query type: Label values, and the metric `kube_pod_info` to get a list of pods and namespaces in the cluster. We can use similar queries for nodes, deployments, etc. See the example dashboard.

Variables can be ordered and used in the queries that set up other variables, so we get for example only the pods that belongs to the selected namespace.

Then, we can use the variables in the queries for the visualizations: `$variable_name` 



# Autoscaling

There are two different dimensions of autoscaling on Kubernetes: node autoscaling and pod autoscaling.

### Node Autoscaling

The Cluster Autoscaler controller can connect with your cloud provider and scale up and down nodes depending on the resource requests of the pods in the cluster. When a pod can't be scheduled because of lacking resources (CPU or memory) in the cluster, the cluster autoscaler checks if scaling up instances in your cloud provider could fit the Pending pods. In that case, it triggers a new instance that will join the cluster. It can also scale down if there are resources to spare.

It's a key component of any Kubernetes cluster in cloud.

https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler

### Pod autoscaling

The other dimeension of Kubernetes scaling is pod scaling based on given metrics.

#### Horizontal Pod Autoscaler (hpa)

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

Is the basic object to configure pod autoscaling on Kubernetes. In a hpa we set two fields:

- **scaleTargetRef** defines the object to be scaled (deployment, statefulset, etc)
- **metrics** defines de metrics and targets

We can also configure settings about `behavior` to scale up and down.

```
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

`target` is the target for this metric **per pod**. With that example, we set that the average memory usage amoing all pods in the deployment must be <= 80% of the requests.

##### Basic autoscaling with CPU and memory metrics

CPU and memory metrics are available to Kubernetes API to be used in hpa's easily wiht no further integrations. To use those basic metrics, we set the `type: Resource` in the metric.

To use `Resource` metrics we first need to deploy the **metrics server** to the cluster (https://github.com/kubernetes-sigs/metrics-server). In microk8s, we just run `microk8s enable metrics-server

Once metrics-server is up, we can query pod metrics using `kubectl get podmetrics`. Those are the metrics that will be used in the hpa for CPU and memory.

Hpa also allows to use custom metrics. But we need to deploy other controllers that get those metrics for us and make them available fro autoscaling. The best project to use cursom metrics for the autscaler is KEDA:

#### KEDA

KEDA allows integration of any kind of metrics into hpa objects. It defines a new object, `ScaledObject`, that connects with Prometheus (or any other provider) to get the metrics and configures a hpa based on those metrics.

Installation:

```
helm repo add kedacore https://kedacore.github.io/charts  
helm install keda kedacore/keda --namespace keda --create-namespace --values helm/keda-values.yaml
```

ScaledObject:

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: nginx-scaledobject
  labels:
    app: nginx
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicaCount: 2
  maxReplicaCount: 10

  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090
      metricName: nginx_http_requests_total
      threshold: '30'
      #activationThreshold: '1'
      query: sum(rate(nginx_http_requests_total{namespace="default"}[5m]))
```

It defines scaleTargetRef and other options similar to hpa. But in `triggers` we retrieve a metric from our Prometheus server and seet the target (`threshold`). It works the same way as target for hpa.

The hpa object resulting from this ScaledObject is like

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  ...
spec:
  maxReplicas: 10
  minReplicas: 1
  metrics:
  - external:
      metric:
        name: s0-prometheus
        selector:
          matchLabels:
            scaledobject.keda.sh/name: nginx-scaledobject
      target:
        averageValue: "30"
        type: AverageValue
    type: External
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
```

The external metrics `s0-prometheus` is provided by `keda-operator-metrics-apiserver` with the metric we defined.

References:

- Installing: https://keda.sh/docs/2.18/deploy/
- ScaledObject: https://keda.sh/docs/2.18/reference/scaledobject-spec/
- All available scalers: https://keda.sh/docs/2.18/scalers/

In the course we also integrated KEDA self metrics in Prometheus and deployed these dashboards to Grafana:

- https://grafana.com/grafana/dashboards/23951-kubernetes-autoscaling-keda-scaled-object/
- https://grafana.com/grafana/dashboards/22128-horizontal-pod-autoscaler-hpa/

### kube-green

kube-green is a controller for Kubernetes that allows to set an schedule to downscale resources (weekends, out of work, etc).

Installing:

```
helm repo add kube-green https://kube-green.github.io/helm-charts/
helm install kube-green kube-green/kube-green --namespace kube-green --create-namespace --values helm/kube-green-values.yaml
```

kube-green sefines the object `SleepInfo` to configure schedules and targets for our resources (deployments, statefulsets, etc)

```
apiVersion: kube-green.com/v1alpha1
kind: SleepInfo
metadata:
  name: test
spec:
  weekdays: "*"
  sleepAt: "18:03"
  # sleepAt: "03 18 * * *"
  wakeUpAt: "07:00" # Next day
  # wakeUpAt: "05 18 * * *"  # Same day
  timeZone: "Europe/Madrid"
  suspendCronJobs: false
  suspendDeployments: true
  suspendStatefulSets: true
  includeRef:
    - matchLabels: 
        app: nginx
```

We set `weekdays` with our working days, `sleepAt` with the time to run the sleep command, `wakeUpAt` with the time to run the wake up command. THey accept an hour or a cron expression (recommeneded for fine control).

`includeRef` tells kube-green what resources to scale down and back up.

kube-green only accepts deployments, statefulset and cronjobs. But we can define `patches` to modify other resources:

```  patches:
    - target:
        group: keda.sh
        kind: ScaledObject
      patch: |-
        - path: /spec/minReplicaCount
          op: add
          value: 0
        - path: /spec/maxReplicaCount
          op: add
          value: 0
```

In the example, we tell kube-green to modify my keda object to set the maxReplicas and minReplicas to zero, so KEDA doesn't scale up the deployment during sleep hours.

References:

Installing: https://kube-green.dev/docs/installation/install/
SleepInfo: https://kube-green.dev/docs/configuration/

# Cert Manager

https://cert-manager.io/

Cert manager is a controller that automatically issues certificates for the domains we define in the ingresses.

It integrates with many Certificate Authorities.

### Installing

```
helm repo add jetstack https://charts.jetstack.io --force-update

helm install   cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.19.1   --values helm/cert-manager-values.yaml
```

### Creating a ClsterIssuer (or namespaced Issuers)

ClusterIssuer defines the integration with our Certificate Authority

In the course we first created a ClusterIssuer to create a self-signed CA root certificate:

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-bootstrap
spec:
  selfSigned: {}

---

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: local-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: local-dev-ca
  secretName: local-ca-secret
  issuerRef:
    name: selfsigned-bootstrap
    kind: ClusterIssuer
```

Then we create the clusterIssuer that will be used for the certificates for the domains in ingresses:

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: local-ca-issuer
spec:
  ca:
    secretName: local-ca-secret
```

To issue a certificate for an Ingress, we set an annotation and just add the domains to the tls section:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: http-ingress
  annotations:
    cert-manager.io/cluster-issuer: "local-ca-issuer"
spec:
  ingressClassName: nginx-public
  rules:
  - host: nginx.127.0.0.1.sslip.io
    http:      
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
  tls:
    - hosts:
        - nginx.127.0.0.1.sslip.io
      secretName: my-app-tls
```

Cert manager will create an object `Certificate` for `nginx.127.0.0.1.sslip.io`. The certificate data is then stored in the referenced secret `my-app-tls`, and Ingress will use it for the requests to that domain.

# External Secrets Operator

External Secrets Operator synchronises secrets stored in an external source (AWS secrets manager, etc) to secrets in the Kubernetes cluster.

### Installing

```
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```

### SecretsStore and ClusterSecretsStore

https://external-secrets.io/latest/api/secretstore/

Defines the integration with the external secrets source:

```
apiVersion: v1
kind: Secret
metadata:
  name: awssm-secret
data:
  access-key: <AWS ACCESS KEY>
  secret-access-key: <AWS SECRET KEY>
---

apiVersion: external-secrets.io/v1
kind: SecretStore
metadata:
  name: aws-secretsmanager
spec:
  provider:
    aws:
      service: SecretsManager
      region: eu-central-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: awssm-secret
            key: access-key
          secretAccessKeySecretRef:
            name: awssm-secret
            key: secret-access-key
```

In the example we store the relevant credentials in a Secret, and then create the SecretStore for AWS secrets manager, referencing the secret for credentials.

### ExternalSecret and ClusterExternalSecrets

https://external-secrets.io/latest/api/externalsecret/

Describes what data should be fetched, how the data should be transformed and saved as a Secret in Kubernetes-

```
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: example
spec:
  refreshInterval: 12h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: secret-to-be-created
    creationPolicy: Owner
  dataFrom:
  - extract:
      key: formadoresit 
```

The example extracts the data stored in the secret named `formadoresit` in AWS secrets manager, and stores the data in the target Kubernetes secret `secret-to-be-created`

We can then mount the secret in the pods.

### Alternatives

Storing credentials in Kubernetes secrets is still not so secure, as those are not encrypted.

Ideally our pods should read directly secrets from the external source, bypassing kubernetes secrets completely.

To automate this process we can use Secrets Store CSI driver: https://secrets-store-csi-driver.sigs.k8s.io/

It's a controller that creates a new kind of volume that we can mount into the pods. At runtime, this kind of volume retrieves the secrets for the external source and mounts them into the pod, with no extra objects.


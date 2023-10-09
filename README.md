## Title ##
How to Specify a Disruption Budget for your Kubernetes Application

## Meta-description ##
Learn what a pod disruption budget is, why it is important, and how to create and use one for your Kubernetes application to ensure high availability and resilience.

## Introduction ##
Kubernetes is a powerful platform for deploying and managing containerized applications at scale. However, Kubernetes clusters are not immune to disruptions, such as node failures, maintenance operations, or scaling events. These disruptions can affect the availability and performance of your applications, especially if they are not designed to handle them gracefully.

To help you protect your applications from disruptions, Kubernetes offers a feature called pod disruption budget (PDB). A PDB is an API object that specifies the minimum number or percentage of pods that must be available for a set of pods during a disruption. By creating a PDB, you can ensure that your application has enough replicas to function properly, even when some pods are evicted or rescheduled.

In this article, we will explain what a PDB is, why it is useful, and how to create and use one for your Kubernetes application. We will also share some best practices and examples of PDBs in action.

## Outline ##
- What is a pod disruption budget (PDB)?
  - Definition and purpose of a PDB
  - Types of disruptions: voluntary and involuntary
  - How PDBs work with Kubernetes controllers and evictions
- How to create a PDB for your application
  - Determine the minimum number of instances or availability percentage
  - Create a YAML file with the PDB definition
  - Apply the YAML file with kubectl
- How to use a PDB for your application
  - Check the status of a PDB with kubectl
  - Simulate disruptions with kubectl drain or delete
  - Update or delete a PDB with kubectl
- Best practices for using PDBs
  - Understand your application's availability requirements
  - Use selectors wisely to target the relevant pods
  - Opt for percentage-based disruption budgets when possible
  - Use PDBs with higher-level concepts such as deployments or statefulsets
  - Account for involuntary disruptions and plan for recovery
- Examples of PDBs for different types of applications
  - Stateless frontends
  - Single-instance stateful applications
  - Multiple-instance stateful applications such as Consul, ZooKeeper, or etcd
  - Restartable batch jobs

## Article ##

### What is a pod disruption budget (PDB)? ###

A pod disruption budget (PDB) is an API object that specifies the minimum number or percentage of pods that must be available for a set of pods during a disruption. A disruption is any event that causes one or more pods to be unavailable, either temporarily or permanently. Disruptions can be either voluntary or involuntary.

Voluntary disruptions are those that are initiated by the cluster administrator or the user, such as:

- Draining a node for maintenance or upgrade using `kubectl drain`
- Scaling down a node pool or a cluster using `kubectl scale`
- Deleting pods using `kubectl delete`

Involuntary disruptions are those that are caused by factors outside the control of the cluster administrator or the user, such as:

- Node failures due to hardware issues or power outages
- Node termination due to cloud provider actions or policies
- Pod eviction due to resource pressure or node taints

A PDB helps you protect your application from disruptions by ensuring that there are always enough pods available to serve requests and perform tasks. A PDB also helps Kubernetes respect your application's availability needs when performing voluntary disruptions. For example, if you have a PDB that specifies that at least three pods must be available for your application, Kubernetes will not evict more than one pod at a time when draining a node.

A PDB works with Kubernetes controllers that manage pods, such as deployments, replica sets, stateful sets, daemon sets, or jobs. A PDB applies to a set of pods that match a label selector, which is specified in the `.spec.selector` field of the PDB object. The label selector can match any pods in the cluster, regardless of their controller.

A PDB has two fields that define the minimum availability level for the matching pods: `.spec.minAvailable` and `.spec.maxUnavailable`. You can use either one of them, but not both. These fields can be either an absolute number or a percentage.

- `.spec.minAvailable` specifies the minimum number or percentage of pods that must be available after a disruption. For example, if you have 10 pods and set `.spec.minAvailable` to 3, Kubernetes will ensure that at least three pods are available after any disruption. If you set `.spec.minAvailable` to 50%, Kubernetes will ensure that at least five pods are available after any disruption.
- `.spec.maxUnavailable` specifies the maximum number or percentage of pods that can be unavailable after a disruption. For example, if you have 10 pods and set `.spec.maxUnavailable` to 2, Kubernetes will ensure that no more than two pods are unavailable after any disruption. If you set `.spec.maxUnavailable` to 30%, Kubernetes will ensure that no more than three pods are unavailable after any disruption.

### How to create a PDB for your application  ###

To create a PDB for your application, you need to follow these steps:

1. Determine the minimum number of instances or availability percentage for your application. This depends on your application's design, architecture, and availability requirements. For example, if your application is a stateless frontend that can handle requests with any number of pods, you might want to set a low minimum availability level, such as 10% or 1 pod. If your application is a stateful service that needs a quorum to operate, you might want to set a high minimum availability level, such as 50% or the quorum size.
2. Create a YAML file with the PDB definition. The YAML file should have the following structure:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: <name of the PDB>
spec:
  minAvailable: <minimum number or percentage of pods>
  # or
  maxUnavailable: <maximum number or percentage of pods>
  selector:
    matchLabels:
      <label selector for the pods>
```

Replace the placeholders with the appropriate values for your application. For example, if you want to create a PDB for an application with the label `app: my-app` that requires at least three pods to be available at any time, you can use the following YAML file:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: my-app
```

In Our Case we need to add this in our manifest file

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: #{namespace}-pdb
  namespace: #{namespace}
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: "#{namespace}#"
      version: "#{version}#"   
```

3. Apply the YAML file with `kubectl`. Use the `kubectl apply` command to create the PDB object in your cluster:

```bash
kubectl apply -f <name of the YAML file>
```

Replace the placeholder with the name of your YAML file. For example:

```bash
kubectl apply -f my-app-pdb.yaml
```

You should see a message confirming that the PDB was created:

```bash
poddisruptionbudget.policy/my-app-pdb created
```

### How to use a PDB for your application ###

Once you have created a PDB for your application, you can use it to monitor and manage the availability of your pods during disruptions. Here are some common tasks that you can perform with a PDB:

- Check the status of a PDB with `kubectl`. Use the `kubectl get` command to view the current status of your PDB:

```bash
kubectl get pdb <name of the PDB>
```

Replace the placeholder with the name of your PDB. For example:

```bash
kubectl get pdb my-app-pdb
```

You should see an output similar to this:

```bash
NAME         MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
my-app-pdb   3               N/A               2                     10m
```

The output shows the following information:

- The name of the PDB (`my-app-pdb`)
- The minimum number or percentage of pods that must be available (`3`)
- The maximum number or percentage of pods that can be unavailable (`N/A`)
- The number of additional pods that can be disrupted at the moment (`2`)
- The age of the PDB (`10m`)

You can also use the `kubectl describe` command to get more details about your PDB, such as the label selector, the current and desired number of pods, and any events related to the PDB.

- Simulate disruptions with `kubectl drain` or `delete`. You can test how your PDB works by simulating voluntary disruptions using `kubectl drain` or `delete`. For example, if you want to drain a node that has some pods matching your PDB, you can use the following command:

```bash
kubectl drain <name of the node> --ignore-daemonsets --delete-emptydir-data
```

Replace the placeholder with the name of the node. For example:

```bash
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
```

This command will try to evict all pods from the node, except for daemon sets and mirror pods. However, it will respect your PDB and ensure that the minimum availability level is maintained. You should see an output similar to this:

```bash
node/node-1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-ds-amd64-4tjz6, kube-system/weave-net-z9j47
evicting pod default/my-app-5c8dccc9d4-qw7t8
evict

(1) Specifying a Disruption Budget for your Application | Kubernetes. https://kubernetes.io/docs/tasks/run-application/configure-pdb/.
(2) kubernetes_pod_disruption_budget - Terraform Registry. https://registry.terraform.io/providers/dvulpe/kubernetes/latest/docs/resources/pod_disruption_budget.
(3) Pod Disruption Budgets: Why and How to Use PDBs. https://cast.ai/blog/pod-disruption-budgets-in-your-deployment/.
(4) Operator best practices - Basic scheduler features in Azure Kubernetes .... https://learn.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler.
(5) PodDisruptionBudget in Action with Examples | GoLinuxCloud. https://www.golinuxcloud.com/poddisruptionbudget/.
(6) Kubernetes - ReplicaSet vs PodDisruptionBudget - Stack Overflow. https://stackoverflow.com/questions/66227319/kubernetes-replicaset-vs-poddisruptionbudget.
(7) Best practices for upgrading clusters | Google Kubernetes Engine (GKE .... https://cloud.google.com/kubernetes-engine/docs/best-practices/upgrading-clusters.
(8) Kubernetes Best Practices :: DigitalOcean Documentation. https://docs.digitalocean.com/products/kubernetes/concepts/best-practices/.
(9) undefined. https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md.
(10) undefined. http://kubernetes.io/docs/user-guide/annotations.
(11) undefined. https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md.
(12) undefined. http://kubernetes.io/docs/user-guide/labels.
(13) undefined. http://kubernetes.io/docs/user-guide/identifiers.

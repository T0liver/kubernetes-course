# Chapter 11: Cheat Sheet & Glossary

[← Chapter 10](10-ecosystem-and-whats-next.md) | [Back to README](README.md)

Your quick-reference chapter. Bookmark this one — you'll come back to it far more than any other.

## `kubectl` command reference, by task

### Inspecting

```bash
kubectl get pods                         # list Pods in the current namespace
kubectl get pods -A                      # list Pods in ALL namespaces
kubectl get pods -o wide                 # include Node, IP, etc.
kubectl get all                          # Pods, Deployments, Services, etc. in one view
kubectl describe pod <name>              # full detail + recent Events (first stop when troubleshooting)
kubectl get events --sort-by=.lastTimestamp   # cluster-wide recent events, chronological
kubectl explain <resource>               # built-in docs for any object's fields (e.g. kubectl explain deployment.spec)
```

### Deploying / applying

```bash
kubectl apply -f file.yaml               # create or update from a manifest (idempotent — safe to re-run)
kubectl apply -f ./folder/               # apply every manifest in a folder
kubectl create deployment NAME --image=IMAGE     # quick, imperative Deployment (fine for learning/testing)
kubectl expose deployment NAME --port=80 --type=ClusterIP   # quick Service in front of a Deployment
kubectl delete -f file.yaml              # delete everything defined in a manifest
```

### Debugging

```bash
kubectl logs <pod>                       # container stdout/stderr
kubectl logs -f <pod>                    # follow/tail logs live
kubectl logs --previous <pod>            # logs from the pod's last crashed instance
kubectl exec -it <pod> -- sh             # shell into a running container (if it has one)
kubectl debug -it <pod> --image=busybox --target=<container>   # attach a debug container without a shell in the image
kubectl port-forward <pod> 8080:80       # tunnel a local port to a Pod, bypassing Services entirely
```

### Scaling & updating

```bash
kubectl scale deployment NAME --replicas=N        # change replica count directly
kubectl set image deployment/NAME CONTAINER=IMAGE:TAG   # trigger a rolling update
kubectl rollout status deployment/NAME            # watch a rollout's progress live
kubectl rollout history deployment/NAME           # list revisions
kubectl rollout undo deployment/NAME              # roll back to the previous revision
kubectl rollout undo deployment/NAME --to-revision=N   # roll back to a specific revision
```

### Namespaces & context

```bash
kubectl get namespaces
kubectl config get-contexts              # list clusters/contexts kubectl knows about
kubectl config use-context <name>        # switch which cluster kubectl talks to
kubectl config set-context --current --namespace=<ns>   # default to a namespace, instead of typing -n every time
```

### Config & secrets

```bash
kubectl create configmap NAME --from-literal=KEY=VALUE
kubectl create secret generic NAME --from-literal=KEY=VALUE
kubectl get secret NAME -o jsonpath='{.data.KEY}' | base64 -d   # decode a Secret's value
```

### Cleanup

```bash
kubectl delete pod <name>
kubectl delete deployment <name>
kubectl delete namespace <name>          # deletes everything inside it too — use carefully
```

## Glossary

Every major term introduced in this course, one line each, roughly in the order they first appeared.

- **Desired state** — what you've declared you want the cluster to look like; Kubernetes continuously works to make actual state match it. ([Ch. 1](01-why-kubernetes.md))
- **Declarative** — describing the *end result* you want, rather than the steps to get there (contrast with *imperative*, like a plain `docker run`). ([Ch. 1](01-why-kubernetes.md))
- **Self-healing** — Kubernetes automatically replacing failed Pods/containers to keep actual state matching desired state. ([Ch. 1](01-why-kubernetes.md))
- **Cluster** — the whole set of machines (Nodes) running Kubernetes together. ([Ch. 1](01-why-kubernetes.md), [Ch. 2](02-architecture.md))
- **Node** — one machine (physical or virtual) in the cluster; either control-plane or worker. ([Ch. 2](02-architecture.md))
- **Control plane** — the components that hold and enforce cluster desired state (API server, etcd, scheduler, controller manager). ([Ch. 2](02-architecture.md))
- **`kube-apiserver`** — the front door; every interaction with the cluster goes through it. ([Ch. 2](02-architecture.md))
- **`etcd`** — the distributed key-value store holding all cluster state. ([Ch. 2](02-architecture.md))
- **`kube-scheduler`** — assigns newly created Pods to a suitable Node. ([Ch. 2](02-architecture.md))
- **`kube-controller-manager`** — runs the control loops (controllers) that reconcile actual state to desired state. ([Ch. 2](02-architecture.md))
- **`cloud-controller-manager`** — integrates the cluster with cloud-provider-specific infrastructure (e.g., provisioning a `LoadBalancer`). ([Ch. 2](02-architecture.md))
- **`kubelet`** — the per-Node agent that keeps that Node's Pods running as instructed. ([Ch. 2](02-architecture.md))
- **`kube-proxy`** — maintains per-Node network rules that make Services work. ([Ch. 2](02-architecture.md), [Ch. 4](04-networking-deep-dive.md))
- **Container runtime** — the software that actually runs containers (e.g., containerd, CRI-O), speaking the Container Runtime Interface (CRI). ([Ch. 1](01-why-kubernetes.md), [Ch. 2](02-architecture.md))
- **Pod** — the smallest deployable unit: one or more containers sharing network and (optionally) storage. ([Ch. 1](01-why-kubernetes.md), [Ch. 3](03-core-workloads.md))
- **ReplicaSet** — ensures a set number of identical Pod replicas are running. ([Ch. 3](03-core-workloads.md))
- **Deployment** — manages ReplicaSets to provide declarative updates, rollbacks, and scaling for stateless workloads. ([Ch. 3](03-core-workloads.md))
- **StatefulSet** — like a Deployment, but for workloads needing stable per-Pod identity and storage. ([Ch. 3](03-core-workloads.md))
- **DaemonSet** — ensures one Pod copy runs on every (matching) Node. ([Ch. 3](03-core-workloads.md))
- **Job** — runs a Pod to completion for a one-off task. ([Ch. 3](03-core-workloads.md))
- **CronJob** — a Job on a recurring schedule. ([Ch. 3](03-core-workloads.md))
- **CNI (Container Network Interface)** — the pluggable interface Kubernetes uses to delegate actual pod-to-pod networking to a provider (e.g., Cilium, Calico, Flannel). ([Ch. 4](04-networking-deep-dive.md))
- **Service** — a stable virtual IP/DNS name in front of a changing set of Pods. ([Ch. 4](04-networking-deep-dive.md))
- **ClusterIP** — a Service type reachable only inside the cluster (the default). ([Ch. 4](04-networking-deep-dive.md))
- **NodePort** — a Service type exposing a static port on every Node's IP. ([Ch. 4](04-networking-deep-dive.md))
- **LoadBalancer** — a Service type that provisions an external cloud load balancer. ([Ch. 4](04-networking-deep-dive.md))
- **Ingress** — the legacy API object/controller pattern for HTTP(S) routing into a cluster; superseded by the Gateway API for new clusters. ([Ch. 4](04-networking-deep-dive.md))
- **Gateway API** — the modern, role-oriented API for HTTP(S)/traffic routing into a cluster, recommended over Ingress for new setups since `ingress-nginx`'s March 2026 retirement. ([Ch. 4](04-networking-deep-dive.md))
- **CoreDNS** — the DNS server providing cluster-internal service discovery. ([Ch. 4](04-networking-deep-dive.md))
- **ConfigMap** — non-sensitive key-value configuration data, injectable into Pods. ([Ch. 5](05-config-and-storage.md))
- **Secret** — like a ConfigMap, but for sensitive data — base64-encoded, not encrypted, by default. ([Ch. 5](05-config-and-storage.md), [Ch. 7](07-kubernetes-security-basics.md))
- **Volume** — storage attached to a Pod, with a lifecycle tied to the Pod (or longer, for persistent types). ([Ch. 5](05-config-and-storage.md))
- **`emptyDir`** — a temporary Volume type, deleted with its Pod; useful for scratch space shared between containers in a Pod. ([Ch. 5](05-config-and-storage.md))
- **PersistentVolume (PV)** — an actual piece of provisioned storage in the cluster. ([Ch. 5](05-config-and-storage.md))
- **PersistentVolumeClaim (PVC)** — a Pod's request for storage, bound to a matching PV. ([Ch. 5](05-config-and-storage.md))
- **StorageClass** — defines how to dynamically provision a PV on demand. ([Ch. 5](05-config-and-storage.md))
- **Requests** — the resources (CPU/memory) a container is guaranteed and the scheduler plans around. ([Ch. 6](06-scheduling-scaling-health.md))
- **Limits** — the hard ceiling on a container's resource use. ([Ch. 6](06-scheduling-scaling-health.md))
- **Liveness probe** — checks whether a container is still working; failure triggers a restart. ([Ch. 6](06-scheduling-scaling-health.md))
- **Readiness probe** — checks whether a container should currently receive traffic; failure removes it from Service load-balancing without a restart. ([Ch. 6](06-scheduling-scaling-health.md))
- **Startup probe** — delays liveness/readiness checks until a slow-starting container has finished starting. ([Ch. 6](06-scheduling-scaling-health.md))
- **HorizontalPodAutoscaler (HPA)** — automatically adjusts replica count based on observed metrics. ([Ch. 6](06-scheduling-scaling-health.md))
- **Taint** — a mark on a Node that repels Pods unless they tolerate it. ([Ch. 6](06-scheduling-scaling-health.md))
- **Toleration** — a mark on a Pod allowing it onto a Node with a matching taint. ([Ch. 6](06-scheduling-scaling-health.md))
- **Affinity / anti-affinity** — scheduling preferences/requirements based on labels, for co-locating or spreading out Pods. ([Ch. 6](06-scheduling-scaling-health.md))
- **RBAC (Role-Based Access Control)** — governs who/what can do what to which resources. ([Ch. 7](07-kubernetes-security-basics.md))
- **Role / ClusterRole** — a set of permissions, namespace-scoped or cluster-wide. ([Ch. 7](07-kubernetes-security-basics.md))
- **RoleBinding / ClusterRoleBinding** — grants a Role/ClusterRole to a user, group, or service account. ([Ch. 7](07-kubernetes-security-basics.md))
- **Least privilege** — the principle of granting only the access actually needed, no more. ([Ch. 7](07-kubernetes-security-basics.md))
- **NetworkPolicy** — a firewall-like rule set controlling pod-to-pod traffic; only enforced if the CNI plugin supports it. ([Ch. 7](07-kubernetes-security-basics.md))
- **Default-deny** — a NetworkPolicy pattern blocking all traffic by default, then explicitly allowing only what's needed. ([Ch. 7](07-kubernetes-security-basics.md))
- **Pod Security Admission (PSA) / Pod Security Standards** — the current built-in mechanism and profiles (Privileged, Baseline, Restricted) for pod-level security policy, replacing the removed PodSecurityPolicy. ([Ch. 7](07-kubernetes-security-basics.md))
- **PodSecurityPolicy (PSP)** — the deprecated (v1.21) and removed (v1.25) predecessor to Pod Security Admission. ([Ch. 7](07-kubernetes-security-basics.md))
- **`kind` / Minikube** — tools for running a local Kubernetes cluster on your own machine. ([Ch. 8](08-hands-on-lab.md))
- **Rolling update** — replacing Pods with a new version gradually, via a new ReplicaSet scaling up as the old one scales down. ([Ch. 9](09-day-to-day-operations.md))
- **Rollback** — reverting a Deployment to a previous ReplicaSet/revision. ([Ch. 9](09-day-to-day-operations.md))
- **`kubectl debug`** — attaches a temporary debugging container to a Pod without altering its image. ([Ch. 9](09-day-to-day-operations.md))
- **Helm** — Kubernetes's standard package manager; installs templated bundles of manifests called charts. ([Ch. 9](09-day-to-day-operations.md), [Ch. 10](10-ecosystem-and-whats-next.md))
- **Chart** — a packaged, templated Helm bundle of Kubernetes manifests. ([Ch. 9](09-day-to-day-operations.md))
- **GitOps** — an operating model where Git is the source of truth and an in-cluster controller reconciles the cluster to match it. ([Ch. 10](10-ecosystem-and-whats-next.md))
- **ArgoCD / Flux** — the two most widely used GitOps controllers. ([Ch. 10](10-ecosystem-and-whats-next.md))
- **Managed Kubernetes (EKS / GKE / AKS)** — cloud-provider-run Kubernetes where the provider operates the control plane for you. ([Ch. 10](10-ecosystem-and-whats-next.md))
- **KCNA / KCSA / CKAD / CKA / CKS** — the current Linux Foundation/CNCF Kubernetes certifications, from entry-level (KCNA, KCSA) through hands-on administration (CKA) and application development (CKAD) to advanced security (CKS). ([Ch. 10](10-ecosystem-and-whats-next.md))

---
[← Chapter 10](10-ecosystem-and-whats-next.md) | [Back to README](README.md)

# Chapter 8: Hands-On Lab

[← Chapter 7](07-kubernetes-security-basics.md) | [Back to README](README.md) | Next: [Chapter 9 — Day-to-Day Operations →](09-day-to-day-operations.md)

This is the chapter where everything from Chapters 1–7 becomes real. It's split in two: **Part A** gets you a cluster in your browser, right now, with nothing installed. **Part B** is optional — installing a local cluster on your own machine, useful once you know you'll be doing this regularly. **Either way, skip to "The Walkthrough" once you have a cluster** — the exercise is identical regardless of which platform you used to get there.

## Part A — Try it without installing anything (do this first)

All of these were checked for being live and free at the time of writing. Free browser-based Kubernetes playgrounds have a history of shutting down (Docker's own "Play with Kubernetes" ecosystem traces back to this space, and Katacoda — once one of the most popular options — was fully shut down back in 2023), so if a link below is stale by the time you read this, search for its replacement rather than assuming Kubernetes learning environments in general have gone away.

| Platform | What it's good for | Limitations | Link |
|---|---|---|---|
| **Killercoda** | Full interactive terminal + pre-built Kubernetes scenarios, no signup required for many scenarios; also referenced directly from Kubernetes's own documentation as a learning environment | Session time limits (commonly around an hour); environment resets after the session ends | [killercoda.com](https://killercoda.com/) |
| **Google Cloud Shell** | A real, persistent (per your Google account) cloud shell with `kubectl` pre-installed; pair it with a free/trial GKE Autopilot cluster or a local `kind`/Minikube cluster running inside the Cloud Shell VM | Requires a Google account; Cloud Shell sessions themselves are time-boxed per session (though your files persist); a real GKE cluster costs money beyond any free trial credit — for this lab, prefer running `kind` *inside* Cloud Shell to stay free | [cloud.google.com/shell](https://cloud.google.com/shell) |
| **KodeKloud free playgrounds** | Pre-configured Kubernetes playgrounds (including specific version choices) aimed squarely at learners, and part of a platform that also has structured Kubernetes courses if you want to go further | Free playgrounds typically require a free KodeKloud account and have session/resource limits | [kodekloud.com](https://kodekloud.com/) |
| **Play with Kubernetes** (`labs.play-with-k8s.com`) | Historically a very popular, zero-signup, instant multi-node cluster in the browser | It's a community-maintained project without the backing or update cadence of the options above — it was reachable at the time of writing, but treat it as a "try it, and fall back to Killercoda or KodeKloud if it's flaky" option rather than your first choice | [labs.play-with-k8s.com](https://labs.play-with-k8s.com/) |

**Recommendation:** start with **Killercoda** or **KodeKloud** — both are actively maintained, purpose-built for exactly this kind of exercise, and referenced by the wider Kubernetes learning community. Pick whichever's signup flow you prefer, launch a plain Kubernetes playground/scenario (not a specific certification scenario), and you'll land in a terminal with `kubectl` already configured against a real (if temporary) cluster.

## Part B — Local install (optional, useful to know)

If you know you'll be doing this often, a local cluster you fully control is worth setting up eventually — but it's genuinely optional for this course; the walkthrough below works identically on a browser playground.

1. **Install a local cluster tool** — either works well for learning:
   - **[`kind`](https://kind.sigs.k8s.io/)** ("Kubernetes IN Docker") — runs a real Kubernetes cluster as Docker containers. Since you already have Docker knowledge, this tends to feel the most natural.
   - **[Minikube](https://minikube.sigs.k8s.io/)** — runs a full Kubernetes cluster in a VM or container on your machine; slightly more resource-heavy than `kind`, with a few more built-in conveniences (like `minikube dashboard`).
2. **Install `kubectl`** — the command-line tool you'll use for everything in this course. [Official install instructions](https://kubernetes.io/docs/tasks/tools/#kubectl) cover macOS, Windows, and Linux.
3. **A container runtime to build/run images with.** Both `kind` and Minikube need a container engine underneath:
   - **Docker Desktop** is the default choice most people already have, but its licensing changes in recent years pushed a number of companies (and individuals) toward alternatives — worth being aware of, especially in a work setting where licensing terms matter.
   - **[Rancher Desktop](https://rancherdesktop.io/)** is a free, open-source alternative that provides a Docker-compatible engine and can run Kubernetes itself; it's a common substitute at companies that moved off Docker Desktop for licensing reasons. Verify current licensing terms for whichever tool you choose before using it at work — this is exactly the kind of detail that can change, and it's a company policy question as much as a technical one.

Either `kind` or Minikube plus `kubectl` gets you the same starting point as Part A: a working cluster and a terminal.

## The Walkthrough

Everything from here works the same whether you're in a browser playground or a local cluster. This deploys a small multi-container app — a web frontend plus a database — end to end.

### Step 1 — Confirm your cluster is up

```bash
kubectl cluster-info
kubectl get nodes
```

You should see at least one Node in `Ready` status.

### Step 2 — Write your first Deployment + Service YAML

Create a file called `webapp.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: nginxdemos/hello
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 100m
              memory: 128Mi
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  type: ClusterIP
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 80
```

This uses `nginxdemos/hello`, a small public image that returns a page showing which specific Pod served the request — useful for *seeing* load-balancing happen in Step 6, rather than just being told it does.

### Step 3 — Add a database

Create a second file, `database.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  labels:
    app: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres:16-alpine
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          ports:
            - containerPort: 5432
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 250m
              memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  type: ClusterIP
  selector:
    app: db
  ports:
    - port: 5432
      targetPort: 5432
```

This pulls its password from a Secret rather than hardcoding it — tying directly back to [Chapter 5](05-config-and-storage.md). Create that Secret first:

```bash
kubectl create secret generic db-secret --from-literal=password=changeme123
```

### Step 4 — Deploy everything

```bash
kubectl apply -f webapp.yaml
kubectl apply -f database.yaml
```

### Step 5 — Check status

```bash
kubectl get deployments,pods,svc
kubectl describe pod -l app=webapp
```

Wait until all Pods show `Running` and `1/1` (or `2/2`) under `READY`. If a Pod is stuck, `kubectl describe pod <name>` is your first troubleshooting stop — it shows recent events, which is usually where the actual error message lives (this pattern gets covered more in [Chapter 9](09-day-to-day-operations.md)).

### Step 6 — View logs and see load-balancing in action

```bash
kubectl logs -l app=webapp --tail=20
kubectl run tmp-shell --rm -it --image=busybox -- sh
# inside the temporary shell:
wget -qO- webapp
wget -qO- webapp
wget -qO- webapp
exit
```

Run the `wget` line a few times — with `nginxdemos/hello` you should see the responding Pod's hostname/IP change between requests, which is the Service's load-balancing across your 2 `webapp` replicas, made visible.

### Step 7 — Scale up and down

```bash
kubectl scale deployment webapp --replicas=5
kubectl get pods -l app=webapp -w
```

(`Ctrl+C` to stop watching once you see 5 Pods running.) Then scale back down:

```bash
kubectl scale deployment webapp --replicas=2
```

### Step 8 — Tear it all down

```bash
kubectl delete -f webapp.yaml
kubectl delete -f database.yaml
kubectl delete secret db-secret
```

Confirm everything's gone:

```bash
kubectl get all
```

## What you just did, mapped back to the course

- **Step 2/3** — Deployments and Services, from [Chapters 3](03-core-workloads.md) and [4](04-networking-deep-dive.md).
- **Step 3** — Secrets injection, from [Chapter 5](05-config-and-storage.md).
- **Step 2** — resource requests/limits and a readiness probe, from [Chapter 6](06-scheduling-scaling-health.md).
- **Step 6** — cluster DNS and Service load-balancing, from [Chapter 4](04-networking-deep-dive.md).
- **Step 7** — the same mechanism the HPA drives automatically, from [Chapter 6](06-scheduling-scaling-health.md).

## 🎥 Video

*A single short (≤20 min), currently-live video walking through this exact "Deployment + Service + database, deploy/scale/logs/teardown" flow end to end wasn't something this course could pin down with a verified, stable link at the time of writing. The [official Kubernetes "Deploying an application" tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/) is a reliable, always-current, interactive alternative that covers the same ground step by step.*

## 💡 Fun facts

- **`nginxdemos/hello` returning the responding Pod's own hostname is a small but genuinely useful debugging trick** — the same pattern (having an app expose its own Pod identity) is commonly used in real clusters to verify that load-balancing, rolling updates, and canary deployments are actually distributing traffic the way you expect.
- **Katacoda, once one of the most widely used free interactive learning platforms for Kubernetes (and referenced directly from Kubernetes's own official documentation for years), was fully shut down in 2023** after its parent company (O'Reilly) discontinued it — a useful reminder that even official-feeling free tools in this space can and do disappear, which is exactly why this chapter double-checked its links rather than assuming yesterday's popular answer still works today.

---
[← Chapter 7](07-kubernetes-security-basics.md) | [Back to README](README.md) | Next: [Chapter 9 — Day-to-Day Operations →](09-day-to-day-operations.md)

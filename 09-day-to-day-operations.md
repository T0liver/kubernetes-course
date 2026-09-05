# Chapter 9: Day-to-Day Operations

[← Chapter 8](08-hands-on-lab.md) | [Back to README](README.md) | Next: [Chapter 10 — Ecosystem & What's Next →](10-ecosystem-and-whats-next.md)

This chapter is about the things you'll actually do most days once a cluster is up and running: shipping new versions, reading what went wrong, and the first tool most teams add on top of raw `kubectl`.

## Rolling updates and rollbacks

Recall from [Chapter 3](03-core-workloads.md) that a Deployment manages ReplicaSets, and updating a Deployment's image creates a *new* ReplicaSet while scaling the old one down. That's a **rolling update**: old and new versions briefly coexist, and traffic gradually shifts, rather than an all-at-once cutover.

```bash
kubectl set image deployment/webapp webapp=nginxdemos/hello:plain-text
kubectl rollout status deployment/webapp
```

`kubectl rollout status` blocks and shows live progress — genuinely useful to run right after a deploy, rather than immediately jumping to `kubectl get pods` and eyeballing it yourself.

If a rollout goes wrong:

```bash
kubectl rollout history deployment/webapp
kubectl rollout undo deployment/webapp
```

`rollout undo` reverts to the previous ReplicaSet — Kubernetes kept it around specifically for this. You can also target a specific earlier revision with `kubectl rollout undo deployment/webapp --to-revision=<N>`, using the revision numbers shown by `rollout history`.

## Reading logs and describing objects

Two commands do most of the diagnostic work in Kubernetes:

- **`kubectl logs <pod>`** — stdout/stderr from a container, exactly like `docker logs`. Add `-f` to follow/tail live, `--previous` to see the logs from a container's *last* crashed instance (essential when a container is stuck in a restart loop and you need to see why the *previous* attempt died, not the brand-new one).
- **`kubectl describe <object> <name>`** — not logs, but a full dump of an object's spec, status, and — critically — its recent **Events**. Events are where scheduling failures, image pull errors, and probe failures actually get reported. If a Pod won't start, `kubectl describe pod` is where you look before anything else.

## A basic troubleshooting workflow

When something isn't working, this order covers the large majority of cases:

1. **`kubectl get pods`** — is the Pod even there? What phase is it in (`Pending`, `CrashLoopBackOff`, `ImagePullBackOff`, `Running`)?
2. **`kubectl describe pod <name>`** — check the Events section at the bottom. `Pending` usually means a scheduling problem (resource requests too high for available Nodes, an unmet taint/toleration); `ImagePullBackOff` means the image name/tag/registry credentials are wrong; `CrashLoopBackOff` means the container starts and then exits.
3. **`kubectl logs <pod>`** (and `--previous` if it's crash-looping) — what did the application itself say before it died?
4. **`kubectl get events --sort-by=.lastTimestamp`** — a cluster-wide, chronological view of recent events, useful when you're not even sure which object is the problem yet.

## `kubectl debug`: getting a shell into a broken or minimal container

Sometimes you need to poke around *inside* a running (or crashed) Pod, but the container image doesn't have a shell, `curl`, or any other debugging tool in it (which is increasingly common and deliberate — see the "minimal base images" point in [Chapter 7](07-kubernetes-security-basics.md)). `kubectl debug` solves this without permanently modifying the Pod's image:

```bash
# Attach a temporary debugging container to a running Pod, sharing its process namespace:
kubectl debug -it <pod-name> --image=busybox --target=<container-name>

# Or create a full copy of a Pod with a debug container added, leaving the original untouched:
kubectl debug <pod-name> -it --image=busybox --copy-to=debug-copy --container=debug
```

This is the modern, built-in replacement for the older habit of `kubectl exec`-ing into a container and hoping it happens to have the tool you need already installed — which increasingly it won't, on purpose.

## A first look at Helm

As your manifests grow — a Deployment, a Service, a ConfigMap, a Secret, maybe more, per application — hand-writing and re-applying each YAML file for every environment (dev/staging/prod) gets repetitive and error-prone. **Helm** is Kubernetes's most widely used package manager, solving exactly that:

- A **chart** is a packaged, templated bundle of Kubernetes manifests (think: an installable "package," similar in spirit to an `apt` or `npm` package, but for a set of Kubernetes objects).
- Charts are **templated** — the same chart can be installed multiple times with different `values.yaml` overrides (different replica counts, image tags, or resource limits per environment).
- Installing a chart is one command instead of `kubectl apply -f` across a folder of files:

```bash
helm install my-release some-chart --set replicaCount=3
```

You don't need to author your own charts to get value from Helm on day one — a huge number of common tools (databases, monitoring stacks, ingress controllers, and more) already ship official or community-maintained Helm charts, so `helm install` is very often the fastest way to stand up infrastructure you didn't write yourself. [Chapter 10](10-ecosystem-and-whats-next.md) picks this back up alongside the rest of the ecosystem.

## 🧪 Hands-on checkpoint

If you have a cluster available (and still have the `webapp` Deployment from [Chapter 8](08-hands-on-lab.md), or recreate it quickly):

```bash
kubectl set image deployment/webapp webapp=nginxdemos/hello:plain-text
kubectl rollout status deployment/webapp
kubectl rollout undo deployment/webapp
kubectl rollout status deployment/webapp
```

You just shipped a change and rolled it back, live — the exact sequence you'll reach for the first time a real deploy goes wrong.

## 🎥 Video

**[What is Helm in Kubernetes? Helm and Helm Charts explained](https://www.youtube.com/watch?v=-ykwb1d0DXU)** — TechWorld with Nana
A focused explainer on exactly the Helm concepts introduced above — charts, templating, and why teams adopt it — good preparation before you first `helm install` something for real.

## 💡 Fun facts

- **A Helm chart's name is a nautical pun that fits the whole Kubernetes naming theme** — a ship's helm is literally the wheel/mechanism used to steer it, extending the same "Kubernetes = helmsman" etymology from [Chapter 1](01-why-kubernetes.md).
- **`kubectl rollout undo` doesn't "undo" by magic** — it works because the old ReplicaSet from before your update was never deleted, just scaled to zero. Rolling back is really just scaling the old ReplicaSet back up and the new one back down, using the exact same mechanism as a forward rollout.

---
[← Chapter 8](08-hands-on-lab.md) | [Back to README](README.md) | Next: [Chapter 10 — Ecosystem & What's Next →](10-ecosystem-and-whats-next.md)

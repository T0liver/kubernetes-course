# Chapter 10: Ecosystem & What's Next

[← Chapter 9](09-day-to-day-operations.md) | [Back to README](README.md) | Next: [Chapter 11 — Cheat Sheet & Glossary →](11-cheatsheet-and-glossary.md)

This closing chapter is a map of what's around Kubernetes — so that when you hear these names at work, you know roughly where they fit — plus some thoughts on where to go next if you want to keep building on this course.

## Helm, recapped

From [Chapter 9](09-day-to-day-operations.md): Helm is Kubernetes's standard package manager — templated bundles of manifests (**charts**) that can be installed, upgraded, and rolled back as a single unit, with per-environment overrides via values files. If you remember one thing: **`helm install` is very often faster than hand-writing YAML for anything you didn't build yourself.**

## GitOps: ArgoCD and Flux, conceptually

**GitOps** is an operating model, not a specific tool: your cluster's desired state (Deployments, Services, config — everything) lives as YAML files in a Git repository, treated as the single source of truth. A GitOps controller running *inside* the cluster continuously watches that Git repo and reconciles the live cluster to match it — the same "desired state, continuously reconciled" idea from [Chapter 1](01-why-kubernetes.md), just applied one level up, to *entire cluster configurations* instead of individual Pods.

The two most widely used GitOps controllers are **ArgoCD** and **Flux**. In both cases, the practical shift from a traditional CI/CD pipeline is:

- **Traditional CI/CD**: a pipeline (outside the cluster) runs `kubectl apply` or `helm upgrade` *against* the cluster, triggered by a code change.
- **GitOps**: a controller *inside* the cluster pulls from Git and applies changes itself; a `git push` to the right repo is what triggers a deployment, and the controller can also detect and automatically correct manual/out-of-band changes ("drift") that don't match Git.

You don't need hands-on GitOps experience for this course — just recognize that if a team says "we do GitOps" or mentions ArgoCD/Flux, they mean this pattern.

## Managed Kubernetes: what "managed" actually removes from your plate

**EKS** (AWS), **GKE** (Google Cloud), and **AKS** (Azure) are the three major clouds' managed Kubernetes offerings. "Managed" specifically means:

- The cloud provider runs and operates the **control plane** for you (the `kube-apiserver`, `etcd`, scheduler, controller-manager from [Chapter 2](02-architecture.md)) — you don't patch it, back it up, or scale it yourself, and in most cases you don't even have direct machine access to it.
- You typically still manage (or at least configure) the **worker nodes** — though even that can be further abstracted away by "autopilot"/"serverless node" modes on some of these platforms, where the provider manages node provisioning and sizing too.
- You still fully own everything you deploy *onto* the cluster: your Deployments, Services, RBAC policies, NetworkPolicies, and so on.

In short: managed Kubernetes removes the operational burden of running the control plane, not the responsibility of using Kubernetes correctly and securely once you have one.

## Certification paths

If formal certification is something you or your employer wants, here's how the current Linux Foundation/CNCF options relate to each other — genuinely relevant for you given the security side of your role:

| Certification | Format | Level | Notes |
|---|---|---|---|
| **KCNA** (Kubernetes and Cloud Native Associate) | Multiple-choice | Entry-level | Broad foundational knowledge across Kubernetes and the cloud-native ecosystem — a reasonable first step if you want a structured on-ramp before this course's material fully sinks in |
| **KCSA** (Kubernetes and Cloud Native Security Associate) | Multiple-choice | Entry-level, security-focused | Validates baseline security configuration knowledge — directly aligned with [Chapter 7](07-kubernetes-security-basics.md), and a lighter entry point than CKS below |
| **CKAD** (Certified Kubernetes Application Developer) | Hands-on, performance-based | Intermediate | Focused on building and deploying applications *onto* Kubernetes, rather than administering the cluster itself |
| **CKA** (Certified Kubernetes Administrator) | Hands-on, performance-based | Intermediate | Focused on cluster administration — the closest match to this course's overall scope, if you want to go deeper on operations |
| **CKS** (Certified Kubernetes Security Specialist) | Hands-on, performance-based | Advanced (requires an active CKA first) | The deep, hands-on security certification — the natural longer-term target given your role, once CKA is in hand |

Given your specific situation (new job, security responsibilities, wants working knowledge first): **KCNA and/or KCSA are the most proportionate next step**, if you decide to pursue certification at all — both are multiple-choice, achievable without months of hands-on lab time, and KCSA in particular maps directly onto [Chapter 7](07-kubernetes-security-basics.md)'s content. CKA and then CKS would be the longer-term path if the role grows into deeper hands-on cluster administration and security work. (The Linux Foundation also bundles all five of these together under a "Kubestronaut" badge for people who complete the full set — not something to aim for immediately, but good to know it exists if you end up on the certification track long-term.)

## Where to go from here

You now have working knowledge across the full stack this course promised: what problem Kubernetes solves, how it's built, the core objects, networking (including the current Ingress → Gateway API shift), config/storage, scaling/health, security fundamentals, one real deployment from scratch, and the day-to-day operational habits. The single highest-value next step is simply **using it** — the [hands-on lab in Chapter 8](08-hands-on-lab.md) is designed to be repeated with your own, real application rather than the sample one.

## 💡 Fun facts

- **ArgoCD's name follows the same mythological-reference pattern as a lot of cloud-native tooling** — Argo was the ship in Greek mythology that Jason and the Argonauts sailed, continuing the nautical theme that started with Kubernetes's own name and logo back in [Chapter 1](01-why-kubernetes.md).
- **KCSA was one of the newer additions to the Linux Foundation's Kubernetes certification lineup** — introduced specifically to give people an associate-level, multiple-choice entry point into Kubernetes security topics, without requiring the full hands-on rigor (and CKA prerequisite) of CKS.

---
[← Chapter 9](09-day-to-day-operations.md) | [Back to README](README.md) | Next: [Chapter 11 — Cheat Sheet & Glossary →](11-cheatsheet-and-glossary.md)

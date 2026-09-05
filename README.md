# Kubernetes Basics — A Working-Knowledge Course

A self-paced, no-fluff course for getting from "I know Docker" to "I can work with Kubernetes day-to-day, including its security side." Built as a **GitHub book**: plain Markdown files you can read on GitHub, in an editor, or in any Markdown viewer.

## Who this is for

You already know Docker — images, containers, `docker run`, `docker-compose`. You don't know Kubernetes yet, and you're about to need it at work, including some security responsibility. This course is **not** a certification-prep course and it doesn't try to make you an expert. It aims for solid working knowledge in **3–5 hours** (reading + short videos + one hands-on lab).

## How this course is built

- Every chapter is a **self-contained Markdown file** — read them in order, or jump to the one you need.
- 🎥 **Videos** — 1–2 short (≤20 min), currently-live YouTube videos per chapter, chosen to add something the text doesn't.
- 💡 **Fun facts** — quick, verified trivia to help things stick.
- 🧪 **Hands-on checkpoints** — tiny, doable-in-minutes actions, even before the main lab.
- **Diagrams** — Mermaid diagrams in fenced code blocks. GitHub renders these natively; if your viewer doesn't, paste the block into the [Mermaid Live Editor](https://mermaid.live).
- ⚠️ **Currency flags** — this ecosystem moves fast. Anywhere something is likely to change soon, you'll see a short flag telling you to double-check.

## Before you start

- You know what a container, an image, and `docker-compose` are.
- You're comfortable in a terminal.
- You don't need anything installed yet — [Chapter 8](08-hands-on-lab.md) starts with a browser-only option.

## Table of contents

| # | Chapter | What you'll get |
|---|---------|------------------|
| 1 | [Why Kubernetes Exists](01-why-kubernetes.md) | The problem Kubernetes solves, mapped to what you already know from Docker |
| 2 | [Kubernetes Architecture](02-architecture.md) | Control plane and node components, and how a `kubectl apply` becomes a running Pod |
| 3 | [Core Workload Objects](03-core-workloads.md) | Pods, ReplicaSets, Deployments, StatefulSets, DaemonSets, Jobs/CronJobs — and when to use which |
| 4 | [Networking Deep Dive](04-networking-deep-dive.md) | Cluster networking, Services, Ingress vs. the Gateway API, and cluster DNS |
| 5 | [Configuration & Storage](05-config-and-storage.md) | ConfigMaps, Secrets (and their limits), Volumes, PVs/PVCs, StorageClasses |
| 6 | [Scheduling, Scaling & Health](06-scheduling-scaling-health.md) | Requests/limits, probes, the Horizontal Pod Autoscaler, taints/affinity basics |
| 7 | [Kubernetes Security Basics](07-kubernetes-security-basics.md) | The attack surface, RBAC, secrets management, Network Policies, Pod Security Admission, image supply chain |
| 8 | [Hands-On Lab](08-hands-on-lab.md) | Deploy, expose, scale, and tear down a real multi-container app — browser-first, local-install optional |
| 9 | [Day-to-Day Operations](09-day-to-day-operations.md) | Rolling updates/rollbacks, logs, troubleshooting, `kubectl debug`, first look at Helm |
| 10 | [Ecosystem & What's Next](10-ecosystem-and-whats-next.md) | Helm, GitOps, managed Kubernetes, and which certification to consider |
| 11 | [Cheat Sheet & Glossary](11-cheatsheet-and-glossary.md) | Command reference by task, plus every bolded term defined in one place |

## A note on currency

At the time of writing (checked against sources current as of **early September 2026**):

- The current stable Kubernetes minor version is **1.37** ("Garhwal," released August 26, 2026); Kubernetes supports the three most recent minor versions (currently 1.37, 1.36, 1.35).
- **`ingress-nginx` was retired in March 2026.** The **Gateway API** is now the CNCF-recommended default for new clusters; classic `Ingress` is still common in existing environments. [Chapter 4](04-networking-deep-dive.md) covers both and is explicit about which is which.
- **Pod Security Admission / Pod Security Standards** remain the current built-in mechanism for pod-level security policy (successor to the removed PodSecurityPolicy). No further replacement has landed as of this writing.

Check the [Kubernetes release notes](https://kubernetes.io/releases/) and the [Gateway API site](https://gateway-api.sigs.k8s.io/) if it's been a while since this was written.

## License / use

This is a personal learning resource. Feel free to fork it, edit it, and make it your own.

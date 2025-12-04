# Kubernetes Bank-Style Guardrails Lab

In my previous role at a bank, developers used a heavily restricted internal PaaS built on Kubernetes.  
Most security and operational rules were invisible to us — they were enforced silently by the platform team.

To truly understand what was hidden under the hood, I rebuilt the most important banking-grade guardrails on my own kubeadm cluster (v1.30).

## What this lab demonstrates
- Real production restrictions that exist in every serious financial company
- How they are implemented today using standard open-source tools (no custom code)
- What happens when you remove them (immediate OOM, scheduling failures, security exposure)

## Policies included (all tested and working)

| # | Policy                              | Type      | Purpose                                           | Banking reality check                          |
|---|-------------------------------------|-----------|---------------------------------------------------|------------------------------------------------|
| 1 | disallow-latest-tag                 | validate  | Block `:latest` images                            | Always forbidden                               |
| 2 | require-requests-limits             | validate  | Must define requests & limits                     | One pod could kill the node without this       |
| 3 | disallow-privileged                 | validate  | Block privileged containers                       | Security team red line                         |
| 4 | disallow-host-network-port          | validate  | Block hostNetwork & hostPort                      | Breaks network isolation                       |
| 5 | require-run-as-non-root             | validate  | Containers must not run as root                   | CIS benchmark + regulator requirement          |
| 6 | require-labels (app + env)          | validate  | Enforce mandatory labels                          | Needed for billing, observability, blast radius|
| 7 | restrict-image-registries           | validate  | Only allow internal registry                      | Never pull from Docker Hub in prod             |
| 8 | auto-add-default-limits             | **mutate**| Auto-inject reasonable defaults if missing        | What the bank PaaS did silently                |

All **validate** policies use `validationFailureAction: Enforce` → immediate rejection on `kubectl apply` (exactly like banking production).  
Policy #8 is a **mutate** policy → silently fixes missing limits.

## Why I built this
After leaving the bank environment, I immediately broke my own cluster multiple times by doing things the PaaS never allowed.  
This lab is the result: now I fully understand why each restriction exists.

Cluster: self-built with kubeadm on Ubuntu 22.04 → Kubernetes v1.30.3  
Tools: Kyverno v1.12.6 (the current industry standard for policy enforcement)

Feel free to clone and try breaking the rules — you’ll see the exact error messages we never saw in the bank :)

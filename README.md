# Kubernetes Bank-Style Guardrails Lab

In my previous job at a bank, we used a heavily locked-down internal PaaS on top of Kubernetes.  
Developers never saw raw nodes, etcd, or even most YAML fields — everything was enforced automatically.

I rebuilt the exact same restrictions that existed in every serious financial institution, so I now understand **why** each rule is non-negotiable.

Self-built kubeadm cluster • Kubernetes v1.30.3 • Kyverno v1.12.6

## The 8 policies & why banks enforce them

| # | Policy file                        | Type     | Rule                                                                 | Real banking / regulatory reason                                                                                  |
|---|------------------------------------|----------|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 1 | disallow-latest-tag.yaml           | validate | Block any image with `:latest`                                       | Prevents accidental production rollouts when base image is rebuilt. Auditors hate it.                              |
| 2 | require-requests-limits.yaml       | validate | Every container must declare requests & limits                       | One rogue pod can squeeze out all others → node OOM → service outage. Happened in real banks before.              |
| 3 | disallow-privileged.yaml           | validate | privileged: true forbidden                                          | Instant regulator violation + escape-to-host possible. Zero-trust environment.                                    |
| 4 | disallow-host-network-port.yaml    | validate | No hostNetwork, no hostPort                                          | Breaks pod network isolation; makes NetworkPolicy useless and opens side-channel attacks.                        |
| 5 | require-run-as-non-root.yaml       | validate | Containers must not run as UID 0                                     | Root on container ≈ root on host if any future breakout occurs. Required by PCI-DSS, SOC2, etc.                   |
| 6 | require-labels.yaml                | validate | app= and env= labels mandatory                                       | Without them: cost allocation fails, observability breaks, incident blast radius becomes the whole cluster.       |
| 7 | restrict-image-registries.yaml     | validate | Only internal Harbor/registry allowed                                | Public registries = supply-chain attack vector. Banks only trust images they built and scanned themselves.       |
| 8 | auto-add-default-limits.yaml       | mutate   | If limits missing → auto-inject 500m CPU / 512Mi memory              | “Silent safety net” — exactly what the bank PaaS did so developers never triggered OOM accidentally.              |

All **validate** policies → `validationFailureAction: Enforce` → immediate rejection on apply  
Policy 8 → **mutate** → automatically fixes missing limits (the magic developers never noticed in the bank)

## What happens when you remove them?
I tried. Results:
- No limits → pod OOMKilled in <40 seconds
- latest tag → would have caused unnoticed drift in production
- privileged container → would have failed every single security audit

This tiny repo is the reason banking clusters survive Black Friday and regulatory audits while staying boringly stable.

Feel free to clone, break the rules on purpose, and watch Kyverno reject you exactly like the bank platform did :)

Happy (and safe) hacking!

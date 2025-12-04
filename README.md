# Kubernetes Bank-Style Guardrails Lab

In my previous job at a bank, the internal PaaS forced strict rules.  
I rebuilt the most important ones on my own kubeadm cluster:

- LimitRange → no pod can use more than 2CPU/4Gi  
- Kyverno policy → block `:latest` tag (exactly like the bank)  
- Tested what happens without limits → immediate OOMKilled

This repo contains 8 production-grade Kyverno policies I rebuilt from real banking restrictions.
All tested on my own kubeadm cluster (v1.23.3).

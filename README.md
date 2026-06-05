# Retail Store Platform — Cluster Policies

This repo holds the cluster-level rules for the platform: who can do what, which services are allowed to talk to each other, and how much each namespace is allowed to use. It's all plain Kubernetes YAML managed with Kustomize, and ArgoCD applies it before any application gets deployed.

This is one of four repos. Start at **[retail-store-infra](https://github.com/erysimum/retail-store-infra)** for the full picture; that's the repo that builds the cluster this one configures.

---

## What's in here

Everything lives in `cluster-config/base/`:

```
cluster-config/base/
├── kustomization.yaml     # ties the resources below together
├── namespace.yaml         # the 5 app namespaces
├── rbac.yaml              # who can read, who can deploy
├── network-policy.yaml    # which service can talk to which
└── resource-quota.yaml    # CPU/memory/pod caps per namespace
```

**Namespaces.** Five: `ui-dev`, `catalog-dev`, `cart-dev`, `checkout-dev`, `orders-dev`. One per service.

**RBAC.** Two ClusterRoles and ten RoleBindings. Developers get a read-only role (get/list/watch). ArgoCD gets a deployer role with full create/update/delete. Each namespace gets both roles bound, so devs can look but only ArgoCD can change things.

**NetworkPolicy.** Every namespace starts with deny-all for both incoming and outgoing traffic, then opens up only the paths that are actually needed. For example, catalog only accepts traffic from UI and only reaches out to its database and DNS. Nothing can talk to anything unless a rule says so.

**ResourceQuota and LimitRange.** Each namespace is capped (CPU and memory requests, memory limits, and a max pod count). There are deliberately no CPU limits, only memory limits. The reasoning: a pod over its CPU "limit" just gets throttled (slow), while a pod over its memory limit gets killed. So we cap the dangerous one and let CPU burst.

---

## How it gets deployed

You don't apply this by hand. ArgoCD watches this repo and applies it as the first thing on the cluster (sync-wave 0), before any application. That ordering matters: the namespaces, quotas, and network rules have to exist before the apps land in them.

The ArgoCD `Application` that points here lives in the **[retail-store-gitops](https://github.com/erysimum/retail-store-gitops)** repo (`argocd/platform-app-dev.yaml`).


---

## License

MIT

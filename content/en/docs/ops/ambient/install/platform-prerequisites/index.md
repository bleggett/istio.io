---
title: Platform-Specific Prerequisites
description: Platform-Specific Prerequisites for Installing Istio Ambient Mode.
weight: 4
owner: istio/wg-environments-maintainers
test: no
---

This document covers any platform or environment specific prerequisites for installing Istio in ambient mode.

## CNI

### Cilium

1. Cilium currently defaults to proactively deleting other CNI plugins and their config, and must be configured with
`cni.exclusive = false` to properly support chaining. See https://docs.cilium.io/en/stable/helm-reference/ for more details.

1. Due to how Cilium manages node identity and internally allow-lists node-level health probes to pods,
applying default-DENY `NetworkPolicy` in a Cilium CNI install with Istio ambient mesh will cause `kubelet` health probes (which are by-default exempted from NetworkPolicy enforcement by Cilium) to be blocked.

    This can be resolved by applying the following `CiliumClusterWideNetworkPolicy`:

    {{< text syntax=yaml snip_id=none >}}
    apiVersion: "cilium.io/v2"
    kind: CiliumClusterwideNetworkPolicy
    metadata:
    name: "allow-ambient-hostprobes"
    spec:
    description: "Allows SNAT-ed kubelet health check probes into ambient pods"
    endpointSelector: {}
    ingress:
    - fromCIDR:
        - "169.254.7.127/32"
    {{< /text >}}

    Please see the [#49277](https://github.com/istio/istio/issues/49277) and [CiliumClusterWideNetworkPolicy](https://docs.cilium.io/en/stable/network/kubernetes/policy/#ciliumclusterwidenetworkpolicy) for more details.
# Try it: a minimal monitoring stack for Kubernetes

This repository contains configuration files to stand up a minimal monitoring
stack on a Kubernetes cluster, as well as a number of sample queries you can
test against your cluster. I tested these configurations and queries against
the latest 1.12 stable release on GKE (1.12.5-gke.10 as of the time of
writing).

This README assumes that you already have access to a Kubernetes cluster
running the 1.12 release. You can spin up a new cluster pretty quickly (it took
me about 10 minutes) using a free GKE trial. See [the Google Cloud Platform
console and developer docs][gke] for how to do that; there are also docs
available for how to install the `gcloud` command line client and `kubectl`
tool.

[gke]: https://console.cloud.google.com/kubernetes

## Set up `kube-state-metrics` (KSM)

[`kube-state-metrics`][ksm] is like a Prometheus adapter for your current
cluster state. It provides many [useful metrics][ksm-metrics] that can help you
understand your cluster's quality of service.

You can deploy the [configurations provided by upstream][ksm-yamls] directly
from the master branch. I tested using the 1.5.0 release of KSM. A copy of
the working 1.5.0 configurations is included in this repository for reference
under the `kube-state-metrics/` folder.

```bash
# Apply KSM configurations to the cluster
kubectl apply -f kube-state-metrics/

# Start a proxy to verify metrics
kubectl proxy &

# Curl the KSM metrics endpoint via the API server proxy
curl http://localhost:8001/api/v1/namespaces/kube-system/services/kube-state-metrics:8080/proxy/metrics
# Output:
# # HELP kube_configmap_info Information about configmap.
# # TYPE kube_configmap_info gauge
# kube_configmap_info{namespace="kube-system",configmap="ingress-uid"} 1
# kube_configmap_info{namespace="kube-system",configmap="extension-apiserver-authentication"} 1
# ...
```

[ksm]: https://github.com/kubernetes/kube-state-metrics
[ksm-metrics]: https://github.com/kubernetes/kube-state-metrics/tree/a6ff45fae22bdab03b1375fd454a9859bebd4d98/docs#exposed-metrics
[ksm-yamls]: https://github.com/kubernetes/kube-state-metrics/tree/a6ff45fae22bdab03b1375fd454a9859bebd4d98/kubernetes

## Set up Prometheus

[Prometheus][prometheus] is a time-series based monitoring system that scrapes
metrics on an interval from instrumented jobs. Most Kubernetes components
export metrics in Prometheus format by default.

While [`prometheus-operator`][prometheus-operator] is a convenient way to
launch Prometheus in a Kubernetes cluster, it is quite complex. Here, I've
included some simple, minimal configuration files to launch a Prometheus
instance on our cluster. These are available in the `prometheus/` folder in
this repo.

```bash
# Apply Prometheus configurations to the cluster
kubectl apply -f prometheus/

# Port-forward the Prometheus service for web access
kubectl port-forward -n kube-system svc/prometheus-server 8080:80 &
```

Now you can visit the Prometheus web UI in your browser at
http://127.0.0.1:8080/graph and try out some queries. Perhaps you could check if your API server is up: http://127.0.0.1:8080/graph?g0.range_input=1h&g0.expr=up%7Bjob%3D%22kubernetes-apiservers%22%7D&g0.tab=0

[prometheus]: https://prometheus.io/
[prometheus-operator]: https://github.com/coreos/prometheus-operator

# License

Copyright (c) 2019 Two Sigma Investments, LP.

Distributed under the Apache License 2.0. See the LICENSE file.

Contents of the `kube-state-metrics/` folder are copyright (c) 2016-2019 The
Linux Foundation via the Cloud Native Computing Foundation project and
distributed under the Apache License 2.0.

These files have been copied, unmodified, from the
[kubernetes/kube-state-metrics][ksm] repository. For full source history, you
can [view the upstream files][ksm-yamls] at the corresponding VCS commit.

The file `prometheus/server-config.yml` is a derivative work of the example Kubernetes configurations provided by the Prometheus project. It copies and modifies the provided [global][prom-global] and [Kubernetes-specific][prom-kube] configuration files.

[prom-global]: https://github.com/prometheus/prometheus/blob/release-2.8/documentation/examples/prometheus.yml
[prom-kube]: https://github.com/prometheus/prometheus/blob/release-2.2/documentation/examples/prometheus-kubernetes.yml

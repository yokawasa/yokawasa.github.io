---
layout: post
status: publish
published: true
title: Getting Kubernetes Metrics using kubectl
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2020-05-24 11:00:00 +0900'
tags:
- Kubernetes
- kubectl
- metricsAPI
- prometheus
---

There are various ways to obtain Kubernetes metrics which you can use to visualize, monitor and alert on your metrics. In reality you most likely use managed or unmanaged OSS metrics collection and visualization tools, but in this post, I introduce how to get raw Kubernetes metrics using kubectl.

## Kubernetes Raw Metrics via Prometheus metrics endpoint

Many Kubernetes components exposes their metrics via the `/metrics` endpoint, including API server, etcd and many other add-ons. These metrics are in [Prometheus format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md), and can be defined and exposed using [Prometheus client libs](https://prometheus.io/docs/instrumenting/clientlibs/)

```
kubectl get --raw /metrics
```

**Sample output - /metrics**

```
APIServiceOpenAPIAggregationControllerQueue1_adds 282663
APIServiceOpenAPIAggregationControllerQueue1_depth 0
APIServiceOpenAPIAggregationControllerQueue1_longest_running_processor_microseconds 0
APIServiceOpenAPIAggregationControllerQueue1_queue_latency{quantile="0.5"} 63
APIServiceOpenAPIAggregationControllerQueue1_queue_latency{quantile="0.9"} 105
APIServiceOpenAPIAggregationControllerQueue1_queue_latency{quantile="0.99"} 126
APIServiceOpenAPIAggregationControllerQueue1_queue_latency_sum 1.4331448e+07
APIServiceOpenAPIAggregationControllerQueue1_queue_latency_count 282663
APIServiceOpenAPIAggregationControllerQueue1_retries 282861
APIServiceOpenAPIAggregationControllerQueue1_unfinished_work_seconds 0
APIServiceOpenAPIAggregationControllerQueue1_work_duration{quantile="0.5"} 59
APIServiceOpenAPIAggregationControllerQueue1_work_duration{quantile="0.9"} 98
APIServiceOpenAPIAggregationControllerQueue1_work_duration{quantile="0.99"} 2003
APIServiceOpenAPIAggregationControllerQueue1_work_duration_sum 2.1373689e+07
APIServiceOpenAPIAggregationControllerQueue1_work_duration_count 282663
...
```

## Kubernetes Raw Metrics via metrics API

You can access Kubernetes [Metrics API](https://github.com/kubernetes/metrics) via kubectl proxy like this:

```
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes/<node-name>
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
```
(ref: [feiskyer/kubernetes-handbook](https://github.com/feiskyer/kubernetes-handbook/blob/master/en/addons/metrics.md#metrics-api))

Outputs via metrics API are unformated JSON, thus it's good to use `jq` to parse it.

**Sample output -  /apis/metrics.k8s.io/v1beta1/nodes**

```
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes | jq
```

```json

{
  "kind": "NodeMetricsList",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes"
  },
  "items": [
    {
      "metadata": {
        "name": "ip-xxxxxxxxxx.ap-northeast-1.compute.internal",
        "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/ip-xxxxxxxxxx.ap-northeast-1.compute.internal",
        "creationTimestamp": "2020-05-24T01:29:05Z"
      },
      "timestamp": "2020-05-24T01:28:58Z",
      "window": "30s",
      "usage": {
        "cpu": "105698348n",
        "memory": "819184Ki"
      }
    },
    {
      "metadata": {
        "name": "ip-yyyyyyyyyy.ap-northeast-1.compute.internal",
        "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/ip-yyyyyyyyyy.ap-northeast-1.compute.internal",
        "creationTimestamp": "2020-05-24T01:29:05Z"
      },
      "timestamp": "2020-05-24T01:29:01Z",
      "window": "30s",
      "usage": {
        "cpu": "71606060n",
        "memory": "678944Ki"
      }
    }
  ]
}

```


**Sample output - /apis/metrics.k8s.io/v1beta1/namespaces/NAMESPACE/pods/PODNAME**
```
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/cluster-autoscaler-7d8d69668c-5rcmt | jq
```

```json
{
  "kind": "PodMetrics",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "name": "cluster-autoscaler-7d8d69668c-5rcmt",
    "namespace": "kube-system",
    "selfLink": "/apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/cluster-autoscaler-7d8d69668c-5rcmt",
    "creationTimestamp": "2020-05-24T01:33:17Z"
  },
  "timestamp": "2020-05-24T01:32:58Z",
  "window": "30s",
  "containers": [
    {
      "name": "cluster-autoscaler",
      "usage": {
        "cpu": "1030416n",
        "memory": "27784Ki"
      }
    }
  ]
}
```

## See also
- https://github.com/kubernetes/metrics
- [feiskyer/kubernetes-handbook](https://github.com/feiskyer/kubernetes-handbook/blob/master/en/addons/metrics.md#metrics-api)
- https://prometheus.io/docs/instrumenting/clientlibs/
- [Prometheus format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)
- https://github.com/yokawasa/kubectl-tips

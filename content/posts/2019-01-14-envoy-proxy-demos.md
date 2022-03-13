---
author: Yoichi Kawasaki
date: "2019-01-14T19:32:40Z"
published: true
status: publish
tags:
- envoy
- envoyproxy
- servicemesh
- istio
- docker-compose
title: Set of Envoy Proxy features demos
---

Set of demos to demonstrate Envoy Proxy features

## HTTP Routing: Simple Match Routing

All traffic is routed by the `front envoy` to the `service containers`. Internally the traffic is routed to the service envoys, then the service envoys route the request to the flask app via the loopback address. In this demo, all traffic is routed to the service envoys like this:
- A request (path `/service/blue` & port `8000`) is routed to `service_blue`
- A request (path `/service/green` & port `8000`) is routed to `service_green`
- A request (path `/service/red` & port `8000`) is routed to `service_red`

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-httproute-simple-match.png)

Key definition 1 - `virtual_hosts` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-simple-match/front-envoy.yaml)
```yaml
    virtual_hosts:
    - name: backend
        domains:
        - "*"
        routes:
        - match:
            prefix: "/service/blue"
        route:
            cluster: service_green
        - match:
            prefix: "/service/blue"
        route:
            cluster: service_blue
        - match:
            prefix: "/service/red"
        route:
            cluster: service_red
```

Key definition 2 - `clusters` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-simple-match/front-envoy.yaml)
```yaml
  clusters:
  - name: service_blue
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_blue
        port_value: 80
  - name: service_green
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_green
        port_value: 80
  - name: service_red
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_red
        port_value: 80
```

To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-simple-match)


## HTTP Routing: Routing based on Header Condition

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-httproute-header-match.png)

Key definition 1 - `virtual_hosts` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-header-match/front-envoy.yaml)
```yaml
    virtual_hosts:
    - name: backend
        domains:
        - "*"
        routes:
        - match:
            prefix: "/service/blue"
            headers:
            - name: "x-canary-version"
                exact_match: "service_green"
        route:
            cluster: service_green
        - match:
            prefix: "/service/blue"
        route:
            cluster: service_blue
        - match:
            prefix: "/service/red"
        route:
            cluster: service_red
```

Key definition 2 - `clusters` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-header-match/front-envoy.yaml)
```yaml
  clusters:
  - name: service_blue
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_blue
        port_value: 80
  - name: service_green
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_green
        port_value: 80
  - name: service_red
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_red
        port_value: 80
```

To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-header-match)

## HTTP Routing: Blue Green Traffic Splitting

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-httproute-blue-green.png)

Key definition 1 - `virtual_hosts` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-blue-green/front-envoy.yaml)
```yaml
    virtual_hosts:
    - name: backend
        domains:
        - "*"
        routes:
        - match:
            prefix: "/service/blue"
        route:
            weighted_clusters:
            clusters:
            - name: service_green
                weight: 10
            - name: service_blue
                weight: 90
        - match:
            prefix: "/service/red"
        route:
            cluster: service_red
```

Key definition 2 - `clusters` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/httproute-blue-green/front-envoy.yaml)
```yaml
  clusters:
  - name: service_blue
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_blue
        port_value: 80
  - name: service_green
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_green
        port_value: 80
  - name: service_red
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service_red
        port_value: 80
```

To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-blue-green)


## Fault Injection

[Fault injection](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/fault_filter) is a technique for improving the coverage of a test by introducing faults to test code paths, in particular error handling code paths, that might otherwise rarely be followed. The `filter` ( [http_filters](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/http_filters) in the demo ) can be used to inject `delays` and `abort` requests with user-specified error codes.

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-fault-injection.png)

Front-proxy configurations are the same as the ones in [HTTP Routing: Simple Match Routing](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-simple-match). Differences from [HTTP Routing: Simple Match Routing](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-simple-match) are the following 2 fault injections:
- Fault injection `abort` (50% of requests will be aborted with 503 error code) for the requests to `service_red`
- Fault injection `delay` (50% of requests will be 10 seconds delayed) for the requests to `service_blue`

Key definition 1 - `http_filters` in [service-envoy-fault-injection-abort.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/fault-injection/service-envoy-fault-injection-abort.yaml)
```yaml
    http_filters:
    - name: envoy.fault
    config:
        abort:
        http_status: 503
        percentage:
            numerator: 50
            denominator: HUNDRED
```
> For `numerator` and `denominator` of `percentage`, see [type.FractionalPercent](https://www.envoyproxy.io/docs/envoy/latest/api-v2/type/percent.proto#envoy-api-msg-type-fractionalpercent)

Key definition 2 - `http_filters` in [service-envoy-fault-injection-delay.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/fault-injection/service-envoy-fault-injection-delay.yaml)
```yaml
    http_filters:
    - name: envoy.fault
    config:
        delay:
        type: fixed
        fixed_delay: 10s
        percentage:
            numerator: 50
            denominator: HUNDRED
```
> For `fixed_delay` and `percentage` of delay injection, see [config.filter.fault.v2.FaultDelay](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/fault/v2/fault.proto#envoy-api-msg-config-filter-fault-v2-faultdelay)


To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/fault-injection)

## Circuit Breaker

[Circuit Breaking](https://www.envoyproxy.io/learn/circuit-breaking) lets you configure failure thresholds that ensure safe maximums after which these requests stop. This allows for a more graceful failure, and time to respond to potential issues before they become larger.

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-circuit-breaker.png)

- Circuit Breaker is configured in `service_red` such that Circuit Breaker will be triggered with the following conditions:
  - Envoy makes more than `max connection#:1`
  - Envoy makes more than `max parallel requests#:1`

Key definition  - `clusters` in [service-envoy-circuitbreaker.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/circuit-breaker/service-envoy-circuitbreaker.yaml)
```yaml
  clusters:
  - name: local_service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    circuit_breakers:
      thresholds:
        max_connections: 1
        max_pending_requests: 1
        max_requests: 1
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 8080
```
> For the detail of `circuit_breakers` configuration, see [cluster.CircuitBreakers](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster/circuit_breaker.proto#envoy-api-msg-cluster-circuitbreakers)

To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/circuit-breaker)

## Timeouts and Retries

![](https://raw.githubusercontent.com/yokawasa/envoy-proxy-demos/master/assets/demo-timeouts-retries.png)

Front-proxy configurations are the same as the ones in [HTTP Routing: Simple Match Routing](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-simple-match). Differences from [HTTP Routing: Simple Match Routing](https://github.com/yokawasa/envoy-proxy-demos/tree/master/httproute-simple-match) are the following 2 additional behaviors:
- `Timeouts` (5 seconds) for the request to `service_blue`
- `Retries` that Envoy will attempt to do if `service_red` responds with any 5xx response code

For Service Containers, `delay` fault injection and `abort` fault injection are configured in `service_blue` and `service_red` respectively (which are the same configuration as the ones in [Fault Injection Demo](https://github.com/yokawasa/envoy-proxy-demos/tree/master/fault-injection))

Key definition - `virtual_hosts` in [front-envoy.yaml](https://github.com/yokawasa/envoy-proxy-demos/blob/master/timeouts-retries/front-envoy.yaml)
```yaml
    virtual_hosts:
    - name: backend
        domains:
        - "*"
        routes:
        - match:
            prefix: "/service/blue"
        route:
            cluster: service_blue
            timeout: 5s
        - match:
            prefix: "/service/green"
        route:
            cluster: service_green
        - match:
            prefix: "/service/red"
        route:
            cluster: service_red
            retry_policy:
            retry_on: "5xx"
            num_retries: 3
            per_try_timeout: 5s
```
> - `timeout`:  (Duration) Specifies the timeout for the route. If not specified, the default is 15s. For more detail, see `timeout` section in [RouteAction](https://www.envoyproxy.io/docs/envoy/v1.5.0/api-v2/rds.proto#routeaction)
> - `retry_policy` indicates the retry policy for all routes in this virtual host. For more detail on retry_policy, see [route.RetryPolicy](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto.html#envoy-api-msg-route-retrypolicy)


To continue, go to [github](https://github.com/yokawasa/envoy-proxy-demos/tree/master/timeouts-retries)

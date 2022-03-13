---
author: Yoichi Kawasaki
date: "2018-08-11T16:32:40Z"
published: true
status: publish
tags:
- kubernetes
- rbac
title: Accessing RBAC enabled Kubernetes Dashboard
---

This is an article on how you can configure Service Account and RoleBinding in order to make Dashbaord work. As of release Kubernetes v1.7, Dashboard no longer has full admin privileges granted by default. All the privileges are revoked and only minimal privileges granted, that are required to make Dashboard work. With default priviledge, you'll see the following errors showed up on the Dashboard.

![](/assets/20180811-rbac-dashboard-auth-error.png)

## [Azure Kubernetes Service (AKS)] RBAC is enabled by default 

Since Azure CLI version 2.0.40, RBAC is enabled by default. As you can see in the Azure command help, your cluster is RBAC enabled unless you specify `--disable-rbac` during the creation of the cluster. 

```
$ az --version
  azure-cli (2.0.43)

$ az aks create --help

    --disable-rbac                 : Disable Kubernetes Role-Based Access Control.
    --enable-rbac -r  [Deprecated] : Enable Kubernetes Role-Based Access Control. Default:
                                     enabled.
        Argument 'enable_rbac' has been deprecated and will be removed in a future release. Use
        '--disable-rbac' instead.
```

## Option 1: Access to Dashboard with your Service Account

In option 1, I introduce how to give priviledge your Service Account and access to the Dashboard with the account. Actually there are a couple of authorization options, and here I introduce how to authorize with Bear Token

[NOTE] According to [this](https://github.com/kubernetes/dashboard/wiki/Access-control#authentication), 

> As of release 1.7 Dashboard supports user authentication based on:
> 
> - Authorization: Bearer <token> header passed in every request to Dashboard. Supported from release 1.6. Has the highest priority. If present, login view will not be shown.
> - [Bearer Token](https://github.com/kubernetes/dashboard/wiki/Access-control#bearer-token) that can be used on Dashboard **login view**.
> - [Username/password](https://github.com/kubernetes/dashboard/wiki/Access-control#basic) that can be used on Dashboard **login view** (Disabled by default)
> - [Kubeconfig](https://github.com/kubernetes/dashboard/wiki/Access-control#kubeconfig) file that can be used on Dashboard **login view** 


### 1-1. Create your Service Account for Dashboard access 

First of all, create your Service Account `my-admin-user` like this:
```
$ kubectl create serviceaccount my-admin-user -n kube-system
```

Or you can create the Service Account with the following YAML `my-sa.yaml` and deploying it with `kubectl create -f my-sa.yaml`:

```
# my-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-admin-user
  namespace: kube-system
```

Check if your Service Account (`my-admin-user`) has been added.

```
$ kubectl get sa -n kube-system

NAME                                                          SECRETS   AGE
addon-http-application-routing-external-dns                   1         9d
addon-http-application-routing-nginx-ingress-serviceaccount   1         9d
attachdetach-controller                                       1         9d
certificate-controller                                        1         9d
clusterrole-aggregation-controller                            1         9d
cronjob-controller                                            1         9d
daemon-set-controller                                         1         9d
default                                                       1         9d
deployment-controller                                         1         9d
disruption-controller                                         1         9d
endpoint-controller                                           1         9d
generic-garbage-collector                                     1         9d
heapster                                                      1         9d
horizontal-pod-autoscaler                                     1         9d
job-controller                                                1         9d
kube-dns                                                      1         9d
kube-proxy                                                    1         9d
kube-svc-redirector                                           1         9d
kubernetes-dashboard                                          1         9d
my-admin-user                                                 1         15s
namespace-controller                                          1         9d
node-controller                                               1         9d
persistent-volume-binder                                      1         9d
pod-garbage-collector                                         1         9d
pv-protection-controller                                      1         9d
pvc-protection-controller                                     1         9d
replicaset-controller                                         1         9d
replication-controller                                        1         9d
resourcequota-controller                                      1         9d
route-controller                                              1         9d
service-account-controller                                    1         9d
service-controller                                            1         9d
statefulset-controller                                        1         9d
ttl-controller                                                1         9d
tunnelfront                                                   1         9d
```

### 1-2. Binding the role cluster-admin to the Service Account
Create a `ClusterRoleBinding` which gives the role `cluster-admin` (= full admin priviledge) to the ServiceAccount `my-admin-user`

```
$ kubectl create clusterrolebinding my-admin-user -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:my-admin-user
```

Or you can grant the priviledges with the following YAML `my-sa-binding.yaml` and deploying it with `kubectl create -f my-sa-binding.yaml`:
```
# my-sa-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: my-admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: my-admin-user
  namespace: kube-system
```

### 1-3. Authentication Option - Give Bear Token at Dashboard login view
#### 1-3-1. Get the Token of the ServiceAccount

```
$ kubectl get secret $(kubectl get serviceaccount my-admin-user -n kube-system -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" -n kube-system | base64 --decode
```

Or you can obtain the token step by step like this:

```
# Get secret name for my-admin-user
$ kubectl get serviceaccount my-admin-user -n kube-system -o yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2018-08-11T06:53:34Z
  name: my-admin-user
  namespace: kube-system
  resourceVersion: "1075968"
  selfLink: /api/v1/namespaces/kube-system/serviceaccounts/my-admin-user
  uid: 44142169-9d33-11e8-b7d0-de454880a5dc
secrets:
- name: my-admin-user-token-nzp4f


# Get secret string and base64 decoded it
$ kubectl get secret my-admin-user-token-nzp4f -n kube-system -o jsonpath="{.data.token}" | base64 --decode

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJteS1hZG1pbi11c2VyLXRva2VuLW56cDRmIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Im15LWFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI0NDE0MjE2OS05ZDMzLTExZTgtYjdkMC1kZTQ1NDg4MGE1ZGMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06bXktYWRtaW4tdXNlciJ9.2ulwCWd7MOnQjoecAY2NoleIxcHD8tda97ud3cK-kHWmdCodAejvddA4YjYozzBu2bWNA83aVTvKAn5-Uv1DC47U5FPh2LXAXNPXn4PyrdLO7TFZdHYmkvUgJKsg25vJvJmsWF9eQOinjjh_g16aGgdxrWz0NGJz5d1eE5GDP5NXXTTgxXlD_GFQduhlq8kc89dhpDUXMYe60-KzZvNaQhIskPsnxHMix1JrHEdtfciFhHRb2CBNjPWfcg455NGoS9S-k0qTfoIHYJC627p75E8TGqyTIa8TSg8vaif4XWgeg_OZWqEIGHTIrhEAGO4ElFijdZuzAg2-v9BGWe8i4q1i70ca5CwReJTG8t13eeOoEkq--VbhDAMY6rxmx-hi9dwf-zjsD233MdHJLh1yRi0eo_k5ov7fwDDsLQXeCTBIjSAzorvXseWr5m9sQ7yREbjDXCOsHbYo5xNV5ii-yOlxYyiqPxZZnnSwzllj1lwPDLSL0MyxkR9siF52vbkNDe6qdYYMqPtA-jTMIw_iLlB-WeN1Fx8423c4x5wV6IGPJZFuOYZhB0ra4jfRSS39vesaNodW8RjHUiuOSVA8_j-DxwOxa8prynALFWGswSMy6PfVQydouU6vammeqPBel9-IqBeTXY-57YumELG1PdcOcxdrBCZUlxBvJWbItxA
```

#### 1-3-2. Give Bear Token at Dashboard login view

Now that you have a bear token that you need to grant the full priviledges to your Service Account, let's access to the Dashboard login view. 

First of all, create a proxy to the dashboard:

```
$ kubectl proxy

Starting to serve on 127.0.0.1:8001
```
Then, access to the Dashboard login page:

```
$ open http://localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/login
```

Or if it's on Azure, you can leverage Azure CLI command to access the Dashboard like this:
```
$ az aks browse --resource-group <RESOURCE_GROUP> --name <CLUSTER_NAME>
$ open http://127.0.0.1:8001/#!/login
```

You will see the following Dashboard login view. Choose `Token` option and enter the bear token you got above. You will be able to access and operate with the Dashboard without any errors.
![](/assets/20180811-rbac-dashboard-login-view.png)


#### Easy login by giving Authorization header using Browser extention

As introduced in [here](https://github.com/kubernetes/dashboard/wiki/Access-control#authorization-header), install [Requestly browser plugin](https://chrome.google.com/webstore/detail/requestly-redirect-url-mo/mdnleldcmiljblolnjhpnblkcekpdkpa) and configure to make Dashboard use authorization header. You simply need to configure the plugin to pass the following header in accessing the dashboard:

```
Authorization: Bearer <token>
```
HERE is the example screen shot:

![](/assets/20180811-rbac-requestly-sample-config.png)


## Option2: Granting admin privileges to Dashboard's Service Account

In Option 2, I introduce how to give full privilege (role: `cluster-admin`) to the Dashboard's Service Account `kubernetes-dashboard`. With this option, you can skip the authorization process that you do in the option 1 to access Dashboard. However, as mentioned [here](https://github.com/kubernetes/dashboard/wiki/Access-control#admin-privileges), granting admin privileges to Dashboard's Service Account might be a security risk.


First of all, create a `ClusterRoleBinding` which gives the role `cluster-admin` (= full admin priviledge) to the ServiceAccount `kubernetes-dashboard`

```
$ kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```

Or you can do the same with the following YAML `dashboard-sa-binding.yaml` and deploying it with `kubectl create -f dashboard-sa-binding.yaml`:
```
# dashboard-sa-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

Finally, access to the Dashboard. You'll be able to access and operate without any errors.


## LINKS
- [Access Control for Dashboard](https://github.com/kubernetes/dashboard/wiki/Access-control)
- [Service Account Permissions in Using RBAC Authorization @ kubernetesio](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions),

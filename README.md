# kubectl

  599  kubectl get svc
  
  600  kubectl get pod -n kube-system -o wide
  
  601  kubectl get pod kubernetes-dashboard-2039414953-g9qk0 -n kube-system -o wide
  
  602  kubectl get pod kubernetes-dashboard-2039414953-g9qk0 -n kube-system -o yaml
  
  603  kubectl get svc --all-namespaces

## service/svc
$  kubectl get svc kubernetes-dashboard -n kube-system -o json
```
{
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "creationTimestamp": "2017-06-27T21:04:45Z",
        "labels": {
            "k8s-app": "kubernetes-dashboard"
        },
        "name": "kubernetes-dashboard",
        "namespace": "kube-system",
        "resourceVersion": "302",
        "selfLink": "/api/v1/namespaces/kube-system/services/kubernetes-dashboard",
        "uid": "3f6272e8-5b7c-11e7-b20c-286ed488c97d"
    },
    "spec": {
        "clusterIP": "10.105.167.152",
        "ports": [
            {
                "port": 80,
                "protocol": "TCP",
                "targetPort": 9090
            }
        ],
        "selector": {
            "k8s-app": "kubernetes-dashboard"
        },
        "sessionAffinity": "None",
        "type": "ClusterIP"
    },
    "status": {
        "loadBalancer": {}
    }
}
```

$ kubectl describe svc kubernetes-dashboard -n kube-system
```
Name:                   kubernetes-dashboard
Namespace:              kube-system
Labels:                 k8s-app=kubernetes-dashboard
Annotations:            <none>
Selector:               k8s-app=kubernetes-dashboard
Type:                   ClusterIP
IP:                     10.105.167.152
Port:                   <unset> 80/TCP
Endpoints:              10.244.0.9:9090
Session Affinity:       None
Events:                 <none>
```
It has Endpoints which "get svc" doesn't have.

# debug dashboard behind a proxy

## Normal dashboard docker log
docker logs <dashboard container id>
```
# for the default ui page --, which shows the workload page

[2017-06-24T05:55:53Z] Incoming HTTP/1.1 GET /api/v1/workload/default?filterBy=&itemsPerPage=10&page=1&sortBy=d,creationTimestamp request from 10.244.0.1:37598: {}
Getting lists of all workloads
Getting pod metrics
[2017-06-24T05:55:53Z] Outcoming response to 10.244.0.1:37598 with 200 status code
```
If we couldn't see the above log lines, then the request doesn't reach the dashboard container yet.
For example, this one doesn't not have request logs:
```
$ docker logs 7767c9a33f57
Using HTTP port: 8443
Creating API server client for https://10.96.0.1:443
Successful initial request to the apiserver, version: v1.6.6
Creating in-cluster Heapster client
Using service account token for csrf signing
```

## Look at the proxy environment variables in apiserver
```
$ docker inspect 64848a86a5a0

```

## Look at the dashboard service endpoints:
```
$ kubectl describe svc kubernetes-dashboard -n kube-system
Name:                   kubernetes-dashboard
Namespace:              kube-system
Labels:                 k8s-app=kubernetes-dashboard
Annotations:            <none>
Selector:               k8s-app=kubernetes-dashboard
Type:                   ClusterIP
IP:                     10.110.52.122
Port:                   <unset> 80/TCP
Endpoints:              10.244.0.9:9090
Session Affinity:       None
Events:                 <none>
```

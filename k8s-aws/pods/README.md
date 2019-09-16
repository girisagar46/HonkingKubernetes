## Theory

POD are the smallest unit of deployment in k8s. Generally container to pod ratio is 1:1 (i.e. one container inside a pod) but in some advance case, more than 2 container can run inside it.

## practical

Contents inside `first-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```

### Commands related to pods

Deploy the pod to the cluster using the manifest file:
```bash
$ kubectl apply -f first-pod.yml
pod "webapp" created
```

Get the pod info:
```bash
$ kubectl get pod webapp
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          57s
```

Describe the pod:
```bash
$ kubectl describe pod webapp
Name:               webapp
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Mon, 16 Sep 2019 21:27:22 +0900
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"webapp","namespace":"default"},"spec":{"containers":[{"image":"richar...
Status:             Running
IP:                 172.17.0.4
Containers:
  webapp:
    Container ID:   docker://6753e70ebfc0c2ef4dd72f69f29069f578a55212e7698c46c0769a7ee603838f
    Image:          richardchesterwood/k8s-fleetman-webapp-angular:release0
    Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-webapp-angular@sha256:9b98fec20772bd1d7d4c9085048f28af35b31ad3a7b7d3ba395fb512c5c359e6
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 16 Sep 2019 21:27:48 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-slw6s (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-slw6s:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-slw6s
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                    From               Message
  ----     ------            ----                   ----               -------
  Warning  FailedScheduling  4h56m (x4 over 5h1m)   default-scheduler  no nodes available to schedule pods
  Warning  FailedScheduling  8m20s (x2 over 8m20s)  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
  Normal   Scheduled         8m16s                  default-scheduler  Successfully assigned default/webapp to minikube
  Normal   Pulling           8m14s                  kubelet, minikube  Pulling image "richardchesterwood/k8s-fleetman-webapp-angular:release0"
  Normal   Pulled            7m50s                  kubelet, minikube  Successfully pulled image "richardchesterwood/k8s-fleetman-webapp-angular:release0"
  Normal   Created           7m50s                  kubelet, minikube  Created container webapp
  Normal   Started           7m50s                  kubelet, minikube  Started container webapp
```


Connect to the pod. Just like connecting to docker container:
```bash
$ kubectl exec webapp ls
bin
dev
etc
home
lib
media
...
```

Even though the pod is up and running, if you visit cluster IP address (`$ minikube ip`), the page won't be loaded. Because we do not interact directly with the pod. Exposing the pod to the outside world is done by k8s service. 

But the webapp is running locally inside the pod. Let's see.

First `sh` into the pod, then do `wget localhost`
```bash
$ kubectl exec -it webapp sh

/ # wget localhost
Connecting to localhost (127.0.0.1:80)
index.html           100% |****************************************************************************|   585   0:00:00 ETA
/ # 
```




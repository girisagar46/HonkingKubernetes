## Theory

### Points to note:

- Service is a long running object in k8s
- Service will have an IP address and stable fixed port
- We can attach service to pods
- Services help expose pods to the public internet
- From outside world, we connect to service and service redirects request to appropriate pod
- Service and pods are tied together by **Label**
- When we write out service we provide `selector` and `selector` is nothing but series of key:value pair. In the runtime in k8s cluster, service will look for any matching key:value pair amongst the pod and matching selector key:value pairs are tied together with the service. And then service redirects all requests to those pods with the same selector pairs
- When we release new version of our app and deploy it in pod. And we want out client to use the new version, then we just change the label in service to match with the label of our new pod.

## Practical

Edit the `first-pod.yml` to add labels
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```

Create new `webapp-service.yml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: fleetman-webapp
spec:
  # This should match with the pod which you want to tie service and pod
  selector:
    app: webapp
  ports: 
    - port: 80
      name: http
      nodePort: 30001 # Since we are using NodePort type and used only for local development environment.
  # In production environment, we use the type of LoadBalancer
  # Using ClusterIP means that this service is only accessible from within the cluster. Using ClusterIP doesn't expose our service to outside world. It is suitable for micro service architecture which is just making it private (internal service). 
  # But, we want to expose our webapp to public and access from browser. Hence we need to use NodePort
  type: NodePort 
```

Now re-configure the pod because we've added label:
```bash
$ kubectl apply -f first-pod.yml 
pod/webapp configured
```

Then deploy the service:
```bash
$ kubectl apply -f webapp-service.yml 
service/fleetman-webapp created
```

Verify changes:
```bash
$ kubectl get all       
NAME         READY   STATUS    RESTARTS   AGE
pod/webapp   1/1     Running   0          6h8m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/fleetman-webapp   NodePort    10.107.214.187   <none>        80:30001/TCP   22s
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        11d
```

Find the minikube ip address:
```bash
$ minikube ip
192.168.99.101
```

Now visit, `192.168.99.101:30001` in your web browser. You'll see the output. 

Let's say, we now completed a new version of our app. And we want to deploy in a new pod. Let's create a new pod definition inside the `first-pod.yml` file. 

first-pod.yml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
    release: 0
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0

---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
    release: "0-5"
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```

We then change service accordingly:
webapp-service.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: fleetman-webapp
spec:
  # This should match with the pod which you want to tie service and pod
  selector:
    app: webapp
    release: "0-5"
  ports:
    - port: 80
      nodePort: 30001
  type: NodePort
```

Now, apply changes to pods:
```bash
$ kubectl apply -f first-pod.yml
pod/webapp configured
pod/webapp-relase-0-5 created
```

Also, apply changes to service:
```bash
$ kubectl apply -f webapp-service.yml 
service/fleetman-webapp configured
```

Review changes:
```bash
$ kubectl get all
NAME                    READY   STATUS    RESTARTS   AGE
pod/webapp              1/1     Running   0          91s
pod/webapp-relase-0-5   1/1     Running   0          91s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/fleetman-webapp   NodePort    10.107.214.187   <none>        80:30001/TCP   18m
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        11d
```

Describe service:
```bash
$ kubectl describe svc fleetman-webapp
Name:                     fleetman-webapp
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"fleetman-webapp","namespace":"default"},"spec":{"ports":[{"nodePo...
Selector:                 app=webapp,release=0-5
Type:                     NodePort
IP:                       10.107.214.187
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30001/TCP
Endpoints:                172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Note: You might not see the changes right away because pods are being created and images are being downloaded. But, after certain time, after the new pod is up and running successfully, and if you go to the same IP:PORT as before, you'll see new changes.

The best practice is to make sure all your pods are up and running correctly and after that apply the service configuration.

Get all pods with labels:
```yaml
$ kubectl get po --show-labels
NAME                READY   STATUS    RESTARTS   AGE     LABELS
webapp              1/1     Running   0          7m44s   app=webapp,release=0
webapp-relase-0-5   1/1     Running   0          7m44s   app=webapp,release=0-5
```

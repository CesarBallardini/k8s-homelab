# README - minikube lab

## Boot the VM

```bash
vagrant up
vagrant reload
```

The provision installs Docker, Kubectl and Minikube.

After that, initializes minikube cluster using Docker as hypervisor.

ssh into the VM:

```bash
vagrant ssh
```

## Get some status

```bash
# get status
kubectl get nodes
# NAME       STATUS   ROLES                  AGE   VERSION
# minikube   Ready    control-plane,master   85s   v1.20.2


kubectl get pod
# No resources found in default namespace.


kubectl get services
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   94s
```



## Create a pod

`kubectl create --help` shows the components that can be created.

A pod is the smallest unit of work in k8s, but you do not create pods but deployments.

```
kubectl create deployment NAME --image=IMAGE [--dry-run] [options]
```


```bash
kubectl create deployment nginx-depl --image=nginx
# deployment.apps/nginx-depl created


kubectl get deployment
# NAME         READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-depl   1/1     1            1           18s


kubectl get pod
# NAME                          READY   STATUS    RESTARTS   AGE
# nginx-depl-5c8bf76b5b-g9rlw   1/1     Running   0          10m

```

A deployment is a blueprint for creating pods


```bash
kubectl get replicaset
# NAME                    DESIRED   CURRENT   READY   AGE
# nginx-depl-5c8bf76b5b   1         1         1       12m

##
# note the names:
#
# deployment:  nginx-depl
# replica set: nginx-depl-5c8bf76b5b
# pod:         nginx-depl-5c8bf76b5b-g9rlw

```

Replicaset manages the replicas of a Pod. You don't work with replicasets, you do your config with deployments.

Layers of abstraction:
* Deployment manages a ReplicaSet
* ReplicaSet manages all Pos
* Pod is an abstraction of a container
* Container 


* Change the image

```bash
kubectl edit deployment nginx-depl
```

Search the image and add a specific version to it (1.16), save the file

```yaml
    spec:
      containers:
      - image: nginx:1.16
```

```bash
kubectl get pod
# NAME                          READY   STATUS        RESTARTS   AGE
# nginx-depl-5c8bf76b5b-g9rlw   0/1     Terminating   0          23m
# nginx-depl-7fc44fc5d4-nk75r   1/1     Running       0          26s


kubectl get replicaset
# NAME                    DESIRED   CURRENT   READY   AGE
# nginx-depl-5c8bf76b5b   0         0         0       24m
# nginx-depl-7fc44fc5d4   1         1         1       2m7s

```

## Debugging Pods

```bash
kubectl create deployment mongo-depl --image=mongo
# deployment.apps/mongo-depl created

kubectl get pod
# NAME                          READY   STATUS    RESTARTS   AGE
# mongo-depl-5fd6b7d4b4-lxjzn   1/1     Running   0          38s
# nginx-depl-7fc44fc5d4-nk75r   1/1     Running   0          5m21s



kubectl describe pod  mongo-depl-5fd6b7d4b4-lxjzn 
# Name:         mongo-depl-5fd6b7d4b4-lxjzn
# Namespace:    default
# Priority:     0
# Node:         minikube/192.168.49.2
# Start Time:   Fri, 05 Mar 2021 00:00:22 +0000
# Labels:       app=mongo-depl
#               pod-template-hash=5fd6b7d4b4
# Annotations:  <none>
# Status:       Running
# IP:           172.17.0.3
# IPs:
#   IP:           172.17.0.3
# Controlled By:  ReplicaSet/mongo-depl-5fd6b7d4b4
# Containers:
#   mongo:
#     Container ID:   docker://8c514a7736228b076fe38ef971e1144749241e5fe5c94ddde0c9d0f83c57e444
#     Image:          mongo
#     Image ID:       docker-pullable://mongo@sha256:51f6fdbfc622d91e276ade7e6cf6491aa36ff2bd9b158dadb732f9e4a05f33ad
#     Port:           <none>
#     Host Port:      <none>
#     State:          Running
#       Started:      Fri, 05 Mar 2021 00:00:59 +0000
#     Ready:          True
#     Restart Count:  0
#     Environment:    <none>
#     Mounts:
#       /var/run/secrets/kubernetes.io/serviceaccount from default-token-77v9f (ro)
# Conditions:
#   Type              Status
#   Initialized       True 
#   Ready             True 
#   ContainersReady   True 
#   PodScheduled      True 
# Volumes:
#   default-token-77v9f:
#     Type:        Secret (a volume populated by a Secret)
#     SecretName:  default-token-77v9f
#     Optional:    false
# QoS Class:       BestEffort
# Node-Selectors:  <none>
# Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
#                  node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
# Events:
#   Type    Reason     Age   From               Message
#   ----    ------     ----  ----               -------
#   Normal  Scheduled  2m9s  default-scheduler  Successfully assigned default/mongo-depl-5fd6b7d4b4-lxjzn to minikube
#   Normal  Pulling    2m8s  kubelet            Pulling image "mongo"
#   Normal  Pulled     93s   kubelet            Successfully pulled image "mongo" in 35.123398603s
#   Normal  Created    92s   kubelet            Created container mongo
#   Normal  Started    92s   kubelet            Started container mongo
# 


kubectl logs mongo-depl-5fd6b7d4b4-lxjzn
# lots of log lines


kubectl exec -it mongo-depl-5fd6b7d4b4-lxjzn -- /bin/bash
# an interactive terminal on the mongo container

```


# References

* https://kubernetes.io/es/docs/tasks/tools/install-kubectl/
* https://minikube.sigs.k8s.io/docs/start/
* https://www.youtube.com/watch?v=X48VuDVv0do Kubernetes Tutorial for Beginners [FULL COURSE in 4 Hours]


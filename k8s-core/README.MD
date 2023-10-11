# K8s Core Objects Lab

## Pods

In Kubernetes, a group of one or more containers is called a pod. Containers in a pod are deployed together, and are started, stopped, and replicated as a group. The simplest pod definition describes the deployment of a single container. For example, an nginx web server pod might be defined as such

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
  namespace: default
  labels:
    run: nginx
spec:
  containers:
  - name: mynginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

A pod definition is a declaration of a desired state. Desired state is a very important concept in the Kubernetes model. Many things present a desired state to the system, and it is Kubernetes’ responsibility to make sure that the current state matches the desired state. For example, when you create a Pod, you declare that you want the containers in it to be running. If the containers happen to not be running (e.g. program failure, …), Kubernetes will continue to (re-)create them for you in order to drive them to the desired state. This process continues until the Pod is deleted.

```bash
vagrant@master-node:~$ kubectl create -f pod-nginx.yaml
pod/mynginx created
vagrant@master-node:~$
```

list all the pods

```bash
vagrant@master-node:~$ kubectl get po -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP              NODE            NOMINATED NODE   READINESS GATES
mynginx   1/1     Running   0          2m18s   172.16.87.193   worker-node01   <none>           <none>
vagrant@master-node:~$
```

describe the pod:

```bash
vagrant@master-node:~$ kubectl describe po mynginx
Name:             mynginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             worker-node01/192.168.56.21
Start Time:       Tue, 10 Oct 2023 10:43:26 +0000
Labels:           run=nginx
Annotations:      cni.projectcalico.org/containerID: 305e64d72b3dd55cc99c39ac3ce87f57177683f0b9a5150dd9987f3e6b73e0b3
                  cni.projectcalico.org/podIP: 172.16.87.193/32
                  cni.projectcalico.org/podIPs: 172.16.87.193/32
Status:           Running
IP:               172.16.87.193
IPs:
  IP:  172.16.87.193
Containers:
  mynginx:
    Container ID:   cri-o://ee99fb61aac9a091252b76ef2d08ec16f4229854eff0a7b060a7bcf949c4695f
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:32da30332506740a2f7c34d5dc70467b7f14ec67d912703568daff790ab3f755
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 10 Oct 2023 10:44:34 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j42ft (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-j42ft:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m11s  default-scheduler  Successfully assigned default/mynginx to worker-node01
  Normal  Pulling    3m11s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     2m4s   kubelet            Successfully pulled image "nginx:latest" in 1m6.427288057s (1m6.427302625s including waiting)
  Normal  Created    2m4s   kubelet            Created container mynginx
  Normal  Started    2m4s   kubelet            Started container mynginx
vagrant@master-node:~$
```

Now delete the Pod:

```bash
vagrant@master-node:~$ kubectl delete po mynginx
pod "mynginx" deleted
vagrant@master-node:~$
```

In the example above, we had a pod with a single container nginx running inside.

Kubernetes let's user to have multiple containers running in a pod. All containers inside the same pod share the same resources, e.g. network and volumes and are always scheduled togheter on the same node. The primary reason that Pods can have multiple containers is to support helper applications that assist a primary application. Typical examples of helper applications are data pullers, data pushers, and proxies. Helper and primary applications often need to communicate with each other, typically through a shared filesystem or loopback network interface.

## Labels

In Kubernetes, labels are a system to organize objects into groups. Labels are key-value pairs that are attached to each object. Label selectors can be passed along with a request to the apiserver to retrieve a list of objects which match that label selector.

To add a label to a pod, add a labels section under metadata in the pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
  namespace: default
  labels:
    run: nginx
    type: webserver
spec:
  containers:
  - name: mynginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Here, we have added two labels to the pod. Create the Pod:

```bash
vagrant@master-node:~$ kubectl create -f pod-nginx-label.yaml
pod/mynginx created
vagrant@master-node:~$
```

You can then get the pod by mentioning the label in the get pod command:

```bash
vagrant@master-node:~$ kubectl get po -l type=webserver
NAME      READY   STATUS    RESTARTS   AGE
mynginx   1/1     Running   0          18s
vagrant@master-node:~$
```

You can also add labels dynamically to Kubernetes objects like Pods, nodes etc. Please execute the following commands in order to implement the same:

```bash
vagrant@master-node:~$ kubectl label pod mynginx tier=fe
pod/mynginx labeled
vagrant@master-node:~$ kubectl get po -l tier=fe
NAME      READY   STATUS    RESTARTS   AGE
mynginx   1/1     Running   0          2m20s
vagrant@master-node:~$ kubectl label node worker-node01 rack=R1
node/worker-node01 labeled
vagrant@master-node:~$ kubectl label node worker-node02 rack=R2
node/worker-node02 labeled
vagrant@master-node:~$ kubectl get nodes -l rack=R1
NAME            STATUS   ROLES    AGE   VERSION
worker-node01   Ready    worker   50m   v1.27.1
vagrant@master-node:~$ kubectl get nodes -l rack=R2
NAME            STATUS   ROLES    AGE   VERSION
worker-node02   Ready    worker   46m   v1.27.1
vagrant@master-node:~$
```


## Replicasets

A Replicaset ensures that a specified number of pod replicas are running at any one time. In other words, a Replicaset makes sure that a pod or homogeneous set of pods are always up and available. If there are too many pods, it will kill some. If there are too few, it will start more. Unlike manually created pods, the pods maintained by a Replicaset are automatically replaced if they fail, get deleted, or are terminated.

A Replicaset configuration consists of:

    - The number of replicas desired
    - The pod definition
    - The selector to bind the managed pod

A selector is a label assigned to the pods that are managed by the replicaset. Labels are included in the pod definition that the replicaset instantiates. The replicaset uses the selector to determine how many instances of the pod are already running in order to adjust as needed.

Observe the below yaml file:

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
      name: nginx
      labels:
        run: nginx
        type: front-end
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
  selector:
        matchLabels:
            type: front-end
```

Create and see the RS as well as the pods that were created by issuing the following set of commands:

```bash
vagrant@master-node:~$ kubectl create -f nginx-rs.yaml
replicaset.apps/nginx created
vagrant@master-node:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   3         3         1       4s
vagrant@master-node:~$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
mynginx       1/1     Running   0          13m
nginx-2qksr   1/1     Running   0          8s
nginx-5mlvj   1/1     Running   0          8s
nginx-njzvc   1/1     Running   0          8s
vagrant@master-node:~$ kubectl describe rs nginx
Name:         nginx
Namespace:    default
Selector:     type=front-end
Labels:       <none>
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  run=nginx
           type=front-end
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  107s  replicaset-controller  Created pod: nginx-5mlvj
  Normal  SuccessfulCreate  107s  replicaset-controller  Created pod: nginx-njzvc
  Normal  SuccessfulCreate  107s  replicaset-controller  Created pod: nginx-2qksr
vagrant@master-node:~$
```

The ReplicaSet makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling control agent, by simply updating the replicas field. For example, scale dow to zero replicas in order to delete all pods controlled by a given replicaset

```bash
vagrant@master-node:~$ kubectl scale rs nginx --replicas=0
replicaset.apps/nginx scaled
vagrant@master-node:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   0         0         0       3m8s
vagrant@master-node:~$ kubectl get po
NAME      READY   STATUS    RESTARTS   AGE
mynginx   1/1     Running   0          16m
vagrant@master-node:~$
```

We can also scale out the ReplicaSet to create new Pods:

```bash
vagrant@master-node:~$ kubectl scale rs nginx --replicas=5
replicaset.apps/nginx scaled
vagrant@master-node:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   5         5         0       3m59s
vagrant@master-node:~$ kubectl get po
NAME          READY   STATUS              RESTARTS   AGE
mynginx       1/1     Running             0          17m
nginx-cphrk   0/1     ContainerCreating   0          8s
nginx-mm8rh   0/1     ContainerCreating   0          8s
nginx-tzvmz   1/1     Running             0          8s
nginx-vk5n5   0/1     ContainerCreating   0          8s
nginx-z2ld7   1/1     Running             0          8s
vagrant@master-node:~$
```

Also in case of failure of a node, the replicaset takes care of keep the same number of pods by scheduling the containers running on the failed node to the remaining nodes in the cluster.

To delete a replicaset:

```bash
vagrant@master-node:~$ kubectl delete rs nginx
replicaset.apps "nginx" deleted
vagrant@master-node:~$
```

## Deployments

A Deployment provides declarative updates for pods and replicas. You only need to describe the desired state in a Deployment object, and it will change the actual state to the desired state. The Deployment object defines the following details:

    - The elements of a ReplicaSet definition
    - The strategy for transitioning between deployments

To create a deployment for our nginx webserver, use the nginx-deploy.yaml file which has the following contents

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

Then delete the Pod which we created during step 2 for the Pod labelling excercise as it has the same label as the pods that will be deployed by this deployment:

```bash
vagrant@master-node:~$ kubectl delete po mynginx
pod "mynginx" deleted
vagrant@master-node:~$
```

and then create the deployment:

```bash
vagrant@master-node:~$ kubectl create -f nginx-deploy.yaml
deployment.apps/nginx created
vagrant@master-node:~$
```

The deployment creates the following objects:

```bash
vagrant@master-node:~$ kubectl get all -l run=nginx
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-54d484cf5c-qvttb   1/1     Running   0          5m34s
pod/nginx-54d484cf5c-tb7z7   1/1     Running   0          5m34s
pod/nginx-54d484cf5c-trrx6   1/1     Running   0          5m34s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           5m34s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-54d484cf5c   3         3         3       5m34s
vagrant@master-node:~$
```

According to the definitions set in the file, above, there are three pods and a replica set. You may notice that the name of the replica set is always <name-of-deployment>-<hash value of the pod template>

A deployment - as a replicaset, can be scaled up and down

```bash
vagrant@master-node:~$ kubectl scale deploy nginx --replicas=6
deployment.apps/nginx scaled
vagrant@master-node:~$ kubectl get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           15m
vagrant@master-node:~$
```

In addition to replicas management, a deployment also defines the strategy for updates pods:

```yaml
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
```

In the snippet above, we set the update strategy as rolling update. During the lifetime of an application, some services need to be update, for example because the image changed. To update a service without an outage, Kubernetes updates one or more pod at a time, rather than taking down the entire application.

For example, to update the pods with a different version of nginx image

```bash
vagrant@master-node:~$ kubectl set image deploy nginx nginx=nginx:1.9.1
deployment.apps/nginx image updated
vagrant@master-node:~$ kubectl rollout status deploy nginx
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 3 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 4 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 5 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 5 of 6 updated replicas are available...
deployment "nginx" successfully rolled out
vagrant@master-node:~$ kubectl get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           18m
vagrant@master-node:~$ kubectl get po
NAME                   READY   STATUS    RESTARTS   AGE
nginx-956858c7-b6ncm   1/1     Running   0          60s
nginx-956858c7-bsn5x   1/1     Running   0          95s
nginx-956858c7-cgvcv   1/1     Running   0          71s
nginx-956858c7-p65vx   1/1     Running   0          61s
nginx-956858c7-pjsts   1/1     Running   0          51s
nginx-956858c7-wnwjn   1/1     Running   0          95s
vagrant@master-node:~$
```

Deployment can ensure that only a certain number of pods may be down while they are being updated. By default, it ensures that at least 25% less than the desired number of pods are up. Deployment can also ensure that only a certain number of pods may be created above the desired number of pods. By default, it ensures that at most 25% more than the desired number of pods are up.

## DaemonSets

A Daemon Set is a controller type ensuring each node in the cluster runs a pod. As new node is added to the cluster, a new pod is added to the node. As the node is removed from the cluster, the pod running on it is removed and not scheduled on another node. Deleting a Daemon Set will clean up all the pods it created.

The configuration file nginx-daemon-set.yaml defines a daemon set for the nginx application

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    run: nginx
  name: nginx-ds
  namespace:
spec:
  selector:
    matchLabels:
      run: nginx-ds
  template:
    metadata:
      labels:
        run: nginx-ds
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

If we create the daemonset and see the output of the daemonset and all the resources it created, we would see that one nginx pod runs on each worker node:

```bash
vagrant@master-node:~$ kubectl create -f nginx-daemon-set.yaml
daemonset.apps/nginx-ds created
vagrant@master-node:~$ kubectl get all -l run=nginx-ds -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
pod/nginx-ds-48p8v   1/1     Running   0          41s   172.16.87.206   worker-node01   <none>           <none>
pod/nginx-ds-vhhsj   1/1     Running   0          41s   172.16.158.13   worker-node02   <none>           <none>
vagrant@master-node:~$
```

Following things need to be understood for DaemonSets:

    - A Pod will be automatically started on each of the worker node
    - If a node is delete, the Pod is also deleted and is not scheduled on any other nodes
    - DaemonSets are not scheduled by the Kube-Scheduler and hence can run even there is no Kube-Scheduler process running

## Scheduling

### Manually scheduling Pod with nodeName attribute

Consider the following Pod definition file pod-node-name.yaml which has the nodeName attribute set such that it gets scheduled on the worker-node01

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        app: app1
        type: front-end
spec:
    containers:
    - name: nginx-controller
      image: nginx
    nodeName: worker-node01
```

Create the Pod and see the Pod getting run:

```bash
vagrant@master-node:~$ kubectl create -f pod-node-name.yaml
pod/simple-webapp created
vagrant@master-node:~$ kubectl get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP              NODE            NOMINATED NODE   READINESS GATES
nginx-956858c7-b6ncm   1/1     Running   0          40m    172.16.87.205   worker-node01   <none>           <none>
nginx-956858c7-bsn5x   1/1     Running   0          41m    172.16.87.203   worker-node01   <none>           <none>
nginx-956858c7-cgvcv   1/1     Running   0          40m    172.16.87.204   worker-node01   <none>           <none>
nginx-956858c7-p65vx   1/1     Running   0          40m    172.16.158.11   worker-node02   <none>           <none>
nginx-956858c7-pjsts   1/1     Running   0          40m    172.16.158.12   worker-node02   <none>           <none>
nginx-956858c7-wnwjn   1/1     Running   0          41m    172.16.158.10   worker-node02   <none>           <none>
nginx-ds-48p8v         1/1     Running   0          21m    172.16.87.206   worker-node01   <none>           <none>
nginx-ds-vhhsj         1/1     Running   0          21m    172.16.158.13   worker-node02   <none>           <none>
simple-webapp          1/1     Running   0          3m2s   172.16.87.207   worker-node01   <none>           <none>
vagrant@master-node:~$
```
As we can see that the Pod `simple-webapp` was able to get scheduled on worker-node01. Let's now delete the pod as we would be using pods with the same name for the next set of steps:

```bash
vagrant@master-node:~$ kubectl delete po simple-webapp
pod "simple-webapp" deleted
vagrant@master-node:~$
```
### Taints and Tolerations

Let's taint the worker-node01 with the following key/value pairs:

```bash
vagrant@master-node:~$ kubectl taint nodes worker-node01 app=blue:NoSchedule
node/worker-node01 tainted
vagrant@master-node:~$
```

Let us restart all the pods of the deployment that we created earlier:

```bash
vagrant@master-node:~$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           73m
vagrant@master-node:~$ kubectl get po -l run=nginx -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
nginx-956858c7-b6ncm   1/1     Running   0          56m   172.16.87.205   worker-node01   <none>           <none>
nginx-956858c7-bsn5x   1/1     Running   0          57m   172.16.87.203   worker-node01   <none>           <none>
nginx-956858c7-cgvcv   1/1     Running   0          56m   172.16.87.204   worker-node01   <none>           <none>
nginx-956858c7-p65vx   1/1     Running   0          56m   172.16.158.11   worker-node02   <none>           <none>
nginx-956858c7-pjsts   1/1     Running   0          56m   172.16.158.12   worker-node02   <none>           <none>
nginx-956858c7-wnwjn   1/1     Running   0          57m   172.16.158.10   worker-node02   <none>           <none>
vagrant@master-node:~$
```
As we can see the replicas are distributed between worker nodes 1 and 2. Let's restart the pods of the deployment by running the following command and wait for the restart to complete

```bash
vagrant@master-node:~$ kubectl rollout restart deploy nginx
deployment.apps/nginx restarted

vagrant@master-node:~$ kubectl rollout status deploy nginx
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 6 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 3 out of 6 new replicas have been updated...
...
deployment "nginx" successfully rolled out
vagrant@master-node:~$
```

Let's look at the Pods of this deployment and where are they scheduled:

```bash
vagrant@master-node:~$ kubectl get po -l run=nginx -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
nginx-67668f65b7-6g5jl   1/1     Running   0          97s   172.16.158.22   worker-node02   <none>           <none>
nginx-67668f65b7-78sjb   1/1     Running   0          78s   172.16.158.26   worker-node02   <none>           <none>
nginx-67668f65b7-gq6mv   1/1     Running   0          72s   172.16.158.27   worker-node02   <none>           <none>
nginx-67668f65b7-qsfgc   1/1     Running   0          90s   172.16.158.24   worker-node02   <none>           <none>
nginx-67668f65b7-sc77j   1/1     Running   0          84s   172.16.158.25   worker-node02   <none>           <none>
nginx-67668f65b7-w57wg   1/1     Running   0          97s   172.16.158.23   worker-node02   <none>           <none>
vagrant@master-node:~$
```

As we can see, all the pods are deployed on worker-node02 as worker-node01 is tainted. Let's add a toleration in the Pod Spec within the Deployment:

First Delete the Deployment:

```bash
vagrant@master-node:~$ kubectl delete deployment nginx
deployment.apps "nginx" deleted
vagrant@master-node:~$
```

No consider the file nginx-deploy-tolerations.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
```

Create the deployment and observe how the Pods are scheduled:

```bash
vagrant@master-node:~$ kubectl create -f nginx-deploy-tolerations.yaml
deployment.apps/nginx created
vagrant@master-node:~$ kubectl get po -l run=nginx -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
nginx-559fdf7f69-dxlbm   1/1     Running   0          19s   172.16.87.211   worker-node01   <none>           <none>
nginx-559fdf7f69-j5wwx   1/1     Running   0          19s   172.16.87.210   worker-node01   <none>           <none>
nginx-559fdf7f69-m826n   1/1     Running   0          19s   172.16.158.28   worker-node02   <none>           <none>
vagrant@master-node:~$
```

As you can see the Pods are again scheduled between, both worker-node01 and worker-node02.

If we want to schedule the Pods only on a particular node, we can:
    - Either use nodeName. This bypasses the kube-scheduler completely.
    - Use NodeAffinity Rules along with labelling the desired nodes with some key/value Pairs. Tainting the node further ensures that no other Pods are scheduled on this, making some nodes exclusive for some applications.
We have already seen how to use nodeName in Pod definitions. Let's now use NodeAffinity Rules.

Let's delete the deployment first:

```bash
vagrant@master-node:~$ kubectl delete deployment nginx
deployment.apps "nginx" deleted
vagrant@master-node:~$
```

First Label the node where we want to run our application:

```bash
vagrant@master-node:~$ kubectl label nodes worker-node01 only=nginx
node/worker-node01 labeled
vagrant@master-node:~$
```


Consider the file `nginx-deploy-nodeaffinity.yaml` which adds the nodeaffinity rules:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: only
                operator: In
                values:
                - nginx
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
```
Create the deployment and observer the pods:

```bash
vagrant@master-node:~$ kubectl create -f nginx-deploy-nodeaffinity.yaml
deployment.apps/nginx created
vagrant@master-node:~$ kubectl get po -l run=nginx -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
nginx-79748db8b-74kvp   1/1     Running   0          62s   172.16.87.212   worker-node01   <none>           <none>
nginx-79748db8b-c7zph   1/1     Running   0          62s   172.16.87.214   worker-node01   <none>           <none>
nginx-79748db8b-ln2rg   1/1     Running   0          62s   172.16.87.213   worker-node01   <none>           <none>
vagrant@master-node:~$
```

As we can see the pods in the deployment are only running on worker-node01.

Since the worker-node01 is also tainted, no other pods will be accepted here, till the time the pods tolerate the taints. Let's verify by creating a deployment with 3 replicas imperatively

```bash
vagrant@master-node:~$ kubectl create deployment n1 --image=nginx --replicas=3
deployment.apps/n1 created
vagrant@master-node:~$ kubectl get po -l app=n1 -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP              NODE            NOMINATED NODE   READINESS GATES
n1-cfcbcb76-8xzgf   1/1     Running   0          75s   172.16.158.31   worker-node02   <none>           <none>
n1-cfcbcb76-jq6jp   1/1     Running   0          75s   172.16.158.29   worker-node02   <none>           <none>
n1-cfcbcb76-xrnxq   1/1     Running   0          75s   172.16.158.30   worker-node02   <none>           <none>
vagrant@master-node:~$
```
As you can see, since the pods in the deployment do not have a toleration by default for the taint in worker-node01 all of them are deployed in worker-node02


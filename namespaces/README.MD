# Namespaces and Resource Controls

## Namespaces
Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. Within the same namespace, kubernetes objects name should be unique. Different objects in different namespaces may have the same name.

Kubernetes comes with a few initial namespaces. Some of the important ones are as follows:
    - **default:** the default namespace for objects with no other namespace
    - **kube-system** the namespace for objects created by the kubernetes system
To get all the namespaces, issue the following command:

```bash
vagrant@master-node:~$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   3h31m
kube-node-lease        Active   3h31m
kube-public            Active   3h31m
kube-system            Active   3h31m
kubernetes-dashboard   Active   3h23m
vagrant@master-node:~$
```

To see all the objects for a particular namespace:

```bash
vagrant@master-node:~$ kubectl get all --namespace default
NAME                        READY   STATUS    RESTARTS   AGE
pod/n1-cfcbcb76-8xzgf       1/1     Running   0          15m
pod/n1-cfcbcb76-jq6jp       1/1     Running   0          15m
pod/n1-cfcbcb76-xrnxq       1/1     Running   0          15m
pod/nginx-79748db8b-74kvp   1/1     Running   0          19m
pod/nginx-79748db8b-c7zph   1/1     Running   0          19m
pod/nginx-79748db8b-ln2rg   1/1     Running   0          19m
pod/nginx-ds-48p8v          1/1     Running   0          96m
pod/nginx-ds-vhhsj          1/1     Running   0          96m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.17.0.1   <none>        443/TCP   3h33m

NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-ds   1         1         1       1            1           <none>          96m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/n1      3/3     3            3           15m
deployment.apps/nginx   3/3     3            3           19m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/n1-cfcbcb76       3         3         3       15m
replicaset.apps/nginx-79748db8b   3         3         3       19m
vagrant@master-node:~$
```
or to view objects across all namespaces:

```bash
vagrant@master-node:~$ kubectl get deploy -A
NAMESPACE              NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
default                n1                          3/3     3            3           16m
default                nginx                       3/3     3            3           20m
kube-system            calico-kube-controllers     1/1     1            1           3h34m
kube-system            coredns                     2/2     2            2           3h34m
kube-system            metrics-server              1/1     1            1           3h34m
kubernetes-dashboard   dashboard-metrics-scraper   1/1     1            1           3h25m
kubernetes-dashboard   kubernetes-dashboard        1/1     1            1           3h25m
vagrant@master-node:~$
```

Please, note that not all kubernetes objects are in namespaces, i.e. nodes, are cluster resources not included in any namespaces.

Observe the projectone-ns.yaml configuration file

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: projectone
```
Create the namespace and see if it is created:

```bash
vagrant@master-node:~$ kubectl create -f projectone-ns.yaml
namespace/projectone created
vagrant@master-node:~$ kubectl get ns
NAME                   STATUS   AGE
default                Active   3h36m
kube-node-lease        Active   3h36m
kube-public            Active   3h36m
kube-system            Active   3h36m
kubernetes-dashboard   Active   3h28m
projectone             Active   65s
vagrant@master-node:~$
```

Observe the file nginx-pod-projectone.yaml

```yaml
```

Create the Pod and list the Pod in the projectone namespace:

```bash
vagrant@master-node:~$ kubectl create -f nginx-pod-projectone.yaml
pod/mynginx created
vagrant@master-node:~$ kubectl get po -n projectone
NAME      READY   STATUS    RESTARTS   AGE
mynginx   1/1     Running   0          10s
vagrant@master-node:~$
```
Deleting a namespace will result in deleting of all resources within the namespace:

```bash
vagrant@master-node:~$ kubectl get po -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS        AGE
default                n1-cfcbcb76-8xzgf                            1/1     Running   0               24m
default                n1-cfcbcb76-jq6jp                            1/1     Running   0               24m
default                n1-cfcbcb76-xrnxq                            1/1     Running   0               24m
default                nginx-79748db8b-74kvp                        1/1     Running   0               28m
default                nginx-79748db8b-c7zph                        1/1     Running   0               28m
default                nginx-79748db8b-ln2rg                        1/1     Running   0               28m
default                nginx-ds-48p8v                               1/1     Running   0               105m
default                nginx-ds-vhhsj                               1/1     Running   0               105m
kube-system            calico-kube-controllers-786b679988-fx999     1/1     Running   0               3h41m
kube-system            calico-node-6lkk9                            1/1     Running   0               3h41m
kube-system            calico-node-6zbzh                            1/1     Running   0               3h38m
kube-system            calico-node-wmthn                            1/1     Running   0               3h33m
kube-system            coredns-5d78c9869d-c9r7c                     1/1     Running   0               3h41m
kube-system            coredns-5d78c9869d-f8k28                     1/1     Running   0               3h41m
kube-system            etcd-master-node                             1/1     Running   0               3h42m
kube-system            kube-apiserver-master-node                   1/1     Running   0               3h41m
kube-system            kube-controller-manager-master-node          1/1     Running   0               3h42m
kube-system            kube-proxy-d65fq                             1/1     Running   0               3h33m
kube-system            kube-proxy-h5gqv                             1/1     Running   0               3h38m
kube-system            kube-proxy-kxdl4                             1/1     Running   0               3h41m
kube-system            kube-scheduler-master-node                   1/1     Running   0               3h41m
kube-system            metrics-server-754586b847-fbs8q              1/1     Running   3 (3h36m ago)   3h41m
kubernetes-dashboard   dashboard-metrics-scraper-5cb4f4bb9c-gmf7d   1/1     Running   0               3h33m
kubernetes-dashboard   kubernetes-dashboard-6967859bff-znz7b        1/1     Running   0               3h33m
projectone             mynginx                                      1/1     Running   0               2m28s
```
Deleting projectone namespace leads to deletion of the pods in that namespace as well, as shown below:
```bash
vagrant@master-node:~$ kubectl delete ns projectone
namespace "projectone" deleted
vagrant@master-node:~$ kubectl get po -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS        AGE
default                n1-cfcbcb76-8xzgf                            1/1     Running   0               25m
default                n1-cfcbcb76-jq6jp                            1/1     Running   0               25m
default                n1-cfcbcb76-xrnxq                            1/1     Running   0               25m
default                nginx-79748db8b-74kvp                        1/1     Running   0               29m
default                nginx-79748db8b-c7zph                        1/1     Running   0               29m
default                nginx-79748db8b-ln2rg                        1/1     Running   0               29m
default                nginx-ds-48p8v                               1/1     Running   0               107m
default                nginx-ds-vhhsj                               1/1     Running   0               107m
kube-system            calico-kube-controllers-786b679988-fx999     1/1     Running   0               3h43m
kube-system            calico-node-6lkk9                            1/1     Running   0               3h43m
kube-system            calico-node-6zbzh                            1/1     Running   0               3h39m
kube-system            calico-node-wmthn                            1/1     Running   0               3h34m
kube-system            coredns-5d78c9869d-c9r7c                     1/1     Running   0               3h43m
kube-system            coredns-5d78c9869d-f8k28                     1/1     Running   0               3h43m
kube-system            etcd-master-node                             1/1     Running   0               3h43m
kube-system            kube-apiserver-master-node                   1/1     Running   0               3h43m
kube-system            kube-controller-manager-master-node          1/1     Running   0               3h43m
kube-system            kube-proxy-d65fq                             1/1     Running   0               3h34m
kube-system            kube-proxy-h5gqv                             1/1     Running   0               3h39m
kube-system            kube-proxy-kxdl4                             1/1     Running   0               3h43m
kube-system            kube-scheduler-master-node                   1/1     Running   0               3h43m
kube-system            metrics-server-754586b847-fbs8q              1/1     Running   3 (3h37m ago)   3h43m
kubernetes-dashboard   dashboard-metrics-scraper-5cb4f4bb9c-gmf7d   1/1     Running   0               3h34m
kubernetes-dashboard   kubernetes-dashboard-6967859bff-znz7b        1/1     Running   0               3h34m
vagrant@master-node:~$
```

## Quotas and Limits

### Quotas

Namespaces let different users or teams to share a cluster with a fixed number of nodes. It can be a concern that one team could use more than its fair share of resources. Resource quotas are the tool to address this concern.

A resource quota provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed in that project.

Users create resources in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in the resource quota. If creating or updating a resource violates a quota constraint, the request will fail. When quota is enabled in a namespace for compute resources like cpu and memory, users must specify resources consumption, otherwise the quota system rejects pod creation.

Consider quota.yaml configuration file to assign constraints to current namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: project-quota
spec:
  hard:
    limits.memory: 1Gi
    limits.cpu: 1
    pods: 10
```
Create the quota and see the default namespace now

```bash
vagrant@master-node:~$ kubectl create -f quota.yaml
resourcequota/project-quota created
vagrant@master-node:~$ kubectl describe ns default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:          project-quota
  Resource       Used  Hard
  --------       ---   ---
  limits.cpu     0     1
  limits.memory  0     1Gi
  pods           8     10

No LimitRange resource.
vagrant@master-node:~$
```

Current namespace has now hard constraints set to 1 core CPU, 1 GB of RAM and max 10 running pods. Having set constraints for the namespace, all further requests for pod creation inside that namespace, must specify resources consumption, otherwise the quota system will reject the pod creation.

```bash
vagrant@master-node:~$ kubectl run nginx-test --image=nginx
Error from server (Forbidden): pods "nginx-test" is forbidden: failed quota: project-quota: must specify limits.cpu for: nginx-test; limits.memory for: nginx-test
vagrant@master-node:~$
```

The reason is that, by default, a pod try to allocate all the CPU and memory available in the system. Since we have limited cpu and memory consumption for the namespaces, the quota system cannot honor a request for pod creation crossing these limits.

We can specify the limits on Pods by creating resourceLimits in the Pod definition file. Consider nginx-deploy-limited.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 1
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
        resources:
          limits:
            cpu: "200m"
            memory: "512Mi"
        ports:
        - containerPort: 80
          protocol: TCP
```
Delete the nginx deployment first if it exists

```bash
vagrant@master-node:~$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
n1      3/3     3            3           37m
nginx   3/3     3            3           41m
vagrant@master-node:~$ kubectl delete deployment nginx
deployment.apps "nginx" deleted
```

Then create the deployment with limits specified:

```bash
vagrant@master-node:~$ kubectl create -f nginx-deploy-limited.yaml
deployment.apps/nginx created
vagrant@master-node:~$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
n1      3/3     3            3           49m
nginx   1/1     1            1           6s
vagrant@master-node:~$ kubectl get po -l run=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-796f587cf7-ll68z   1/1     Running   0          3m37s
vagrant@master-node:~$
```

Let's scale the replicas to 3 replicas and see what happens:

```bash
vagrant@master-node:~$ kubectl scale deploy nginx --replicas=3
deployment.apps/nginx scaled
vagrant@master-node:~$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
n1      3/3     3            3           53m
nginx   2/3     2            2           4m31s
vagrant@master-node:~$ kubectl get po -l run=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-796f587cf7-ll68z   1/1     Running   0          4m38s
nginx-796f587cf7-ndst7   1/1     Running   0          11s
vagrant@master-node:~$
```

As you can see it could scale up to 2 replicas only as the limits on memory was exhausted:

```bash
vagrant@master-node:~$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
n1-cfcbcb76        3         3         3       55m
nginx-796f587cf7   3         2         2       5m50s
vagrant@master-node:~$ kubectl describe rs nginx-796f587cf7
Name:           nginx-796f587cf7
Namespace:      default
Selector:       pod-template-hash=796f587cf7,run=nginx
Labels:         pod-template-hash=796f587cf7
                run=nginx
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 4
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/nginx
Replicas:       2 current / 3 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=796f587cf7
           run=nginx
  Containers:
   nginx:
    Image:      nginx:latest
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:        200m
      memory:     512Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  ReplicaFailure   True    FailedCreate
Events:
  Type     Reason            Age                From                   Message
  ----     ------            ----               ----                   -------
  Normal   SuccessfulCreate  6m2s               replicaset-controller  Created pod: nginx-796f587cf7-ll68z
  Normal   SuccessfulCreate  95s                replicaset-controller  Created pod: nginx-796f587cf7-ndst7
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-46bf4" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-c6ckt" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-blxlc" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-ch5d5" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-kxnfk" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-j4dmb" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-vwkr2" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-vtmmc" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      95s                replicaset-controller  Error creating: pods "nginx-796f587cf7-829gn" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
  Warning  FailedCreate      29s (x6 over 93s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-796f587cf7-4wf2q" is forbidden: exceeded quota: project-quota, requested: limits.memory=512Mi, used: limits.memory=1Gi, limited: limits.memory=1Gi
vagrant@master-node:~$
```
### Limit Ranges

Quotas lets the cluster administrators to control the resource consumption within a shared cluster. However, a single namespace may be used by more than a single user and it may deploy more than an application. To avoid a single pod consumes all resource of a given namespace, kubernetes introduces the limit range object. The limit range object limits the resources that a pod can consume by specifying the minimum, maximum and default resource consumption.

Consider limitranges.yaml defines limits for all containers running in the current namespace

```yaml
---
kind: LimitRange
apiVersion: v1
metadata:
  name: container-limit-ranges
spec:
  limits:
  - type: Container
    max:
      cpu: 200m
      memory: 512Mi
    min:
      cpu:
      memory:
    default:
      cpu: 100m
      memory: 256Mi
```

Create the LimitRanges object:

```bash
vagrant@master-node:~$ kubectl create -f limitranges.yaml
limitrange/container-limit-ranges created
vagrant@master-node:~$ kubectl describe ns default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:          project-quota
  Resource       Used  Hard
  --------       ---   ---
  limits.cpu     400m  1
  limits.memory  1Gi   1Gi
  pods           7     10

Resource Limits
 Type       Resource  Min  Max    Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---  ---    ---------------  -------------  -----------------------
 Container  memory    0    512Mi  256Mi            256Mi          -
 Container  cpu       0    200m   100m             100m           -
vagrant@master-node:~$
```

Scale the replicas of the nginx deployment to zero

```bash
vagrant@master-node:~$ kubectl scale deploy nginx --replicas=0
deployment.apps/nginx scaled
vagrant@master-node:~$ kubectl get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/0     0            0           15m
vagrant@master-node:~$
```

Edit the deployment by issuing the following command:

```bash
vagrant@master-node:~$ kubectl edit deploy nginx
```

In the file that gets opened, scroll down to the `limits` section and make the cpu as `250m`. Save and exit the file. After exiting the edit mode you should get the following output

```bash
vagrant@master-node:~$ kubectl edit deploy nginx

deployment.apps/nginx edited
```

Check the Deployment again:

```bash
vagrant@master-node:~$ kubectl get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/0     0            0           18m
vagrant@master-node:~$
```

Scale the deployment to one replica

```bash
vagrant@master-node:~$ kubectl get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     0            0           19m
vagrant@master-node:~$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
n1-cfcbcb76        3         3         3       68m
nginx-5974479d99   1         0         0       90s
nginx-796f587cf7   0         0         0       19m
```
As you can see the pods are not getting created. Since we have edited the deployment we see two Replicasets for the deployment. Describe the Replicaset which has the DESIRED as 1

```bash
vagrant@master-node:~$ kubectl describe rs nginx-5974479d99
Name:           nginx-5974479d99
Namespace:      default
Selector:       pod-template-hash=5974479d99,run=nginx
Labels:         pod-template-hash=5974479d99
                run=nginx
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 2
Controlled By:  Deployment/nginx
Replicas:       0 current / 1 desired
Pods Status:    0 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=5974479d99
           run=nginx
  Containers:
   nginx:
    Image:      nginx:latest
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:        250m
      memory:     512Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  ReplicaFailure   True    FailedCreate
Events:
  Type     Reason        Age               From                   Message
  ----     ------        ----              ----                   -------
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-mkd5s" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-snp7z" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-clwcf" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-b8bmn" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-jdhwj" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-gjc57" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  42s               replicaset-controller  Error creating: pods "nginx-5974479d99-fw99k" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  41s               replicaset-controller  Error creating: pods "nginx-5974479d99-l2sn9" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  41s               replicaset-controller  Error creating: pods "nginx-5974479d99-bnwnc" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
  Warning  FailedCreate  1s (x5 over 39s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-5974479d99-wpskr" is forbidden: maximum cpu usage per Container is 200m, but limit is 250m
vagrant@master-node:~$
```

As we saw, we hit the LimitRange for this namespace. Just to recap, quota defines the total amount of resources within a namespace, while limit ranges define the resource usage for a single pod within the same namespace.
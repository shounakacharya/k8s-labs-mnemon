# Storage using PV and PVCs

In this exercise, you create a hostPath PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

In a production cluster, you would not use hostPath. Instead a cluster administrator would provision a network resource like a Google Compute Engine persistent disk, an NFS share, or an Amazon Elastic Block Store volume. Cluster administrators can also use StorageClasses to set up dynamic provisioning.


1. Open a shell in your host machine and navigate to the root directory of this repository from where you ssh to the master node previously. You should be in the `k8s-labs-mnemon` directory. From this directory issue the following commands to get the names of the booted nodes

```bash
PS C:\Users\shoun\OneDrive\Documents\kubernetes\k8s-labs-mnemon> vagrant status
Current machine states:

master                    running (virtualbox)
node01                    running (virtualbox)
node02                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
PS C:\Users\shoun\OneDrive\Documents\kubernetes\k8s-labs-mnemon>
```

As we can see we have a 3 node cluster. 1 master and 2 worker nodes

2. Repeat the above process on one more shell

3. Now from the first shell ssh to the master node:

```bash
PS C:\Users\shoun\OneDrive\Documents\kubernetes\k8s-labs-mnemon> vagrant ssh master
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-67-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Oct 12 03:49:22 PM UTC 2023

  System load:  1.01416015625      Users logged in:        0
  Usage of /:   19.5% of 30.34GB   IPv4 address for eth0:  10.0.2.15
  Memory usage: 23%                IPv4 address for eth1:  192.168.56.20
  Swap usage:   0%                 IPv4 address for tunl0: 172.16.77.128
  Processes:    182

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Thu Oct 12 12:42:29 2023 from 10.0.2.2
vagrant@master-node:~$
```

4. From the second shell ssh to node01

```bash
PS C:\Users\shoun\OneDrive\Documents\kubernetes\k8s-labs-mnemon> vagrant ssh node01
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-67-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Oct 12 03:51:02 PM UTC 2023

  System load:  0.095703125        Users logged in:        0
  Usage of /:   18.3% of 30.34GB   IPv4 address for eth0:  10.0.2.15
  Memory usage: 19%                IPv4 address for eth1:  192.168.56.21
  Swap usage:   0%                 IPv4 address for tunl0: 172.16.87.192
  Processes:    154

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
vagrant@worker-node01:~$
```

5. On the worker node issue the following commands to create a directory called /mnt/data

```bash
vagrant@worker-node01:~$ sudo mkdir /mnt/data
```

Then create a file called as `index.html` in the same directory with the contents as `Hello from Kubernetes storage`.

```bash
vagrant@worker-node01:~$ sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```

Check the file has been properly created or not and its contents properly populated.

```bash
vagrant@worker-node01:~$ cat /mnt/data/index.html
Hello from Kubernetes storage
vagrant@worker-node01:~$
```

6. Now Go back to the shell where you have logged in to the master node:

Navigate to `/vagrant/storage` directory and consider the pv-volume.yaml file

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

This creates a Persistent volume by running the following command and verify it exists:

```bash
vagrant@master-node:/vagrant/storage$ kubectl create -f pv-volume.yaml
persistentvolume/task-pv-volume created
vagrant@master-node:/vagrant/storage$ kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Available           manual                  5s
vagrant@master-node:/vagrant/storage$
```

7. Consider the file pv-claim.yaml to create a Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Create and verify the pvc. The PVC should be in a bound state after this

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

8. Create a Pod that uses this PVC. It would be an nginx container which uses this index.html as its index page. Consider the yaml file pv-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
  labels:
    type: storage
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  nodeName: worker-node01
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

Of special interest here is the nodeName that we have specified so that it gets scheduled on the worker-node01 where we had created the hostpath. Also note the volumes and volumeMounts sections which reference the PVC created earlier and then uses this PVC to mount a volume into the Pod's `/usr/share/nginx/html` directory.

Now create the Pod and ensure it is running

```bash
vagrant@master-node:/vagrant/storage$ kubectl create -f pv-pod.yaml
pod/task-pv-pod created
vagrant@master-node:/vagrant/storage$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          5s
vagrant@master-node:/vagrant/storage$
```

9. Now create a service for this Pod by issuing the following command and note down the cluster IP:

```bash
vagrant@master-node:/vagrant/storage$ kubectl expose pod task-pv-pod --port=8080 --target-port=80
service/task-pv-pod exposed
vagrant@master-node:/vagrant/storage$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes    ClusterIP   172.17.0.1      <none>        443/TCP    34m
task-pv-pod   ClusterIP   172.17.17.158   <none>        8080/TCP   5s
vagrant@master-node:/vagrant/storage$
```

10. Create a client Pod with the busybox image by running the following command and ensure that it is running:

```bash
vagrant@master-node:/vagrant/storage$ kubectl run busybox --image=busybox -- sleep 3600
pod/busybox created
vagrant@master-node:/vagrant/storage$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
busybox       1/1     Running   0          9s
task-pv-pod   1/1     Running   0          50s
^Cvagrant@master-node:/vagrant/storage$
```

11. Access the service by hitting the service IP from within the busybox Pod. In this case the IP was `172.17.17.158` as shown in Step 9

```bash
vagrant@master-node:/vagrant/storage$ kubectl exec -it busybox -- sh
/ # wget -O - 172.17.17.158:8080
Connecting to 172.17.17.158:8080 (172.17.17.158:8080)
writing to stdout
Hello from Kubernetes storage
-                    100% |*************************************************************************************|    30  0:00:00 ETA
written to stdout
/ # exit
vagrant@master-node:/vagrant/storage$
```

## Clean Up

Issue the following commands to clean up the Pods, PVC and PV

```bash
vagrant@master-node:/vagrant/storage$ kubectl delete po busybox task-pv-pod
pod "busybox" deleted
pod "task-pv-pod" deleted
vagrant@master-node:/vagrant/storage$ kubectl delete pvc task-pv-claim
persistentvolumeclaim "task-pv-claim" deleted
vagrant@master-node:/vagrant/storage$ kubectl delete pv task-pv-volume
persistentvolume "task-pv-volume" deleted
vagrant@master-node:/vagrant/autoscaling$ kubectl delete svc task-pv-pod
service "task-pv-pod" deleted
vagrant@master-node:/vagrant/autoscaling$
```


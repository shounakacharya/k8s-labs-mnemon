# Auto-Scaling using Horizontal Pod Autoscaler

A HorizontalPodAutoscaler (HPA for short) automatically updates a workload resource (such as a Deployment or StatefulSet), with the aim of automatically scaling the workload to match demand.

Horizontal scaling means that the response to increased load is to deploy more Pods. This is different from vertical scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the Pods that are already running for the workload.

If the load decreases, and the number of Pods is above the configured minimum, the HorizontalPodAutoscaler instructs the workload resource (the Deployment, StatefulSet, or other similar resource) to scale back down.

This lab walks you through an example of enabling HorizontalPodAutoscaler to automatically manage scale for an example web app. This example workload is Apache httpd running some PHP code.

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

3. From the second shell too ssh to the master node:

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

4. From the first shell navigate to the directory `/vagrant/autoscaling` and look at the php-apache.yaml file. The contents of the file are shown below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache

```

Now create the deployment and service mentioned in this file by issuing the following commands:

```bash
vagrant@master-node:/vagrant/autoscaling$ kubectl create -f php-apache.yaml
deployment.apps/php-apache created
service/php-apache created
vagrant@master-node:/vagrant/autoscaling$
```

Check for the Deployment status and Pod running state as well as the service creation:

```bash
vagrant@master-node:/vagrant/autoscaling$ kubectl create -f php-apache.yaml
deployment.apps/php-apache created
service/php-apache created
vagrant@master-node:/vagrant/autoscaling$ kubectl get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   1/1     1            1           3m48s
vagrant@master-node:/vagrant/autoscaling$ kubectl get po
NAME                          READY   STATUS    RESTARTS   AGE
php-apache-6ccb568796-g2fwt   1/1     Running   0          2m29s
vagrant@master-node:/vagrant/autoscaling$
```

5. Create the HorizontalPodAutoscaler Resource:

Now that the server is running, create the autoscaler using kubectl. There is kubectl autoscale subcommand, part of kubectl, that helps you do this.

```bash
vagrant@master-node:/vagrant/autoscaling$ kubectl autoscale deployment php-apache --cpu-percent=5 --min=1 --max=10
horizontalpodautoscaler.autoscaling/php-apache autoscaled
vagrant@master-node:/vagrant/autoscaling$
```
This creates a HorizontalPodAutoscaler that maintains between 1 and 10 replicas of the Pods controlled by the php-apache Deployment that you created in the first step of these instructions.

Roughly speaking, the HPA controller will increase and decrease the number of replicas (by updating the Deployment) to maintain an average CPU utilization across all Pods of 5%. The Deployment then updates the ReplicaSet - this is part of how all Deployments work in Kubernetes - and then the ReplicaSet either adds or removes Pods based on the change to its `.spec`.

Since each pod requests 200 milli-cores by kubectl run, this means an average CPU usage of 40 milli-cores. See [Algorithm details](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details) for more details on the algorithm.

6. You can check the current status of the newly-made HorizontalPodAutoscaler, by running:

```bash
vagrant@master-node:~$ kubectl get hpa php-apache
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/5%     1         10        1          23m
vagrant@master-node:~$
```
Please note that the current CPU consumption is 0% as there are no clients sending requests to the server (the TARGET column shows the average across all the Pods controlled by the corresponding deployment).

7. Increase the load on the server:

We will run a busybox pod which will keep on hitting our deployment

```bash
vagrant@master-node:/vagrant/autoscaling$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

this will conitnuously keep generating output with values as `welcome to nginx` with some HTML tags.

8. Now on the second shell, issue the command on hpa. We should see the HPA values getting changed slowly

```bash
vagrant@master-node:~$ kubectl get hpa php-apache --watch
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/5%     1         10        1          24m
php-apache   Deployment/php-apache   9%/5%     1         10        1          24m
php-apache   Deployment/php-apache   15%/5%    1         10        2          25m
php-apache   Deployment/php-apache   8%/5%     1         10        3          25m
php-apache   Deployment/php-apache   5%/5%     1         10        3          25m
```

Let it run for 2-3 minutes.

9. Now again on the second shell exit the `kubectl get hpa` command by typing CTRL+C and issue the following command to look at the number of pods in the deployment

```bash
vagrant@master-node:~$ kubectl get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   3/3     3            3           31m
vagrant@master-node:~$
```
As we can see it has increased the replicas to 3

10. Stop the load by typing CTRL+C on the first shell

11. Now again on the second shell issue the following command on hpa to see the replicas going down when we stop the load:

```bash
vagrant@master-node:~$ kubectl get hpa php-apache --watch
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   5%/5%     1         10        3          27m
php-apache   Deployment/php-apache   4%/5%     1         10        3          27m
```
wait till the load reduces to 0%. It should then again take around 2-5 minutes to scale back the replicas to 1

```bash
vagrant@master-node:~$ kubectl get hpa php-apache --watch
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   5%/5%     1         10        3          27m
php-apache   Deployment/php-apache   4%/5%     1         10        3          27m
php-apache   Deployment/php-apache   2%/5%     1         10        3          28m
php-apache   Deployment/php-apache   0%/5%     1         10        3          28m
php-apache   Deployment/php-apache   0%/5%     1         10        3          32m
php-apache   Deployment/php-apache   0%/5%     1         10        2          33m
php-apache   Deployment/php-apache   0%/5%     1         10        1          33m
```

## Clean Up

Issue the following commands to delete the things we created on the first shell:

```bash
vagrant@master-node:/vagrant/autoscaling$ kubectl delete -f php-apache.yaml
deployment.apps "php-apache" deleted
service "php-apache" deleted
vagrant@master-node:/vagrant/autoscaling$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/5%     1         10        1          36m
vagrant@master-node:/vagrant/autoscaling$ kubectl delete hpa php-apache
horizontalpodautoscaler.autoscaling "php-apache" deleted
vagrant@master-node:/vagrant/autoscaling$
```



K8s Logs
---------

### Refer here for details:
[RefHere](https://sematext.com/blog/tail-kubernetes-logs/)

### Tail Cluster Logs
* When we say cluster logs, we’re mostly referring to Kubernetes API server logs, but you also have kube scheduler logs and kube controller logs. All of these tell you everything you need to know about the actions performed by the kube API server. The location of the log files depends on where your Kubernetes cluster is, and you may not always have access to them—in most managed Kubernetes offerings from cloud vendors, like EKS, GKE, and AKS, you won’t. But generally, you’ll find them in the following directories:

* API server logs: `/var/log/kube-apiserver.log`
* Controller logs: `/var/log/kube-controller-manager.log`
* Scheduler Logs: `/var/log/kube-scheduler.log`

* If you do have access to these logs, you can tail them using the Linux tail command, which will start following the kube-apiserver logs:

`tail -f /var/log/kube-apiserver.log`

### Events
* When running a cluster, you have to understand the events the Kubernetes cluster is producing. This helps you a lot to debug any issue in production. Events are the objects that define what’s happening in the cluster, node, or pod or with Kubernetes constructs. Kubernetes events for each resource tell what changes have happened in Kubernetes.

* To see all the events your cluster is generating at a given moment, use:

`kubectl get events`


### Default Logs

`kubectl logs podname -n namespace`

* The above kubectl command shows you the logs of the pod in the specified namespace. Note that the namespace is important, as otherwise, you may not be able to find the podname. Since you haven’t defined the container logs you want to watch, if there is only one container running, you’ll see the log of that container. Otherwise, the command will present you with container names and ask you to specify which one you want.

#### Specific Container Logs

`kubectl logs podname -n namespace -c container_name`

* This kubectl command retrieves the logs of a specific container. The “-c” option lets you specify the container name, which is very helpful when you’re running more than one container in the pod.

### All Containers

`Kubectl logs podname -n namespace –all-containers=true`

* This lets you see the logs of all the containers present in the pod, which is useful in the case of container restarts. Just search through all the container logs at once to figure out which containers are running out of memory.

### Tail Logs from Pods
* Till now, I’ve talked about the logs of containers in the pod. Now, we’ll look at how you can view logs of multiple containers.

* Labels and selectors are a major part of Kubernetes architecture and logging these labels allows you to select the pods whose logs you want to tail.

* For example:

`kubectl logs -l name=myLabel`

* With this kubectl command, you can view all the logs for pods with the label “name=myLabel.”

### Limit the Number of Lines
* Most of the time, logs are huge, and nobody wants to look at more than a few lines, which is why you’ll want to look at a limited number of lines. The following command shows the most recent 100 lines of logs from a given pod:

`kubectl logs podname -n namespace - -tail=100`

* See the Live Logs or Follow
* Looking at the continuous logs is crucial, as when you’re debugging, you need to see the live logs. The kubectl logs command below will start following the log stream:

`kubectl logs podname -n namespace -f`

* Select Pods with Labels
* Use the option below to select multiple pods at once based on the label specified—very helpful if you’re running a microservice application that has, say, 20 pods. This way, you get all the logs of all these pods at the same time.

`kubectl logs -l app_name=nginx`


### Timestamp Option-Based Log

* You can use a kubectl logs command to print the timestamp of each log line to know exactly at what time the log was printed:

`kubectl logs —-timestamps=true -l app_name=nginx`

* The –timestamp option will show you logs of all the pods of an NGINX application with timestamps in each log line.

### Multiple Pods

* Labels and selectors let you look at multiple pods at once. We’ve already seen an example of this in the above sections. With the –selector option in the kubectl logs command, you can see the logs from multiple application pods with specified labels:

`kubectl logs —-selector label_key=label_value`
* This will show logs of all the containers matching the label “app_name=nginx.”


### All Container Logs
* You can use the container command options discussed with the -l option to get the logs of all the containers that are running in all the pods for a given label:

`kubectl logs -l app_name=nginx –all-containers=true`

* Previous Container Logs
* Most of the time when there’s a problem, your container may restart due to a failed health check or OOM. This is when having logs of the previous containers helps a lot. Kubectl lets you do this:

`kubectl logs podname -n namespace -p`

* It will show logs of the last container instance of the pod. Note, you have to specify the name of the container in case you have multiple containers in the pod.

### View Logs Since

* Consider a scenario where you know that the issue happened 30 minutes ago—so you need to view the logs from 30 minutes ago. In this case, since and since-time options are really helpful:

`kubectl logs podname -n namespace –since=1h`

* This kubectl logs command will show all the logs from the last hour:

`kubectl logs pod_name –since-time=2022-04-30T10:00:00Z`

* This shows the logs from the time specified. So if you want to take a look at the logs from an exact timestamp, you can use this option.

### Pod Events

* Pod events are Kubernetes events produced only for that pod. Use the following describe command to see these:

`kubectl describe pod podname`

* At the last part of the output, you’ll see events that are specific to this pod, which helps in debugging if there is any issue with the pod.

Tail Logs from Nodes
----------------------


* Nodes are either virtual or physical machines where you deploy your containerized workloads. Nodes contain the services and resources necessary to run the pods. So it’s very important to understand what logs to look at if we’re debugging at node level.

### Logs

* Worker nodes have two major logs to look at: kubelet logs and kube proxy logs. After these, you should also include CNI pod logs, CSI logs, and kube proxy logs. In addition, the Kubernetes events per node are vital, as these show what’s happening at the node level.

* Find kubelet logs at:

`/var/log/kubelet.log`

* Use the Linux tail utility to look at them. Meanwhile, kube-proxy logs can be found at:

`/var/log/kube-proxy.log`

* There’s also the kubectl logs command to look at the logs of the kube-proxy pod.

* To view the CSI driver or CNI pod logs, you have to use the kubectl command if your CSI and CNI drivers are running as pods. Otherwise, if they’re running as a service on Kubernetes worker nodes, use the respective documentation of the CNI or CSI driver to find them.

* The following command lets you view the CSI of CNI pod logs:

`kubectl logs CNI_POD_NAME -n kube-system`

`kubectl logs CSI_POD_NAME -n kube-system`

* You can see the namespace used here is “kube-system.” This is because all these components run in the kube-system namespace. You can also find kube-proxy pods in this namespace.

### Events

* Whenever you describe any resource, you’ll see the events produced by Kubernetes related to that particular event. So, to describe a particular node and see the events related to that node use:

`kubectl describe node node_name`

### See Deployment Logs

* There can be multiple components you want to look at to debug deployments. First, you have to look at all the pod logs in the deployment, which we discussed before; then, apart from these, there are events of the different components of Kubernetes, which are involved during any deployment.

* These components include the deployment controller, replicaset controller, and the corresponding pod. To look at these, use the describe command:

```bash
kubectl describe deployment deployment_name
kubectl describe replicaset replicaset_name
kubectl describe pod
```

* The output will show you the status and events that the deployment produced in the last section. When you make any change in deployment, you’ll see there’s a new replicaset created. It increases the pods one by one while the old replicaset decreases the pods one by one. This is how the deployment is usually rolled out.

* If you want to look at all the logs for a specific deployment, use the command:

`kubectl logs deployment/deployment_name`


* When debugging, you typically have to look at different parts and components. So it’s always a great idea to have these logs in one place; this gives me more visibility and lets you easily set up dashboards and alerts. This is where Sematext’s log management solution, Sematext Logs, can help.
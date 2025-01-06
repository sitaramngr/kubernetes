# kubernetes
Checking Cluster Status 
$ kubectl version

simple diagnostic for the cluster
$ kubectl get componentstatuses

controller-manager is responsible for running various controllers that regulate behavior in the cluster
scheduler is responsible for placing different Pods onto different nodes in the cluster.
etcd server is the storage for the cluster where all of the API objects are stored.

list out all of the nodes
$ kubectl get nodes

to get more information about a specific node
$ kubectl describe nodes <<nodeName>>

kube-proxy - The Kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster.To do its job, the proxy must be present on every node in the cluster.
$ kubectl get daemonSets --namespace=kube-system kube-proxy

core-dns - provides naming and discovery for the services that are defined in the cluster
$ kubectl get deployments --namespace=kube-system core-dns
$ kubectl get services --namespace=kube-system core-dns
the cluster IP for the DNS will be polulated to /etc/resolv.conf file for the container.

Namespaces - Kubernetes uses namespaces to organize objects in the cluster.kubectl --namespace=mystuff references objects in the mystuff namespace.
$ kubectl config set-context my-context --namespace=mystuff
$ kubectl config use-context my-context

view multiple objects of different types by using a comma separated list of types
$ kubectl get pods,services
$ kubectl describe <resource-name> <obj-name>

command will extract and print the IP address of the specified Pod
$ kubectl get pods my-pod -o jsonpath --template={.status.podIP}

to create or modify an object
$ kubectl apply -f obj.yaml

If you want to see what the apply command will do without actually making the changes, you can use the --dry-run flag to print the objects to the terminal without actually sending them to the server.

The apply command also records the history of previous configurations in an annotation within the object. You can manipulate these records with the edit-last-applied, set-last-applied, and view-last-applied commands.
$ kubectl apply -f myobj.yaml view-last-applied

To delete an object 
$ kubectl delete <resource-name> <obj-name>

to see the logs for a running container
$ kubectl logs <pod-name>
If you instead want to continuously stream the logs back to the terminal without exiting, you can add the -f

use the exec command to execute a command in a running container
$ kubectl exec -it <pod-name> -- bash

If you don’t have bash or some other terminal available within your container, you can always attach to the running process:
$ kubectl attach -it <pod-name>

Copy files to and from a container using the cp command:
$ kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>

opens up a connection that forwards traffic from the local machine on port 8080 to the remote container on port 80.
$ kubectl port-forward <pod-name> 8080:80

to view Kubernetes events,
$ kubectl get events --watch

use the top command to see the list of resources in use by either nodes or Pods.
$ kubectl top nodes
$ kubectl top pods


When you cordon a node, you prevent future Pods from being scheduled onto that machine. When you drain a node, you remove any Pods that are currently running on that machine. A good example use case for these commands would be removing a physical machine for repairs or upgrades. In that scenario, you can use kubectl cordon followed by kubectl drain to safely remove the machine from the cluster. Once the machine is repaired, you can use kubectl uncordon to re-enable Pods scheduling onto the node. There is no undrain command; Pods will naturally get scheduled onto the empty node as they are created.






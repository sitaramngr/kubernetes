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


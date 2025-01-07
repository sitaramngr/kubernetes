Labels are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They can be arbitrary and are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects.

Key	Value
acme.com/app-version  1.0.0
appVersion  1.0.0
app.version  1.0.0
kubernetes.io/cluster-service  true

$ kubectl get deployments --show-labels
$ kubectl label deployments alpaca-test "canary=true"
$ kubectl get deployments -L canary
remove a label by applying a dash-suffix:
$ kubectl label deployments alpaca-test "canary-"
$ kubectl get pods --selector="ver=2"
$ kubectl get pods --selector="app in (alpaca,bandicoot)"



Annotations, on the other hand, provide a storage mechanism that resembles labels: key/value pairs designed to hold nonidentifying information that tools and libraries can leverage. Unlike labels, annotations are not meant for querying, filtering, or otherwise differentiating Pods from each other.
Annotations provide a place to store additional metadata for Kubernetes objects where the sole purpose of the metadata is assisting tools and libraries. 
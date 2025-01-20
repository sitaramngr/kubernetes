 Service accounts are created and managed by Kubernetes itself and are generally associated with components running inside the cluster. User accounts are all other accounts associated with actual users of the cluster, and often include automation like continuous delivery services that run outside the cluster.

Kubernetes supports a number of authentication providers, including:
HTTP Basic Authentication (largely deprecated)
x509 client certificates
Static token files on the host
Cloud authentication providers, such as Azure Active Directory and AWS Identity and Access Management (IAM)
Authentication webhooks

Two pairs of related resources represent roles and role bindings. One pair is scoped to a namespace (Role and RoleBinding), while the other pair is scoped to the cluster (ClusterRole and ClusterRoleBinding).

Role resources are namespaced and represent capabilities within that single namespace. 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-and-services
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]

Sometimes you need to create a role that applies to the entire cluster, or you want to limit access to cluster-level resources. To achieve this, you use the ClusterRole and ClusterRoleBinding resources. 

Using built-in roles
$ kubectl get clusterroles
The cluster-admin role provides complete access to the entire cluster.
The admin role provides complete access to a complete namespace.
The edit role allows an end user to modify resources in a namespace.
The view role allows for read-only access to a namespace.

kubectl get clusterrolebindings

By default, the Kubernetes API server installs a cluster role that allows system:unauthenticated users access to the API server’s API discovery endpoint. For any cluster exposed to a hostile environment (e.g., the public internet) this is a bad idea, and there has been at least one serious security vulnerability via this exposure. If you are running a Kubernetes service on the public internet or an other hostile environment, you should ensure that the --anonymous-auth=false flag is set on your API server.


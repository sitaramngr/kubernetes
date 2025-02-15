defense in depth and principle of least privilege. Defense in depth is a concept where you use multiple layers of security controls across your computing systems that include Kubernetes. The principle of least privilege means giving your workloads access only to resources that are required for them to operate.

At the core of securing Pods is SecurityContext, which is an aggregation of all security-focused fields that may be applied at both the Pod and container specification level.
 1 User permissions and access control (e.g., setting User ID and Group ID)
 2 Read-only root filesystem
 3 Allow privilege escalation
 4 Seccomp, AppArmor, and SELinux profile and label assignments
 5 Run as privileged or unprivileged

apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          privileged: false
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP


runAsNonRoot - The Pod or container must run as a nonroot user. 
runAsUser/runAsGroup
This setting overrides the user and group that the container process is run as. Container images may have this configured as part of the Dockerfile.

fsgroup
Configures Kubernetes to change the group of all files in a volume when they are mounted into a Pod. An additional field, fsGroupChangePolicy, may be used to configure the exact behavior.

allowPrivilegeEscalation
Configures whether a process in a container can gain more privileges than its parent. 

privileged
Runs the container as privileged, which elevates the container to the same permissions as the host.

readOnlyRootFilesystem
Mounts the container root filesystem to read-only. This is a common attack vector and is best practice to enable. Any data or logs that the workloads need write access to can be mounted via a volume

Capabilities
Allow either the addition or removal of groups of privilege that may be required for a workload to operate. For example, your workload may configure the host’s network configuration. Rather than configuring the Pod to be privileged, which is effectively host root access, you could add the specific capability to configure the host networking configuration (NET_ADMIN is the specific capability name). This follows the principal of least privilege.

AppArmor
Controls which files processes can access. AppArmor profiles can be applied to containers via the addition of an annotation of container.apparmor.security.beta.kubernetes.io/<container_name>: <profile_ref> to the Pod specification. Acceptable values for <profile ref> include runtime/default, localhost/<path to profile>, and unconfined. The default is unconfined, which explicitly sets no profile to be applied

Seccomp
Seccomp (secure computing) profiles allow the creation of syscall filters. These filters allow specific syscalls to be allowed or blocked, which limits the surface area of the Linux kernel that is exposed to the processes in the Pods.

SELinux
Defines access controls for files and processes. SELinux operators use labels that are grouped together to create a security context (not to be mistaken with a Kubernetes SecurityContext), which is used to limit access to a process. By default, Kubernetes allocates a random SELinux context for each container; however, you may choose to set one via SecurityContext.

Both AppArmor and seccomp have the ability to set the runtime default profile to be used. Each container runtime ships with default AppArmor and seccomp profiles that have been carefully curated to reduce the attack surface area by removing syscalls and file access that are known to be attack vectors or aren’t commonly used by applications. 

apiVersion: v1
kind: Pod
metadata:
  name: amicontained
  annotations:
    container.apparmor.security.beta.kubernetes.io/amicontained: "runtime/default"
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - image: r.j3ss.co/amicontained:v0.4.9
      name: amicontained
      command: [ "/bin/sh", "-c", "--" ]
      args: [ "amicontained" ]
      securityContext:
        capabilities:
            add: ["SYS_TIME"]
            drop: ["NET_BIND_SERVICE"]
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        privileged: false

There are several tools out there that create a way to generate a seccomp profile from a running Pod, which can then be applied using SecurityContext. One such project is the Security Profiles Operator, which makes it easy to generate and manage Seccomp profiles.


Pod Security
Pod Security allows you to declare different security profiles for Pods. These security profiles are known as Pod Security Standards and are applied at the namespace level.The three Pod Security Standards are as follows:
Baseline
Most common privilege escalation while enabling easier onboarding.
Restricted
Highly restricted, covering security best practices. May cause workloads to break.
Privileged
Open and unrestricted.


There are three modes a policy may be applied to. They are as follows:
Enforce
Any Pods that violate the policy will be denied.
Warn
Any Pods that violate the policy will be allowed, and a warning message will be displayed to the user.
Audit
Any Pods that violate the policy will generate an audit message in the audit log.

Pod Security Standards are applied to a namespace using labels as follows:
Required: pod-security.kubernetes.io/<MODE>: <LEVEL>
Optional: pod-security.kubernetes.io/<MODE>-version: <VERSION> (defaults to latest)

apiVersion: v1
kind: Namespace
metadata:
  name: baseline-ns
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: v1.22
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: v1.22
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.22

Pod Security has made it easy to see which existing workloads violate a Pod Security Standard with a single dry-run command:
$ kubectl label --dry-run=server --overwrite ns \
  --all pod-security.kubernetes.io/enforce=baseline

Service Account Management
Service accounts are Kubernetes resources that provide an identity to workloads that run inside Pods. RBAC can be applied to service accounts to control what resources, via the Kubernetes API, the identity has access to.

By default, Kubernetes creates a default service account in each namespace, which is automatically set as the service account for all Pods. This service account contains a token that is automounted in each Pod and is used to access the Kubernetes API.To disable this behavior, you must add automountServiceAccountToken: false to the service account configuration.This must be done in each namespace.
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
automountServiceAccountToken: false

RuntimeClass
Kubernetes interacts with the container runtime on the node’s operating system via the Container Runtime Interface (CRI). Different RuntimeClasses must be configured by a cluster administrator and may required specific nodeSelectors or tolerations on your workload to be scheduled to the correct node.

apiVersion: v1
kind: Pod
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  runtimeClassName: firecracker
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard

Network Policy
Kubernetes also has a Network Policy API that allows you to create both ingress and egress network policies for your workload. Network policies are configured using labels that allow you to select specific Pods and define how they can communicate with other Pods and endpoints. Network Policy resources are implemented by network plug-ins, such as Calico, Cilium, and Weave Net.

networkpolicy-default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

networkpolicy-kuard-allow-test-source.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-kuard
spec:
  podSelector:
    matchLabels:
      app: kuard
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: test-source


Security Benchmark Tools
There are several open source tools that allow you to run a suite of security benchmarks against your Kubernetes cluster to determine if your configuration meets a predefined set of security baselines. Once such tool is called kube-bench. kube-bench can be used to run the CIS Benchmarks for Kubernetes. Tools like kube-bench running the CIS Benchmarks aren’t specifically focused on Pod security; however, they can certainly expose any cluster misconfigurations and help identify remediations
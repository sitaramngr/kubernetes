create a Pod is via the imperative kubectl run command.
$ kubectl run kuard --generator=run-pod/v1 \
  --image=gcr.io/kuar-demo/kuard-amd64:blue

see the status of this Pod
$ kubectl get pods

The kubectl logs command downloads the current logs from the running instance\pod:
$ kubectl logs podname
Adding the -f flag will cause the logs to stream continuously.Adding the --previous flag will get logs from a previous instance of the container.

Running Commands in Your Container with exec
$ kubectl exec kuard -- date
$ kubectl exec -it kuard -- ash

Liveness Probe- iveness health checks are application-specific, you have to define them in your Pod manifest.the HTTP status code must be equal to or greater than 200 and less than 400 to be considered successful


Readiness Probe- Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from service load balancers. 

Startup Probe - way of managing slow-starting containers. When a Pod is started, the startup probe is run before any other probing of the Pod is started. The startup probe proceeds until it either times out (in which case the Pod is restarted) or it succeeds, at which time the liveness probe takes over.

ubernetes also supports tcpSocket health checks - probe is useful for non-HTTP applications, such as databases or other non–HTTP-based APIs.

exec probes - are often useful for custom application validation logic that doesn’t fit neatly into an HTTP call

Resource requests specify the minimum amount of a resource required to run the application. Resource limits specify the maximum amount of a resource that an application can consume. 

Pod that requested a minimum of 0.5 of a core and 128 MB of memory.  add a limit of 1.0 CPU and 256 MB of memory.

Different Ways of Using Volumes with Pods
Communication/synchronization , Cache - emptyDir volume. Such a volume is scoped to the Pod’s lifespan, but it can be shared between two containers . survive a container restart due to a health-check failure.

Persistent data - wide variety of remote network storage volumes, including widely supported protocols like NFS and iSCSI as well as cloud provider network storage like Amazon Elastic Block Store, Azure File and Azure Disk, and Google’s Persistent Disk.

Mounting the host filesystem - hostPath volume, which can mount arbitrary locations on the worker node into the container

Example 
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: "kuard-data"
      nfs:
        server: my.nfs.server.local
        path: "/exports"
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      volumeMounts:
        - mountPath: "/data"
          name: "kuard-data"
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 30
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
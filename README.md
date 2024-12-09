# kubernetes-troubleshooting-zero-to-hero
Learn how to troubleshoot the most common Kubernetes Issues

## Day-01

### ImagePullBackOff

Video Link - https://youtu.be/vGab4v3RWEw

When a kubelet starts creating containers for a Pod using a container runtime, it might be possible the container is in Waiting state because of ImagePullBackOff.

The status ImagePullBackOff means that a container could not start because Kubernetes could not pull a container image for reasons such as 

- Invalid image name or 
- Pulling from a private registry without imagePullSecret. 

The BackOff part indicates that Kubernetes will keep trying to pull the image, with an increasing back-off delay.

Kubernetes raises the delay between each attempt until it reaches a compiled-in limit, which is 300 seconds (5 minutes).


## Day-02

### CrashLoopBackOff

When you see "CrashLoopBackOff," it means that kubelet is trying to run the container, but it keeps failing and crashing. After crashing, Kubernetes tries to restart the container automatically, but if the container keeps failing repeatedly, you end up in a loop of crashes and restarts, thus the term "CrashLoopBackOff." 

This situation indicates that something is wrong with the application or the configuration that needs to be fixed.

## Day-03

### Pods not schedulable

In Kubernetes, the scheduler is responsible for assigning pods to nodes in the cluster based on various criteria. Sometimes, you might encounter situations where pods are not being scheduled as expected. This can happen due to factors such as node constraints, pod requirements, or cluster configurations.

1. Node Selector
2. Node Affinity
3. Taints
4. Tolerations

## Day-04 

## Forbiden pod error
    1 Check Error Message:
Read the error message to understand what action was denied and for which resource.

   2 Verify Context and User:
Ensure you're using the correct kubectl context and user.(kubectl config current-context)

     3 Check User Permissions:
Verify what actions the user can perform.
kubectl auth can-i --list

     4 Inspect RBAC Roles and Bindings:
List and describe relevant roles and bindings

    5 Adjust RBAC Settings:
Grant necessary permissions by creating or modifying roles and bindings.

## Day-05
OOMKILLLED ERROR ( exit code 137) 
*  Exit Code 127 in Kubernetes typically indicates that a command or binary could not be found or executed within the container For example Command Not Found,Incorrect Path, Missing Dependencies, Insufficient Permissions:
* Exit (code 1)  Kubernetes typically means a container terminated due to an error. It's a generic error, making it challenging to diagnose for example Application Errors, Container Configuration Issues,Failed Health Checks,Dependency Issues,Resource Limit Constraints

Out of Memory (OOM) killed errors in Kubernetes occur when a container exceeds its memory limits, causing the system to terminate the process to free up memory. Here's how to troubleshoot and resolve OOMKilled errors in Kubernetes:
kubectl describe pod <pod-name> -n <namespace>

     1 Identify the OOMKilled Pod:
kubectl describe pod <pod-name> -n <namespace>

2 Check Resource Limits:
Ensure the pod has appropriate memory requests and limits set.
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 10 resources

3 Increase Memory Limits:
Edit the deployment or pod to increase the memory limit.
kubectl edit deployment <deployment-name> -n <namespace>

4 Update the resources section:
yaml
Copy code
resources:
  requests:
    memory: "512Mi"
  limits:
    memory: "1Gi"
  
6  CONFIG OUT OF DATE

ConfigMap or Secret used by the pods has been updated, but pods are not reloaded.
Solution:
Trigger a rolling restart to apply new configuration:
kubectl rollout restart deployment <deployment-name>
Use a reloader (like Reloader or Kustomize) to automatically restart pods on config updates.

** RELOADER
    
Tools like Reloader help to automatically restart pods when a ConfigMap or Secret is updated.
Steps to Implement Reloader:
Deploy Reloader:
helm repo add stakater https://stakater.github.io/stakater-charts
helm install reloader stakater/reloader
Annotate deployments to watch ConfigMaps or Secrets:
annotations:
  reloader.stakater.com/match: "true"

7 ENABLE SERVICE LINK 

The enableServiceLinks field determines whether environment variables are injected into the container for linked services. Problems can occur with unnecessary environment variables.
Solution:
Disable enableServiceLinks in the pod spec if it’s not required:
spec:
  enableServiceLinks: false
The enableServiceLinks field in a Kubernetes Pod specification determines whether environment variables are automatically injected into the container for all services in the namespace. These environment variables are derived from service properties, such as their Cluster IP and port.
   How it Works
When enableServiceLinks is set to true (default behavior), Kubernetes will:

Inject environment variables for all available services in the namespace into the container.
Follow a naming convention like:
<SERVICE_NAME>_SERVICE_HOST for the service's Cluster IP.
<SERVICE_NAME>_SERVICE_PORT for the service's port.

8 CREATE CONTAINER ERROR

Issues with the container image (e.g., not found, unauthorized, corrupt).
Misconfigured container specifications (e.g., invalid commands or environment variables).
Solution:
Check pod events for details:
kubectl describe pod <pod-name>

Verify the container image is accessible and correct:
docker pull <image-name>
Fix errors in container specifications in the deployment YAML.


9 Multi-Attach Volume Error in Kubernetes

This error typically occurs when a Persistent Volume (PV) backed by a storage type that does not support simultaneous attachment to multiple nodes is being accessed by multiple pods or nodes at the same time. This is common with volume types like AWS EBS, GCE PD, or Azure Disks, which by design allow only one node to attach to the volume at any given time.
Common Scenarios Causing Multi-Attach Errors

     *** Pods Scheduled on Multiple Nodes
Cause: When multiple replicas of a pod are scheduled on different nodes, all of them try to attach the same volume, leading to a conflict.
Solution:
Use a ReadWriteOnce (RWO) volume access mode and ensure only one replica of the pod uses the volume:

 *** StatefulSet with Incorrect Pod Affinity
Cause: StatefulSets usually expect one pod per node. If misconfigured, multiple pods might get scheduled, causing conflicts.
Solution:
Ensure StatefulSet is correctly defined and uses anti-affinity rules to prevent pods from being scheduled on the same node.

**** Volume Already Attached to Another Node
Cause: A volume may still be attached to a node due to a previous pod not releasing it correctly.
Solution:
Manually detach the volume from the node (e.g., in AWS):

*** Sing a Volume Type Without Multi-Attach Support
Cause: Attempting to use a storage class that does not support multi-attach (e.g., AWS EBS or Azure Disks).
Solution:
Switch to a volume type that supports ReadWriteMany (RWX) access mode, such as:
NFS, CephFS, GlusterFS,EFS (AWS)

10   Ingress Controller in Kubernetes: Possible Issues and Solutions

An Ingress Controller is a key component in Kubernetes for managing HTTP and HTTPS traffic to services within a cluster. It implements the rules defined in an Ingress resource, enabling routing, TLS termination, and other features. However, configuring and troubleshooting an Ingress Controller can present challenges.
       Common Issues with Ingress Controllers
       
****  Misconfigured Ingress Resource
Problem: The Ingress resource might have incorrect rules, causing traffic not to be routed correctly.
Solution:
Verify the Ingress definition:
kubectl describe ingress <ingress-name>
Check that the host field, paths, and backend service configurations match your services.

*** Missing or Misconfigured Ingress Controller
Problem: No Ingress Controller is deployed, or it's incorrectly configured.
Solution:
Deploy an Ingress Controller. Examples:
NGINX: Install via Helm:
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
Traefik or HAProxy as alternatives.
Ensure the controller is up and running:
kubectl get pods -n ingress-nginx

***  DNS Resolution Issues
Problem: The DNS name specified in the Ingress does not resolve to the correct IP address.
Solution:
Verify that the DNS record points to the Ingress Controller’s external IP:
kubectl get svc -n ingress-nginx
Update DNS records if needed.

***  Service Not Exposed
Problem: The service backend referenced by the Ingress is not reachable or not exposed.
Solution:
Ensure the service is ClusterIP, NodePort, or LoadBalancer type.
Verify that the service selector matches the pod labels:
kubectl describe svc <service-name>

*** SSL/TLS Termination Issues
Problem: TLS configuration is incorrect, resulting in connection failures.
Solution:
Verify the secret containing the TLS certificate:
kubectl describe secret <secret-name>
Check that the secret is referenced in the Ingress:

*** Port Conflicts
Problem: The Ingress Controller is not bound to the correct ports.
Solution:
Check the ports the controller is listening on:
kubectl describe svc ingress-nginx-controller -n ingress-nginx
Ensure no other processes are using those ports.

****  Backend Service Health
Problem: The backend service or pod is unhealthy.
Solution:
Check pod health:
kubectl get pods -o wide
Investigate health checks in the service definition.

*** RBAC or Namespace Issues
Problem: The Ingress Controller does not have permissions or is deployed in the wrong namespace.
Solution:
Verify RBAC roles:
kubectl get clusterrolebinding
Check the namespace where the Ingress Controller is deployed and ensure Ingress resources are created in the same or an allowed namespace.



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


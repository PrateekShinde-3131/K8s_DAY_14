# Kubernetes Taints, Tolerations, and Pod Scheduling

This document describes how to taint worker nodes, apply tolerations on pods, and manage pod scheduling on both worker nodes and the control plane node.

## Prerequisites
- A Kubernetes cluster with at least 1 control plane and 2 worker nodes (worker01, worker02).
- `kubectl` configured and connected to the cluster.
- Basic understanding of taints, tolerations, and pod scheduling in Kubernetes.

## Task Overview
1. **Taint the Worker Nodes**:
    - Taint `worker01` with `gpu=true:NoSchedule`
    - Taint `worker02` with `gpu=false:NoSchedule`
2. **Create a Pod with Nginx Image**:
    - Verify why it does not get scheduled on the worker and control plane nodes.
3. **Add a Toleration to Match the Taint on Worker01**:
    - The pod should now be scheduled on `worker01`.
4. **Remove the Taint from the Control Plane Node**:
    - Create a Redis pod and observe its scheduling behavior.
5. **Reapply the Taint on the Control Plane Node**:
    - Add the taint back after Redis pod scheduling.

## Step-by-Step Instructions

### 1. Taint Worker Nodes

Taint the worker nodes as follows:

```
# Taint worker01
kubectl taint nodes worker01 gpu=true:NoSchedule

# Taint worker02
kubectl taint nodes worker02 gpu=false:NoSchedule
```
### 2. Create a Pod with Nginx Image

Create a simple pod with the Nginx image:

```
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```
Apply the pod configuration:

```
kubectl apply -f nginx-pod.yaml
```
Check the pod status to see why it is not being scheduled:

```
kubectl describe pod nginx-pod
```
The pod should not be scheduled because there are taints on the nodes, and no toleration is present on the pod.

### 3. Add Toleration to the Pod for Worker01
To allow the pod to be scheduled on worker01, modify the pod definition to include a toleration:

```
# nginx-pod-toleration.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```
Apply the modified pod configuration:

```
kubectl apply -f nginx-pod-toleration.yaml
```
Now, the pod should be scheduled on worker01.


### 4. Remove the Taint from the Control Plane Node
Remove the taint from the control plane node to allow pods to be scheduled:

```
kubectl taint nodes <control-plane-node-name> <taint-key>-
```
Verify the taint removal:

```
kubectl describe node <control-plane-node-name> | grep Taints
```
### 5. Create a Pod with Redis Image
Create a pod with the Redis image:

```
# redis-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - name: redis
    image: redis
```
Apply the pod configuration:

```
kubectl apply -f redis-pod.yaml
```
The pod should now be scheduled on the control plane node since there are no taints.

### 6. Reapply the Taint on the Control Plane Node
Reapply the taint on the control plane node:

```
kubectl taint nodes <control-plane-node-name> <taint-key>=<taint-value>:<effect>
```


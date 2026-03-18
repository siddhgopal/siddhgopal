# ⚓ Common Errors Faced by DevOps Engineers in Kubernetes

> Kubernetes is powerful, but it's not without its challenges. As a DevOps engineer, you'll likely encounter a few common errors that can disrupt your workflow. Below is a simple, easy-to-understand breakdown of these issues and how to fix them.

---

## 📋 Table of Contents

| # | Error | Quick Fix |
|---|-------|-----------|
| 1 | [CrashLoopBackOff](#1-crashloopbackoff) | Check logs + resources |
| 2 | [ImagePullBackOff](#2-imagepullbackoff) | Check image name + credentials |
| 3 | [Pod Pending](#3-pod-pending) | Check node resources |
| 4 | [Failed Mount](#4-failed-mount) | Check PV/PVC config |
| 5 | [Node Not Ready](#5-node-not-ready) | Check Kubelet + network |
| 6 | [503 Service Unavailable](#6-503-service-unavailable-ingressload-balancer) | Check Ingress + routing |
| 7 | [Resource Quota Exceeded](#7-resource-quota-exceeded) | Free up namespace resources |
| 8 | [Kubelet Logs Filling Disk](#8-kubelet-logs-filling-up-disk) | Set up log rotation |
| 9 | [API Server Throttling](#9-api-server-throttling) | Reduce API calls |
| 10 | [Pod Eviction](#10-pod-eviction) | Adjust resource limits |

---

## 1. CrashLoopBackOff

### 🔴 What it is
Your container keeps crashing and Kubernetes tries restarting it, but it can't recover. The container starts and stops repeatedly.

### ⚠️ Possible Causes
- Misconfigured environment variables or app settings
- Missing dependencies
- Not enough resources (CPU, memory)
- Bugs in your app causing it to crash

### ✅ How to Fix

```bash
# Check pod logs
kubectl logs <pod-name>

# Check previous crash logs
kubectl logs <pod-name> --previous

# Describe pod for more details
kubectl describe pod <pod-name>
```

> 💡 **Tip:** Review your resource requests and make sure your Pod has enough CPU and memory. Verify environment variables and settings.

---

## 2. ImagePullBackOff

### 🔴 What it is
Kubernetes can't pull the Docker image for your container. The application won't start because the image is unavailable.

### ⚠️ Possible Causes
- Incorrect image name or tag
- Private registry credentials missing or incorrect
- Network issues preventing access to the image registry

### ✅ How to Fix

```bash
# Check pod events
kubectl describe pod <pod-name>

# Test pulling the image manually
docker pull <image-name>

# Check image pull secret
kubectl get secret <secret-name> -o yaml
```

> 💡 **Tip:** Double-check the image name and tag. Verify that Kubernetes has proper credentials for your private registry.

---

## 3. Pod Pending

### 🔴 What it is
Your Pod is stuck in the `Pending` state and won't start.

### ⚠️ Possible Causes
- Insufficient resources on the available nodes (CPU, memory)
- Resource requests for the Pod are too high
- Node affinity or other scheduling issues

### ✅ How to Fix

```bash
# Check node resource availability
kubectl describe node

# Check pod events
kubectl describe pod <pod-name>

# Check all nodes status
kubectl get nodes
```

> 💡 **Tip:** Lower your Pod's resource requests to fit available nodes. Ensure your Pod is allowed to run on the available nodes.

---

## 4. Failed Mount

### 🔴 What it is
A volume (like a persistent storage volume) can't be mounted to your Pod.

### ⚠️ Possible Causes
- No available Persistent Volume (PV) matching the claim
- Incorrect permissions or storage class settings
- Volume misconfiguration in your YAML files

### ✅ How to Fix

```bash
# Check PV and PVC status
kubectl get pv
kubectl get pvc

# Describe PVC for details
kubectl describe pvc <pvc-name>

# Check storage class
kubectl get storageclass
```

> 💡 **Tip:** Ensure the Persistent Volume (PV) matches your Persistent Volume Claim (PVC). Check permissions and storage class for the volume.

---

## 5. Node Not Ready

### 🔴 What it is
A node in your Kubernetes cluster is marked as `Not Ready`, meaning it can't run any Pods.

### ⚠️ Possible Causes
- Network issues or loss of connectivity
- Resource exhaustion (out of CPU or memory)
- Issues with the Kubernetes components like Kubelet not running

### ✅ How to Fix

```bash
# Check node status
kubectl describe node <node-name>

# Check Kubelet logs
journalctl -u kubelet

# Check node resource usage
kubectl top node <node-name>
```

> 💡 **Tip:** Resolve any resource issues or overloading on the node. Check the Kubelet logs to diagnose issues.

---

## 6. 503 Service Unavailable (Ingress/Load Balancer)

### 🔴 What it is
Your service is unreachable, returning a `503 Service Unavailable` error. This typically happens with Ingress controllers or Load Balancers.

### ⚠️ Possible Causes
- Misconfigured Ingress or incorrect service routing
- Backend services or Pods not responding
- DNS or routing issues

### ✅ How to Fix

```bash
# Check Ingress resource
kubectl describe ingress <ingress-name>

# Check backend pods health
kubectl get pods

# Check service endpoints
kubectl get endpoints <service-name>
```

> 💡 **Tip:** Ensure backend services (Pods) are healthy. Review DNS and routing configurations for errors.

---

## 7. Resource Quota Exceeded

### 🔴 What it is
You've exceeded the resource limits (CPU, memory, storage) set for your namespace, so no new Pods can be created.

### ⚠️ Possible Causes
- The namespace has reached its resource limits
- Unused resources are consuming quota space

### ✅ How to Fix

```bash
# Check current quota usage
kubectl describe quota

# Check all resources in namespace
kubectl get all -n <namespace>

# Delete unused resources
kubectl delete pod <pod-name> -n <namespace>
```

> 💡 **Tip:** Free up resources or adjust the limits to allow for more resources to be used.

---

## 8. Kubelet Logs Filling Up Disk

### 🔴 What it is
Kubernetes logs from the Kubelet and other components are filling up disk space, potentially causing system instability.

### ⚠️ Possible Causes
- Logs aren't being rotated, accumulating over time
- High error rates or debug-level logs are being written frequently

### ✅ How to Fix

```bash
# Check disk usage
df -h

# Check kubelet log size
journalctl -u kubelet --disk-usage

# Clear old logs
journalctl --vacuum-time=2d
```

> 💡 **Tip:** Set up log rotation with tools like **Fluentd** or **Logrotate**. Monitor disk space usage regularly.

---

## 9. API Server Throttling

### 🔴 What it is
The Kubernetes API server is receiving too many requests and starts throttling, causing delays or dropped requests.

### ⚠️ Possible Causes
- Excessive or inefficient API calls from automation or other tools
- The API server is running out of resources (CPU, memory)

### ✅ How to Fix

```bash
# Monitor API server resource usage
kubectl top nodes

# Check API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check current resource usage
kubectl describe node <node-name>
```

> 💡 **Tip:** Optimize your automation tools to reduce unnecessary API calls. Monitor the API server's resource usage and adjust resource limits.

---

## 10. Pod Eviction

### 🔴 What it is
Kubernetes evicts (removes) Pods from a node due to resource pressure, such as running out of memory or disk space.

### ⚠️ Possible Causes
- Resource exhaustion (like running out of memory or disk space)
- Pods exceeding their defined resource limits

### ✅ How to Fix

```bash
# Check eviction reason
kubectl describe pod <pod-name>

# Monitor node resource usage
kubectl top nodes
kubectl top pods

# Check node conditions
kubectl get node <node-name> -o yaml | grep -A5 conditions
```

> 💡 **Tip:** Adjust resource requests and limits to prevent eviction. Use `kubectl top nodes` to monitor resource usage and prevent overloading.

---

## 🔧 Quick Reference Commands

```bash
# Most used debugging commands
kubectl get pods -A                          # All pods all namespaces
kubectl describe pod <pod-name>              # Pod details + events
kubectl logs <pod-name>                      # Pod logs
kubectl logs <pod-name> --previous           # Previous crash logs
kubectl exec -it <pod-name> -- /bin/bash     # Shell into pod
kubectl get events --sort-by=.lastTimestamp  # Recent events
kubectl top nodes                            # Node resource usage
kubectl top pods                             # Pod resource usage
```

---

## 👨‍💻 Author

**Siddhgopal Soni** — DevOps Engineer | AWS · Kubernetes · Terraform · CI/CD

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/siddhgopal-soni-010846b8)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)](https://github.com/siddhgopal)

---

⭐ *If this helped you, give this repo a star!*

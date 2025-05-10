# Kubernetes Pod Troubleshooting

This lab will help you develop practical skills for diagnosing and resolving common issues with Kubernetes pods.

## Exercise 1: Understanding Pod Lifecycle and States

First, let's create a simple deployment with a deliberately incorrect image name:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: broken-app
  template:
    metadata:
      labels:
        app: broken-app
    spec:
      containers:
      - name: broken-container
        image: nginx:nonexistent-tag
        ports:
        - containerPort: 80
```

Save this to broken-deployment.yaml and apply it:

```bash
kubectl apply -f broken-deployment.yaml
```

Now investigate the pod states:

```bash
kubectl get pods -l app=broken-app
```

You should see pods in an error state (ImagePullBackOff or ErrImagePull).

## Exercise 2: Diagnosing Pod Issues

Let's learn to diagnose the image pull failure:

```bash
# Get detailed information about the pod
kubectl describe pod <pod-name>
```

Look for the "Events" section at the bottom of the output, which should show image pull errors.

Also, check the logs:

```bash
kubectl logs <pod-name>
```

You may not see logs since the container hasn't started.

## Exercise 3: Fixing the Image Issue

Create a corrected deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fixed-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fixed-app
  template:
    metadata:
      labels:
        app: fixed-app
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.20.0
        ports:
        - containerPort: 80
```

Save this to fixed-deployment.yaml and apply it:

```bash
kubectl apply -f fixed-deployment.yaml
```

Verify that the pods are running:

```bash
kubectl get pods -l app=fixed-app
```

## Exercise 4: Troubleshooting Application Crashes

Create a pod that will crash on startup:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashing-pod
spec:
  containers:
  - name: crashing-container
    image: busybox
    command: ["/bin/sh", "-c", "echo Starting up...; sleep 10; echo Crashing...; exit 1"]
    resources:
      limits:
        memory: "64Mi"
        cpu: "100m"
  restartPolicy: Always
```

Save this to crashing-pod.yaml and apply it:

```bash
kubectl apply -f crashing-pod.yaml
```

Observe the pod's behavior:

```bash
kubectl get pod crashing-pod -w
```

You should see the pod moving between "Running" and "CrashLoopBackOff" states.

Examine the logs to understand the crash:

```bash
kubectl logs crashing-pod
```

You may need to use the `--previous` flag if the container has already restarted:

```bash
kubectl logs crashing-pod --previous
```

## Exercise 5: Troubleshooting Resource Constraints

Create a pod with insufficient resources:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-constrained-pod
spec:
  containers:
  - name: memory-hog
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "100Mi"
        cpu: "100m"
```

Save this to resource-constrained-pod.yaml and apply it:

```bash
kubectl apply -f resource-constrained-pod.yaml
```

Monitor the pod's status:

```bash
kubectl get pod resource-constrained-pod -w
```

The pod will be terminated due to OOMKilled (Out of Memory).

Check the pod's status and events:

```bash
kubectl describe pod resource-constrained-pod
```

## Exercise 6: Troubleshooting Network Connectivity

Create two pods to test network connectivity:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-test-1
  labels:
    app: network-test
spec:
  containers:
  - name: nginx
    image: nginx:1.20.0
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: network-test-2
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
```

Save this to network-test-pods.yaml and apply it:

```bash
kubectl apply -f network-test-pods.yaml
```

Create a service for the nginx pod:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: network-test
  ports:
  - port: 80
    targetPort: 80
```

Save this to nginx-service.yaml and apply it:

```bash
kubectl apply -f nginx-service.yaml
```

Now test connectivity from the second pod:

```bash
kubectl exec -it network-test-2 -- wget -O- nginx-service
```

If there's a network policy issue, this may fail. To diagnose, check if a network policy is restricting traffic:

```bash
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>
```

## Exercise 7: Troubleshooting Init Containers

Create a pod with a deliberately failing init container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  initContainers:
  - name: init-container
    image: busybox
    command: ["/bin/sh", "-c", "echo Init container running...; exit 1"]
  containers:
  - name: main-container
    image: nginx:1.20.0
```

Save this to init-container-pod.yaml and apply it:

```bash
kubectl apply -f init-container-pod.yaml
```

Check the pod status:

```bash
kubectl get pod init-container-pod
```

The pod should be in "Init:CrashLoopBackOff" state.

Examine the init container logs:

```bash
kubectl logs init-container-pod -c init-container
```

## Exercise 8: Using Exec for Live Debugging

Create a simple pod for debugging:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - name: debug-container
    image: ubuntu
    command: ["/bin/sh", "-c", "apt-get update && apt-get install -y curl && sleep 3600"]
```

Save this to debug-pod.yaml and apply it:

```bash
kubectl apply -f debug-pod.yaml
```

Wait for the pod to be running:

```bash
kubectl get pod debug-pod -w
```

Now, exec into the pod to run diagnostic commands:

```bash
kubectl exec -it debug-pod -- /bin/bash
```

Inside the pod, you can run various diagnostic commands:

```bash
# Check DNS resolution
nslookup kubernetes.default.svc.cluster.local

# Check network connectivity
curl -v nginx-service

# Check environment variables
env | sort

# Check mounted volumes
ls -la /var/run/secrets/kubernetes.io/serviceaccount/

# Exit the shell when done
exit
```

## Exercise 9: Troubleshooting Persistent Volume Issues

Create a pod with a PersistentVolumeClaim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: pvc-container
    image: nginx:1.20.0
    volumeMounts:
    - mountPath: "/data"
      name: test-volume
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: test-pvc
```

Save this to pvc-pod.yaml and apply it:

```bash
kubectl apply -f pvc-pod.yaml
```

Check the PVC status:

```bash
kubectl get pvc test-pvc
```

If the PVC is stuck in "Pending" status, there might be issues with your storage class or available PVs.

Check for available PVs:

```bash
kubectl get pv
```

Check the pod status and events:

```bash
kubectl describe pod pvc-pod
```

## Exercise 10: Debugging with Ephemeral Debug Containers (Kubernetes v1.18+)

For clusters supporting the Ephemeral Containers feature (beta in v1.20+):

Create a minimal pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: minimal-pod
spec:
  containers:
  - name: minimal-container
    image: k8s.gcr.io/pause:3.2
```

Save this to minimal-pod.yaml and apply it:

```bash
kubectl apply -f minimal-pod.yaml
```

Now add a debug container to this running pod (requires kubectl v1.18+):

```bash
kubectl debug -it minimal-pod --image=busybox --target=minimal-container
```

Alternatively, create a YAML for the debug container:

```yaml
apiVersion: v1
kind: EphemeralContainers
metadata:
  name: minimal-pod
spec:
  ephemeralContainers:
  - name: debugger
    image: busybox
    command: ["sh"]
    stdin: true
    tty: true
    targetContainerName: minimal-container
```

Save to debug-container.yaml and apply using:

```bash
kubectl replace --raw /api/v1/namespaces/default/pods/minimal-pod/ephemeralcontainers -f debug-container.yaml
```

Then connect to the debug container:

```bash
kubectl attach -it minimal-pod -c debugger
```

## Bonus: Collecting All Pod Information for Later Analysis

Create a script to collect troubleshooting information for all pods:

```bash
#!/bin/bash
mkdir -p pod-diagnostics
cd pod-diagnostics

# Get all pods
kubectl get pods --all-namespaces -o wide > all-pods.txt

# Loop through each pod
for pod in $(kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'); do
  namespace=$(echo $pod | cut -f1 -d/)
  podname=$(echo $pod | cut -f2 -d/)
  
  echo "Collecting information for $namespace/$podname"
  
  # Create directories
  mkdir -p "$namespace/$podname"
  
  # Get pod details
  kubectl describe pod $podname -n $namespace > "$namespace/$podname/describe.txt"
  
  # Get pod logs for each container
  for container in $(kubectl get pod $podname -n $namespace -o jsonpath='{.spec.containers[*].name}'); do
    kubectl logs $podname -c $container -n $namespace > "$namespace/$podname/logs-$container.txt"
    # Try to get previous logs if they exist
    kubectl logs $podname -c $container -n $namespace --previous > "$namespace/$podname/logs-$container-previous.txt" 2>/dev/null
  done
  
  # Get pod events
  kubectl get events -n $namespace --field-selector involvedObject.name=$podname > "$namespace/$podname/events.txt"
done

echo "Diagnostics collected in pod-diagnostics directory"
```

Save this as collect-pod-diagnostics.sh, make it executable, and run it:

```bash
chmod +x collect-pod-diagnostics.sh
./collect-pod-diagnostics.sh
```

This script creates a directory structure with comprehensive diagnostic information for all pods in your cluster, which is valuable for offline analysis or sending to support teams.

## Common Pod States and Their Meanings

- **Pending**: The pod has been accepted but not yet scheduled or containers are still downloading.
- **Running**: The pod has been bound to a node, and all containers have been created and at least one is running.
- **Succeeded**: All containers in the pod have terminated successfully and will not be restarted.
- **Failed**: All containers in the pod have terminated, and at least one container has terminated in failure.
- **Unknown**: The state of the pod could not be determined.
- **ImagePullBackOff/ErrImagePull**: Container image can't be pulled from the registry.
- **CrashLoopBackOff**: The container is crashing repeatedly.
- **OOMKilled**: The container was terminated due to Out of Memory.
- **ContainerCreating**: Containers are being created.

Remember that troubleshooting in Kubernetes requires a methodical approach:
1. Identify the pod and its state
2. Check the pod description for events
3. Examine container logs
4. Use exec to interact with the container if possible
5. Check dependent resources (volumes, secrets, etc.)
6. Examine networking and connectivity

This systematic approach will help you efficiently diagnose and resolve most Kubernetes pod issues.


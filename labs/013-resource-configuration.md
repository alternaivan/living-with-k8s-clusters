# Kubernetes Resource Configuration

This exercise will help you understand how to configure resource limits and requests in Kubernetes.

## Exercise 1: Create a Deployment with Resource Requests

First, create a manifest for a deployment with CPU and memory requests. Here's an example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
```

Save this YAML into a file, say nginx-resources.yaml. Then, apply the manifest file to create the deployment:

```bash
kubectl apply -f nginx-resources.yaml
```

## Exercise 2: Add Resource Limits to the Deployment

Now, let's update the deployment to include resource limits as well:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

Update the deployment:

```bash
kubectl apply -f nginx-resources.yaml
```

## Exercise 3: Configure Ephemeral Storage

Let's create a new deployment with ephemeral storage configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-storage
  template:
    metadata:
      labels:
        app: nginx-storage
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "128Mi"
            cpu: "500m"
            ephemeral-storage: "2Gi"
```

Save this to nginx-storage.yaml and apply:

```bash
kubectl apply -f nginx-storage.yaml
```

## Exercise 4: Verify Resource Settings

Check the resource settings of your pods:

```bash
kubectl describe pod -l app=nginx
kubectl describe pod -l app=nginx-storage
```

Look for the "Limits" and "Requests" sections in the output.

## Exercise 5: Test Resource Limits with a Stress Pod

Create a pod that will attempt to consume more memory than allowed:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-test-pod
spec:
  containers:
  - name: memory-test-container
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
    resources:
      requests:
        memory: "50Mi"
        cpu: "100m"
      limits:
        memory: "100Mi"
        cpu: "200m"
```

Save this to memory-test.yaml and apply:

```bash
kubectl apply -f memory-test.yaml
```

Watch what happens to the pod:

```bash
kubectl get pods memory-test-pod -w
```

You should observe the pod being terminated due to exceeding its memory limit.

## Exercise 6: Create a ResourceQuota for a Namespace

Create a new namespace with a ResourceQuota:

```bash
kubectl create namespace resource-test
```

Create a ResourceQuota for this namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: resource-test
spec:
  hard:
    pods: "10"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.ephemeral-storage: 5Gi
    limits.ephemeral-storage: 10Gi
```

Save this to quota.yaml and apply:

```bash
kubectl apply -f quota.yaml
```

## Exercise 7: Deploy Within the Quota-Limited Namespace

Try to deploy multiple nginx instances in the resource-test namespace:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-quota-test
  namespace: resource-test
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-quota
  template:
    metadata:
      labels:
        app: nginx-quota
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.0
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
```

Save this to nginx-quota.yaml and apply:

```bash
kubectl apply -f nginx-quota.yaml
```

Check how many pods were created and why:

```bash
kubectl get pods -n resource-test
kubectl describe deployment nginx-quota-test -n resource-test
kubectl describe resourcequota compute-resources -n resource-test
```

The deployment should be limited by the ResourceQuota you created.

## Bonus: Create a LimitRange for Default Values

Create a LimitRange to set default resource constraints for containers in the namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: resource-test
spec:
  limits:
  - default:
      cpu: "300m"
      memory: "200Mi"
      ephemeral-storage: "500Mi"
    defaultRequest:
      cpu: "100m"
      memory: "50Mi"
      ephemeral-storage: "100Mi"
    type: Container
```

Save this to limit-range.yaml and apply:

```bash
kubectl apply -f limit-range.yaml
```

Now create a pod without specifying any resource constraints:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-pod
  namespace: resource-test
spec:
  containers:
  - name: default-container
    image: nginx:1.20.0
```

Save this to default-pod.yaml and apply:

```bash
kubectl apply -f default-pod.yaml
```

Check the pod's resource settings:

```bash
kubectl describe pod default-pod -n resource-test
```

You should see that the default limits and requests from the LimitRange have been applied.

Remember to replace any values as needed for your specific Kubernetes environment. These exercises demonstrate the full range of resource configuration capabilities in Kubernetes.


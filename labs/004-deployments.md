# Kubernetes Deployment

This exercise will help you understand how to manage Deployments in Kubernetes.

## Exercise 1: Create a Deployment from Manifest

First, create a manifest for a deployment. Here's an example of a simple deployment manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Save this YAML into a file, say nginx-deployment.yaml. Then, apply the manifest file to create the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

## Exercise 2: Scale Created Deployment

Scale the deployment to 5 replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

## Exercise 3: Demonstrate Manual Deletion of a Replica

First, get the pods of the deployment:

```bash
kubectl get pods -l app=nginx
```

Then, delete one of the pods manually:

```bash
kubectl delete pod <pod-name>
```

Replace <pod-name> with the name of one of the pods.

## Exercise 4: Upgrade the Existing Deployment

Upgrade the deployment to use nginx:1.16.1 image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

## Exercise 5: Rollback to the First Revision

Rollback the deployment to its first revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

## Bonus: Provide Description to rollout/deployment

You can also provide/update the `CHANGE-CAUSE` for the deployment by using the following command:

```bash
kubectl patch deployment nginx-deployment --patch '{"metadata": {"annotations": {"kubernetes.io/change-cause": "Upgrade nginx"}}}'
```

Remember to replace `nginx-deployment`, `nginx`, `nginx:1.14.2`, `80`, and `<pod-name>` with your actual deployment name, app label, image, container port, and pod name.


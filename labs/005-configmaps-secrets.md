# Kubernetes ConfigMaps and Secrets

This exercise will help you understand how to manage ConfigMaps and Secrets in Kubernetes.

## Exercise 1: Create a ConfigMap

First, create a ConfigMap named `my-config` with a key-value pair `my-key=my-value`:

```bash
kubectl create configmap my-config --from-literal=my-key=my-value
```

## Exercise 2: Map it to a Running Object as a Configuration Parameter

Assuming you have a running pod named my-pod, you can map the ConfigMap to it as an environment variable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: MY_KEY
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: my-key
```

## Exercise 3: Create a Secret

Create a Secret named my-secret with a key-value pair my-secret-key=my-secret-value:

```bash
kubectl create secret generic my-secret --from-literal=my-secret-key=my-secret-value
```

## Exercise 4: Map it to a Running Object as a Volume

You can map the Secret to a running pod as a volume. Here's an example of how to do it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
      - name: secret-volume
        mountPath: "/etc/secret"
        readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
```

Remember to replace `my-config`, `my-key`, `my-value`, `my-pod`, `my-container`, `nginx`, `MY_KEY`, `my-secret`, `my-secret-key`, `my-secret-value`, and `/etc/secret` with your actual ConfigMap name, key, value, pod name, container name, image, environment variable, Secret name, Secret key, Secret value, and mount path.

## Exercise 5: Go into the pod and check the secret

To check how secret is mapped within a Pod, execute below command to enter the pod.

```bash
kubectl exec -ti my-pod -- /bin/bash
```

Next, go into the mount path of the secret and use `cat` to show the results.

```bash
cat /etc/secrets/my-secret
```

You will see the secret mapped. If you need to decode it, just take the string from the value and run it into `base64 -d`. Use the following command.

```bash
echo "dGhpc2lzYXRlc3QK" | base64 -d
```

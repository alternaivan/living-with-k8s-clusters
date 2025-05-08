# Kubernetes Pod Scheduling

This exercise will help you understand how to schedule Pods using node labels in Kubernetes.

## Exercise 1: Add a Label to a Node

First, add a label to a node. Replace `my-node` and `disktype` with your node name and label key:

```bash
kubectl label nodes my-node disktype=ssd
```

## Exercise 2: Create a Manifest for a Pod with a nodeSelector Field

Create a manifest for a pod that includes the nodeSelector field. This field matches the label you added to the node:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
  nodeSelector:
    disktype: ssd
```

Save this YAML into a file, say my-pod.yaml.

## Exercise 3: Apply the Created Manifest

Apply the manifest file to create the pod:

```bash
kubectl apply -f my-pod.yaml
```

## Exercise 4: See Where the Pod is Running

Check on which node the pod is running:

```bash
kubectl get pod my-pod -o jsonpath='{.spec.nodeName}'
```

## Exercise 5: Delete Created Pods and Label

Finally, clean up by deleting the created pod and removing the label from the node:

```bash
kubectl delete pod my-pod
kubectl label nodes my-node disktype-
```

Remember to replace `my-pod`, `my-container`, `nginx`, `disktype`, `ssd`, and `my-node` with your actual pod name, container name, image, label key, label value, and node name.

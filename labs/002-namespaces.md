# Kubernetes Namespaces

This guide will help you work with namespaces in Kubernetes using `kubectl`.

## Step 1: List All Namespaces

To list all namespaces in your cluster, use the following command:

```bash
kubectl get namespaces
```

## Step 2: Create a Namespace Imperatively

To create a namespace imperatively, use the `kubectl create namespace` command followed by the name of the namespace. For example, to create a namespace named my-namespace:

```bash
kubectl create namespace my-namespace
```

## Step 3: Connect to the Created Namespace

To switch to your newly created namespace, you can use a context. Set the context for your current session like this:

```bash
kubectl config set-context --current --namespace=my-namespace
```

Now, all kubectl commands will operate in the `my-namespace` namespace.

## Step 4: Create a Namespace Manifest

A namespace can also be created declaratively using a manifest file. Here's an example of a manifest file for a namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

You can save this YAML into a file, say my-namespace.yaml.

Additionally, you can use the following command to create the namespace manifest, but not apply it to the cluster.

```bash
kubectl create namespace my-namespace --dry-run=client -o yaml > my-namespace.yaml
```

## Step 5: Create a Namespace from Manifest

To create the namespace from the manifest file, use the kubectl apply command:

```bash
kubectl apply -f my-namespace.yaml
```

This will create a namespace as defined in the my-namespace.yaml file.

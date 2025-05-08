# Kubernetes Network Policies Exercise

## Objective

This exercise will guide you through the process of creating a NetworkPolicy from a manifest, demonstrating a connection, denying a connection, removing the NetworkPolicy, and observing the outcome.


## Step 1. Demonstrate the Default Connection

As a first step before applying network policies, we'll create a deployment, expose it, and test if the connection to it is working.

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

Now create another Pod to test the connection.

```bash
kubectl run test-pod --image=busybox --restart=Never --rm -it -- wget -qO- http://nginx.default.svc.cluster.local:80
```

Is it allowed or denied? Why?

## Step 2. Create a NetworkPolicy from Manifest

Next, we need to create a NetworkPolicy. This is done by applying a manifest file which describes the policy. Here's an example of how to do it:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
```

Now we need to apply the NetworkPolicy with the following command.

```bash
kubectl apply -f network-policy.yaml
```

## Step 3. Demonstrate Connection

Now, let's test if we are able to connect directly to the resource. We'll use the same command as above: 

```bash
kubectl run denied-pod --image=busybox --restart=Never --rm -it -- wget -qO- http://nginx.default.svc.cluster.local:80 --timeout 10
```

You should see that the connection is denied.

### Allowing Connection

To check if the Network policy is working properly, we will test if we are able to connect to the pod with the following command:

```bash
kubectl run test-pod --labels="access=true" --image=busybox --restart=Never --rm -it -- wget -qO- http://nginx.default.svc.cluster.local:80 --timeout 10
```

The connection should be working. Why?


## Step 4. Remove the NetworkPolicy and See the Outcome

Finally, let's remove the NetworkPolicy and observe that the previously denied Pod can now connect:

```bash
kubectl delete networkpolicy test-network-policy
```

Now try to connect from the denied Pod again:

```bash
kubectl run denied-pod --image=busybox --restart=Never --rm -it -- wget -qO- http://nginx.default.svc.cluster.local:80 --timeout 10
```

You should see that the connection is now successful because the test-network-policy no longer exists.

## Conclusion

In this exercise, you have learned how to create a NetworkPolicy from a manifest, demonstrate connections, deny connections based on policy rules, remove a NetworkPolicy, and observe its impact on connections. This knowledge is crucial for managing network access in Kubernetes.

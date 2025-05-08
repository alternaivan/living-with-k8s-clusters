# Kubernetes Services Exercise

## Objective

This exercise will guide you through the process of creating a Pod, tying it to a service, demonstrating a connection from another Pod, deleting a service, and observing the impact.

## Step 1. Create a Pod and tie it to a Service

```bash
kubectl run my-pod --image=nginx --restart=Never --port=80
```

Now create a Service tied to the Pod

```bash
kubectl expose pod my-pod --name=my-service --target-port=80 --type=ClusterIP
```

## Step 2. Demonstrate a Connection from Another Pod

Create another Pod for testing the connection:

```bash
kubectl run test-pod --image=busybox --restart=Never --rm -it -- wget -qO- http://my-service.default.svc.cluster.local
```

## Step 3. Delete the Service

```bash
kubectl delete svc my-service
```

## Step 4. Demonstrate Impact

Try to connect from the test Pod again:

```bash
kubectl run test-pod --image=busybox --restart=Never --rm -it -- wget -qO- http://my-service.default.svc.cluster.local
```

You should see that the connection fails because the service my-service no longer exists.

## Conclusion

In this exercise, you have learned how to create a Pod and tie it to a service, connect from another Pod, delete a service, and observe the impact of deleting the service. This is fundamental knowledge for managing applications in Kubernetes.

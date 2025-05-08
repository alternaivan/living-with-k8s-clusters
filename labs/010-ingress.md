# Ingress Configuration 

## Prerequisite

Enable `HttpLoadBalancing` add on if not enabled with the following command:

```bash
gcloud container clusters update lab-cluster --update-addons=HttpLoadBalancing=ENABLED --location europe-west3-c
```

## Step 1. Create first demo app

Let's create a first demo app and apply it to the cluster:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment-1
spec:
  selector:
    matchLabels:
      greeting: hello
      version: one
  replicas: 3
  template:
    metadata:
      labels:
        greeting: hello
        version: one
    spec:
      containers:
      - name: hello-app-1
        image: "us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0"
        ports:
        - containerPort: 50000
        env:
        - name: "PORT"
          value: "50000"
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-1
spec:
  type: NodePort
  selector:
    greeting: hello
    version: one
  ports:
  - protocol: TCP
    port: 60000
    targetPort: 50000
```

Short notes on `Service` resource:

- Any Pod that has both the greeting: hello label and the version: one label is a member of the Service.
- GKE forwards requests sent to the Service on TCP port 60000 to one of the member Pods on TCP port 50000.
- The Service type is NodePort, which is required unless you're using container native load balancing. If using container-native load balancing, there is no restriction on type of service. Recommend to use type: ClusterIP.

Run the following command to create the first demo app:

```bash
kubectl apply -f hello-world-1.yaml
```

## Step 2. Create second demo app

Now create the second demo app, with the below example.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment-2
spec:
  selector:
    matchLabels:
      greeting: hello
      version: two
  replicas: 3
  template:
    metadata:
      labels:
        greeting: hello
        version: two
    spec:
      containers:
      - name: hello-app-2
        image: "us-docker.pkg.dev/google-samples/containers/gke/hello-app:2.0"
        ports:
        - containerPort: 8080
        env:
        - name: "PORT"
          value: "8080"
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-2
spec:
  type: NodePort
  selector:
    greeting: hello
    version: two
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

Run the following command to create the second demo app:

```bash
kubectl apply -f hello-world-2.yaml
```

## Step 3. Create Ingress

Create an ingress from the below example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    # If the class annotation is not specified it defaults to "gce".
    kubernetes.io/ingress.class: "gce"
spec:
  ingressClassName: "gce"
  rules:
  - http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-world-1
            port:
              number: 60000
      - path: /v2
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-world-2
            port:
              number: 80
```

Run the following to create an Ingress:

```bash
kubectl apply -f ingress.yaml
```

This manifest describes an Ingress with the following properties:

- There are two GKE Ingress classes. To specify an Ingress class, you need to use `spec.ingressClassName`.
- The `gce` class deploys an external Application Load Balancer.
- The `gce-internal` class deploys an internal Application Load Balancer.
- When you deploy an Ingress resource without the annotations spec.ingressClassName and kubernetes.io/ingress.class, GKE creates an external Application Load Balancer.
- When a client sends a request to the load balancer with URL path /, GKE forwards the request to the hello-world-1 Service on port 60000. When a client sends a request to the load balancer using URL path /v2, GKE forwards the request to the hello-world-2 Service on port 80.

## Step 4. Test external LB

Check the IP of the LB with the following command:

```bash
kubectl get ingress my-ingress --output yaml
```

The output should show the IP of the LB under `status` section.

Next, test `/` and `/v2` endpoints.

```bash
curl load-balancer-ip/
curl load-balancer-ip/v2
```

```bash
# output1
Hello, world!
Version: 1.0.0
Hostname: ...

# output 2
Hello, world!
Version: 2.0.0
Hostname: ...
```

More information can be found [here](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress)

Check [this](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress#how-ingress-xlb-works) to find out how ingress for external load balancing works!

# Kubernetes Persistent Volumes - Static Provisioning Exercise

## Step 1: Create a PV, and a PVC Manifests

First, we need to create a Persistent Volume (PV) manifest. Here's an example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
```

Save this as my-pv.yaml and create the PV with 

```shell
kubectl apply -f my-pv.yaml
```

Now, create a PVC manifest with a following example.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```shell
kubectl apply -f my-pvc.yaml
```

## Step 2: Create a Deployment Which Uses the PVC

Next, we need to create a Deployment that uses the PV. Here's an example of a Deployment that uses the PV:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: my-storage
          persistentVolumeClaim:
            claimName: my-pvc
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          volumeMounts:
            - name: my-storage
              mountPath: "/usr/share/nginx/html"
```

Save this as my-deployment.yaml and create the Deployment with kubectl apply -f my-deployment.yaml.

## Step 3: Test the Persistence

To test the persistence, you can write some data to the mounted path in one of the containers, delete the Deployment, and then recreate it. The data should still be there.

## Step 4: Delete the Deployment and See What Happens

Finally, delete the Deployment with 

```shell
kubectl delete -f my-deployment.yaml
```

Because we set persistentVolumeReclaimPolicy to Retain in our PV, the data will not be deleted when the Deployment is deleted. You can verify this by recreating the Deployment and checking the data.

Remember to clean up after you're done with 

```shell
kubectl delete -f my-pv.yaml.
```

That's it! You've successfully completed the exercise on Kubernetes Persistent Volumes.

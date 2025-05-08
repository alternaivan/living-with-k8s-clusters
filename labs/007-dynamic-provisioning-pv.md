# Kubernetes Persistent Volumes - Dynamic Provisioning Exercise

## Step 1: Show a StorageClass (SC)

Below is an example of a StorageClass (SC) for dynamic provisioning.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Use the following command to get all storage classes provisioned on the cluster.

```shell
kubectl get sc
```

## Step 2: Create a Workload with a PVC

Next, we need to create a workload that uses a Persistent Volume Claim (PVC). Here's an example of a Deployment that uses the PVC:

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

Save this as my-pvc.yaml and create the PVC with `kubectl apply -f my-pvc.yaml`.

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

Save this as my-deployment.yaml and create the Deployment with `kubectl apply -f my-deployment.yaml`. 

This Deployment will create Pods that use the PVC we defined earlier. The PVC will be mounted into the Pods at the /usr/share/nginx/html path. Any data written to this path will be stored on the Persistent Volume (PV) provisioned for the PVC. 

If a Pod is deleted and recreated, the data will still be there because it's stored on the PV, not in the Pod itself. This is how we can test persistence.

## Step 3: Test the Persistence

To test the persistence, you can write some data to the mounted path in one of the containers, delete the Deployment, and then recreate it. The data should still be there.

## Step 4: Delete the Workload and See What Happens

Finally, delete the Deployment with `kubectl delete -f my-deployment.yaml` and see what happens. 

## Step 5: Demonstrate Expansion of PVC

To demonstrate expansion of PVC, you can edit the my-pvc.yaml file and increase the storage request from 1Gi to 2Gi. Then apply the changes with `kubectl apply -f my-pvc.yaml`. Note that not all StorageClasses support volume expansion.

Remember to clean up after you're done with `kubectl delete -f ./`.

That's it! You've successfully completed the exercise on Kubernetes Dynamic Provisioning.

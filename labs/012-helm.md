# Helm Usage Exercise

## Objective

This exercise will guide you through the process of checking the structure of a Helm chart, deploying a Helm chart, demonstrating the history, upgrading, rolling back, and packaging a Helm chart.

## Prerequisites

- A running Kubernetes cluster
- [helm installed and configured](https://helm.sh/docs/intro/install/) 

## Step 1. Check the Structure of a Helm Chart

Helm uses a packaging format called charts. A chart is a collection of files that describe a related set of Kubernetes resources. To check the chart structure we can use the following command to create a dummy chart, or, we can use the existing chart and see what's inside.

```bash
helm create my-chart
```

If you want to use the existing chart, let's use the Apache one packaged by Bitnami. First, we're going to pull the chart from their repository.

```bash
helm pull oci://registry-1.docker.io/bitnamicharts/apache
```

Next, we'll unpackage it.

```bash
tar -xzvf apache-10.2.0.tgz
rm -rf apache-10.2.0.tgz
```

You can check the structure with the following:

```bash
tree apache/
```

## Step 2. Deploy a Helm Chart

Next, we'll deploy a Helm chart to your Kubernetes cluster with the helm install command:

```bash
helm install apache -n test oci://registry-1.docker.io/bitnamicharts/apache --set service.type=ClusterIP
```

## Step 3. Demonstrate the History

Helm tracks each release of your application, making it possible to rollback to previous versions. You can view the history of releases with the helm history command:

```bash
helm history -n test apache
```

## Step 4. Demonstrate Upgrading

You can upgrade your application with the helm upgrade command. Here, we'll upgrade the image tag to latest.

```bash
helm upgrade apache -n test oci://registry-1.docker.io/bitnamicharts/apache --set service.type=ClusterIP --set image.tag=latest
```

## Step 5. Demonstrate Rollback

If something goes wrong during an upgrade, you can rollback to a previous release with the helm rollback command. Here, we determined that the `latest` image won't work for us, so we decide to rollback.

```bash
helm rollback -n test apache 1
```

## Step 6. Package a Helm Chart

Finally, you can package your Helm chart into a versioned chart archive file with the helm package command:

```bash
helm package apache/
```

## Conclusion

In this exercise, you have learned how to check the structure of a Helm chart, deploy a Helm chart, demonstrate the history of releases, upgrade and rollback releases, and package a Helm chart. This knowledge is crucial for managing applications in Kubernetes with Helm.

# Useful gcloud commands

```bash
# enable network policy add on
gcloud container clusters update CLUSTER_NAME --update-addons=NetworkPolicy=ENABLED

# describe the cluster
gcloud container clusters describe CLUSTER_NAME --location LOCATION

# update kubeconfig
gcloud container clusters get-credentials LOCATION --zone ZONE --project PROJECT_NAME
```

# Monitor Environments with Google Cloud Managed Service for Prometheus: Challenge Lab || **GSP364**

**Command:**

```bash
export PROJECT_ID=$(gcloud config get-value project)
export ZONE=$(gcloud compute project-info describe --format="value(commonInstanceMetadata.items[google-compute-default-zone])")

gcloud config set compute/zone $ZONE
````

```bash
gcloud container clusters create gmp-cluster \
    --zone=$ZONE \
    --num-nodes=1 \
    --enable-managed-prometheus

gcloud container clusters get-credentials gmp-cluster --zone=$ZONE
```

```bash
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.7.0/manifests/setup.yaml
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.7.0/manifests/operator.yaml
```

```bash
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.7.0/examples/example-app.yaml
```

```bash
cat <<EOF > op-config.yaml
apiVersion: monitoring.googleapis.com/v1
kind: OperatorConfig
metadata:
  namespace: gmp-public
  name: config
collection:
  filter:
    matchOneOf:
    - '{job="prom-example"}'
    - '{__name__=~"job:.+"}'
EOF
```

```bash
kubectl apply -f op-config.yaml
```
```bash
export PROJECT=$(gcloud config get-value project)
gcloud storage buckets create --project=$PROJECT gs://$PROJECT
gcloud storage cp op-config.yaml gs://$PROJECT
gcloud storage buckets add-iam-policy-binding gs://$PROJECT --member=allUsers --role=roles/storage.objectViewer
```

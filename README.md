# homelab-gitops

## Setup
This is done automatically if using ansible [here](https://github.com/reecetec/homelab-ansible).
```bash
git clone https://github.com/reecetec/homelab-gitops.git
cd homelab-gitops/root
kubectl apply -f root-app.yml
```

### 1. Set secrets for minio and postgres
Note: this isn't fully secure and for actual usage something better should be done.
```bash
kubectl create secret generic postgres-secret \
  --namespace ducklake \
  --from-literal=username='postgres' \
  --from-literal=password='<password>' \
  --from-literal=database='ducklake'

kubectl create secret generic minio-secret \
  --namespace ducklake \
  --from-literal=MINIO_ROOT_USER='admin' \
  --from-literal=MINIO_ROOT_PASSWORD='<password>'
```

### 2. Create the MinIO Bucket (One-Time Setup for ducklake)

The first time you set up this environment, you must create the storage bucket in MinIO.

1.  **Forward the MinIO Console Port:**
    ```bash
    # Run this in a new terminal
    kubectl port-forward -n ducklake svc/minio-svc 9001:9001
    ```
2.  **Log in to the Console:**
    *   Open `http://localhost:9001` in your browser.
    *   Log in with creds set above
3.  **Create the Bucket:**
    *   Click the **"Create Bucket"** button.
    *   Name the bucket `ducklake`.


## Argo Events Hello World Example

Once ArgoCD is deployed, you'll have a working Argo Events hello world example:

### Test the Webhook
```bash
# Port forward to the webhook service
kubectl port-forward svc/webhook-hello-eventsource-svc -n argo-events 12000:12000

# In another terminal, trigger the webhook
curl -X POST http://localhost:12000/hello -d '{"message": "hello"}'
# Should return: success

# Check if the job was created
kubectl get jobs -n argo-events
# Should show: hello-world-xxxxx   Complete   1/1

# Check the job output
kubectl logs job/hello-world-xxxxx -n argo-events
# Should show: Hello World from Argo Events!
```

### Monitor Events
```bash
# Watch all Argo Events resources
kubectl get eventsources,sensors,eventbus -n argo-events

# Monitor events in real-time
kubectl get events -n argo-events --watch

# Check sensor logs
kubectl logs -f deployment/hello-world-sensor-sensor-xxxxx -n argo-events
```

For more details on adding new events and sensors, see [argo-events/README.md](argo-events/README.md).
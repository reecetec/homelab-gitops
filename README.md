# homelab-gitops

## Setup
This is done automatically if using ansible [here](https://github.com/reecetec/homelab-ansible).
```bash
git clone https://github.com/reecetec/homelab-gitops.git
cd homelab-gitops/root
kubectl apply -f root-app.yml
```

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
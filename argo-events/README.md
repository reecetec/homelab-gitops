# Argo Events Configuration

This directory contains a complete Argo Events setup with a clean, organized structure for easy expansion.

## Directory Structure

```
argo-events/
├── platform/           # Argo Events installation & EventBus
│   ├── kustomization.yaml
│   └── eventbus.yaml
├── rbac/               # Service accounts and permissions
│   ├── kustomization.yaml
│   └── workflow-rbac.yaml
├── eventsources/       # Event sources (webhooks, calendars, etc.)
│   ├── kustomization.yaml
│   └── webhook-hello.yaml
├── sensors/            # Event processors and triggers
│   ├── kustomization.yaml
│   └── hello-world.yaml
├── services/           # Custom services (optional)
│   └── kustomization.yaml
└── kustomization.yaml  # Main config that ties everything together
```

## Adding New Event Sources

1. Create a new EventSource file in `eventsources/`:
   ```yaml
   # eventsources/github-webhook.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: EventSource
   metadata:
     name: github-webhook
     namespace: argo-events
   spec:
     service:
       ports:
         - port: 12001
           targetPort: 12001
     webhook:
       deploy:
         port: "12001"
         endpoint: /deploy
         method: POST
   ```

2. Add it to `eventsources/kustomization.yaml`:
   ```yaml
   resources:
     - webhook-hello.yaml
     - github-webhook.yaml  # Add this line
   ```

## Adding New Sensors

1. Create a new Sensor file in `sensors/`:
   ```yaml
   # sensors/deploy-app.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Sensor
   metadata:
     name: deploy-app-sensor
     namespace: argo-events
   spec:
     template:
       serviceAccountName: operate-workflow-sa
     dependencies:
       - name: deploy-dep
         eventSourceName: github-webhook
         eventName: deploy
     triggers:
       - template:
           name: deploy-trigger
           k8s:
             operation: create
             source:
               resource:
                 # Your Kubernetes resource here
   ```

2. Add it to `sensors/kustomization.yaml`:
   ```yaml
   resources:
     - hello-world.yaml
     - deploy-app.yaml  # Add this line
   ```

## Testing the Hello World Example

```bash
# Port forward to the webhook
kubectl port-forward svc/webhook-hello-eventsource-svc -n argo-events 12000:12000

# Trigger the webhook
curl -X POST http://localhost:12000/hello -d '{"message": "hello"}'

# Check the created job
kubectl get jobs -n argo-events
kubectl logs job/hello-world-xxxxx -n argo-events
```

## Common Event Source Types

- **Webhook**: HTTP endpoints for external triggers
- **Calendar**: Time-based triggers
- **Resource**: Watch Kubernetes resources
- **File**: Watch file changes
- **AWS SNS/SQS**: Cloud messaging
- **Redis**: Pub/sub messaging
- **GitHub**: Git repository events

## Common Sensor Triggers

- **Kubernetes**: Create/update K8s resources
- **Argo Workflows**: Trigger workflow executions
- **HTTP**: Send HTTP requests
- **Slack**: Send notifications
- **Email**: Send emails
- **Custom**: Run custom containers

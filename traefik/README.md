# Exposing Services with Traefik

To expose a new service through Traefik, you can create an `IngressRoute` for it.

## Example: Exposing a "whoami" service

1.  **Create the `IngressRoute` manifest**:

    Create a new YAML file in this directory (e.g., `whoami-ingress.yaml`) with the following content:

    ```yaml
    apiVersion: traefik.containo.us/v1alpha1
    kind: IngressRoute
    metadata:
      name: whoami-ingress
      namespace: default # Or the namespace your service is in
    spec:
      entryPoints:
        - web
      routes:
        - match: Host(`whoami.homelab.local`)
          kind: Rule
          services:
            - name: whoami-service # The name of your service
              port: 80 # The port your service is listening on
    ```

2.  **Add to Kustomization**:

    Include the new file in `traefik/kustomization.yaml`:

    ```yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
      - dashboard-ingress.yaml
      - whoami-ingress.yaml # Add your new ingress route here
    ```

3.  **DNS Configuration**:

    Add a DNS entry for `whoami.homelab.local` pointing to your Traefik load balancer IP.

4.  **Commit and Push**:

    Commit the changes to your Git repository. ArgoCD will automatically apply the new configuration, and your service will be accessible at `http://whoami.homelab.local`.

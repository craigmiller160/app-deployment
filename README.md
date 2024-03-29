# App Deployment

A Helm chart that is the template for deploying applications to Kubernetes.

## Pre-Requisites

This chart depends on the `secure_ingress` chart, which must be published.

## Example Values For Consuming Application

```yaml
app_deployment:
  appName: db-backup-service
  image: nexus-docker-craigmiller160.ddns.net/db-backup-service:latest
  deploymentStrategy: Recreate
  # Optional, defaults to 1
  replicas: 1
  # Will create ConfigMap if provided
  configMap:
    spring.profiles.active: prod
    spring.datasource.host: postgres.infra-prod
    spring.datasource.port: "5432"
    oauth2.auth-server-host: https://sso-oauth2-server:8443
    oauth2.client-name: expense-tracker-api
    spring.datasource.db_name: expense_tracker_prod
  # Will create OnePasswordItems if provided
  onepassworditems:
    - name: expense-tracker-api-oauth2-keys
      itemPath: vaults/Home Server (Prod)/items/Expense Tracker API OAuth2 Keys
  secrets:
    - envVariable: db.postgres.user
      secretName: postgres-root-account
      secretKey: username
  # Use this if liveness & readiness probes are the same, otherwise use the individual ones
  commonProbe:
    # Option #1
    exec:
      command:
        - sh
        - /output/liveness.sh
    # Option #2
    httpGet:
      path: /actuator/health
      port: 8443
      scheme: HTTPS
  livenessProbe:
    # Options are the same as commonProbe
  readinessProbe:
    # Options are the same as commonProbe
  volumes:
    # Will create PVC
    - name: volume1
      type: PVC
      deploymentMountPath: /output
    # Will use HostPath
    - name: volume2
      type: HostPath
      hostPath: /otherPath
      deploymentMountPath: /output
    # Will use a secret, which must be available
    - name: volume3
      type: Secret
      deploymentMountPath: /secret
      secretName: my-secret
  # Optional, defaults to ClusterIP if not specified
  serviceType: ClusterIP
  # Will create service if specified
  ports:
    - containerPort: 8443
      # Only required if NodePort service
      nodePort: 30000
      # Only required if multiple ports
      name: ThePort
      # Optional, defaults to TCP
      protocol: TCP
  # Optional, will add a resources section if specified
  resources:
    limits:
      cpu: 50m
      memory: 200Mi
    requests:
      cpu: 100m
      memory: 200Mi
  secure_ingress:
    # This will configure the secure_ingress lib, see that repo for documentation
```
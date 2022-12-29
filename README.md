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
  secrets:
    - envVariable: db.postgres.user
      secretName: postgres-root-account
      secretKey: username
  livenessProbe:
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
```
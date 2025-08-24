# Advanced ArgoCD Features

## Multi-Environment Setup

### 1. Environment-Specific Applications
```yaml
# argocd-apps/sample-app-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-app-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/argocd-tutorial.git
    targetRevision: develop
    path: sample-app
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### 2. Using Kustomize for Environment Differences
```yaml
# environments/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../sample-app

patchesStrategicMerge:
- deployment-patch.yaml

# environments/dev/deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.21-dev
```

## App of Apps Pattern

```yaml
# argocd-apps/root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/argocd-tutorial.git
    targetRevision: HEAD
    path: argocd-apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Helm Integration

```yaml
# argocd-apps/helm-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.helm.sh/stable
    chart: nginx-ingress
    targetRevision: 1.41.3
    helm:
      parameters:
      - name: controller.service.type
        value: LoadBalancer
  destination:
    server: https://kubernetes.default.svc
    namespace: ingress-nginx
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
```

## Health Checks and Sync Hooks

```yaml
# sample-app/deployment.yaml (with health check)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  # ... existing spec ...
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## Resource Hooks

```yaml
# sample-app/pre-sync-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-sync-job
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
      - name: pre-sync
        image: busybox
        command: ["sh", "-c", "echo 'Running pre-sync tasks'"]
      restartPolicy: Never
```

## Sync Policies and Strategies

```yaml
# Advanced sync configuration
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: advanced-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/argocd-tutorial.git
    targetRevision: HEAD
    path: sample-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

## RBAC Configuration

```yaml
# Custom ArgoCD project with RBAC
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My custom project
  sourceRepos:
  - 'https://github.com/your-username/*'
  destinations:
  - namespace: 'my-*'
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  namespaceResourceWhitelist:
  - group: 'apps'
    kind: Deployment
  - group: ''
    kind: Service
  roles:
  - name: developer
    policies:
    - p, proj:my-project:developer, applications, sync, my-project/*, allow
    groups:
    - my-org:developers
```

## Monitoring and Notifications

```yaml
# Notification configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  template.app-deployed: |
    message: |
      {{if eq .serviceType "slack"}}:white_check_mark:{{end}} Application {{.app.metadata.name}} is now running new version.
  trigger.on-deployed: |
    - description: Application is synced and healthy
      send:
      - app-deployed
      when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
```

## Best Practices

1. **Use Projects** for multi-tenancy and RBAC
2. **Implement sync waves** for ordered deployments
3. **Add health checks** for better monitoring
4. **Use hooks** for database migrations and setup tasks
5. **Implement proper RBAC** for security
6. **Monitor application health** and set up notifications
7. **Use the App of Apps pattern** for managing multiple applications

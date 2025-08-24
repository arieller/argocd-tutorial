# Multi-tenancy in ArgoCD

## Overview
Multi-tenancy allows multiple teams to use the same ArgoCD instance while maintaining isolation and security.

## Key Components

### 1. ArgoCD Projects
- **Purpose**: Logical grouping and isolation boundary
- **Controls**: Source repos, destinations, resources, RBAC
- **Benefit**: Teams can only deploy from their repos to their namespaces

### 2. Namespaces
- **Purpose**: Runtime isolation in Kubernetes
- **Benefits**: Resource quotas, network policies, separate environments
- **Example**: `team-frontend-dev`, `team-frontend-prod`

### 3. RBAC Integration
- **Authentication**: OIDC/SAML with corporate identity provider
- **Authorization**: Project-based permissions + Kubernetes RBAC
- **Granular Control**: Who can sync, create, delete applications

## Multi-tenancy Patterns

### Pattern 1: Team-based Isolation
```
Teams: Frontend, Backend, Platform
├── Projects: team-frontend, team-backend, platform-infrastructure
├── Namespaces: frontend-*, backend-*, platform-*
└── Repos: team-specific repositories
```

### Pattern 2: Environment-based Isolation
```
Environments: Dev, Staging, Production
├── Projects: dev-apps, staging-apps, prod-apps
├── Namespaces: *-dev, *-staging, *-prod
└── Approval: Stricter controls for production
```

### Pattern 3: Customer-based Isolation (SaaS)
```
Customers: customer-a, customer-b, customer-c
├── Projects: customer-specific projects
├── Namespaces: customer-specific namespaces
└── Data Isolation: Complete separation per customer
```

## Implementation Example

### Step 1: Create Project Structure
```bash
# Apply team projects
kubectl apply -f examples/argocd-projects.yaml

# Create namespaces with quotas
kubectl apply -f examples/namespace-example.yaml
```

### Step 2: Configure RBAC
```bash
# Apply ArgoCD RBAC configuration
kubectl apply -f examples/argocd-rbac.yaml
```

### Step 3: Deploy Team Applications
```bash
# Deploy team-specific applications
kubectl apply -f examples/multitenant-apps.yaml
```

## Best Practices

### Security
- **Principle of Least Privilege**: Grant minimum required permissions
- **Network Policies**: Isolate traffic between teams/environments
- **Resource Quotas**: Prevent resource exhaustion
- **Image Policies**: Control which container images can be used

### Operational
- **Naming Conventions**: Clear, consistent naming for easy management
- **Monitoring**: Team-specific dashboards and alerts
- **Backup/Recovery**: Separate backup policies per team/environment
- **Cost Allocation**: Track resource usage per team

### Governance
- **Approval Workflows**: Require approvals for production deployments
- **Policy as Code**: Use tools like OPA Gatekeeper for policy enforcement
- **Audit Logging**: Track all changes and access
- **Documentation**: Clear runbooks and escalation procedures

## Common Scenarios

### Scenario 1: New Team Onboarding
1. Create ArgoCD project for the team
2. Set up team-specific namespaces
3. Configure RBAC permissions
4. Provide team with repository templates
5. Set up monitoring and alerting

### Scenario 2: Environment Promotion
1. Development → Staging (automated)
2. Staging → Production (manual approval)
3. Different sync policies per environment
4. Progressive deployment strategies

### Scenario 3: Incident Response
1. Team-specific alerts and dashboards
2. Rollback capabilities per team
3. Emergency access procedures
4. Post-incident review processes

## Monitoring Multi-tenancy

### ArgoCD Metrics
- Applications per team/project
- Sync success rates by team
- Resource utilization per namespace
- RBAC policy violations

### Kubernetes Metrics
- Resource usage by namespace
- Pod failures per team
- Network traffic patterns
- Storage utilization

## Troubleshooting

### Common Issues
1. **Permission Denied**: Check project RBAC configuration
2. **Resource Conflicts**: Verify namespace isolation
3. **Sync Failures**: Check source repository permissions
4. **Policy Violations**: Review Kubernetes RBAC and admission controllers

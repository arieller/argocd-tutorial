# Basic ArgoCD Workflow

## Step 1: Initialize Git Repository

```bash
# Initialize git repo if not already done
cd /home/ariel/argocd-tutorial
git init
git add .
git commit -m "Initial ArgoCD tutorial setup"

# Add your remote repository
git remote add origin https://github.com/your-username/argocd-tutorial.git
git push -u origin main
```

## Step 2: Install ArgoCD

```bash
# Run the setup script
./scripts/setup.sh
```

## Step 3: Access ArgoCD UI

```bash
# Port forward to access the UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Open browser to https://localhost:8080
# Username: admin
# Password: (from setup script output) FXFeRvFluS7VPNXR
```

## Step 4: Deploy Your First Application

### Option A: Using ArgoCD UI
1. Click "New App" in the UI
2. Fill in the details:
   - **Application Name**: sample-app
   - **Project**: default
   - **Repository URL**: your git repository URL
   - **Path**: sample-app
   - **Cluster URL**: https://kubernetes.default.svc
   - **Namespace**: default

### Option B: Using kubectl
```bash
# Update the repository URL in argocd-apps/sample-app.yaml
# Then apply the application
kubectl apply -f argocd-apps/sample-app.yaml
```

## Step 5: Verify Deployment

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# Check deployed resources
kubectl get pods,svc -l app=sample-app

# Check application in ArgoCD UI
```

## Step 6: Test GitOps Workflow

1. **Make a change**: Edit `sample-app/deployment.yaml` (e.g., change replicas to 3)
2. **Commit and push**:
   ```bash
   git add .
   git commit -m "Scale to 3 replicas"
   git push
   ```
3. **Watch ArgoCD sync**: 
   - In UI: Watch the sync status
   - In terminal: `kubectl get pods -l app=sample-app -w`

## Common Commands

```bash
# Check application status
kubectl get app sample-app -n argocd

# Force sync from CLI
kubectl patch app sample-app -n argocd --type='merge' -p='{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD"}}}'

# Check sync status
kubectl describe app sample-app -n argocd
```

## Troubleshooting

- **Application stuck in progressing**: Check pod logs and events
- **Sync fails**: Verify repository access and manifest validity
- **UI not accessible**: Ensure port-forward is running and certificate is accepted

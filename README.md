# rundeck-platform

Terraform creates a local kind cluster and bootstraps Argo CD.
Argo CD then owns everything in `gitops/`.

## Prerequisites
Docker, kind, kubectl, Terraform >= 1.6.

## Run
```bash
cd terraform/cluster
terraform init && terraform apply
export KUBECONFIG=$(terraform output -raw kubeconfig_path)
kubectl get nodes

cd ../bootstrap
terraform init
terraform apply \
  -var kubeconfig_path=$KUBECONFIG \
  -var gitops_repo_url=https://github.com/<you>/rundeck-platform.git

# Argo CD UI
kubectl -n argocd port-forward svc/argocd-server 8080:80
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d
```

## Why two Terraform stacks
The helm provider needs a working kubeconfig at plan time. Splitting
cluster and bootstrap avoids the "provider depends on a resource that
does not exist yet" problem and mirrors real layering (infra -> platform).

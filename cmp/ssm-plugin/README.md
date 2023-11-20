# SSM Plugin

## Testing pod

- Create a pod in the argocd namespace
- Create a `ServiceAccount` with the role `arn:aws:iam::484308071187:role/helm-main-argocd`

```bash
kubectl apply -f - <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-ssm
  namespace: argocd
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::484308071187:role/helm-main-argocd
---
apiVersion: v1
kind: Pod
metadata:
  name: test-ssm
  namespace: argocd
spec:
    serviceAccountName: test-ssm
    containers:
        - name: test-ssm
          image: amazon/aws-cli
          command: ["sleep", "infinity"]
EOF
```

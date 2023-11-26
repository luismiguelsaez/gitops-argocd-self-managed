# SSM Plugin

## Info

- Docker image code: https://github.com/luismiguelsaez/docker-playground/tree/main/argocd/cmp

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
          image: luismiguelsaez/argocd-cmp-default:v0.0.1
          command: ["sleep", "infinity"]
EOF
```

- Get secret value

```bash
aws secretsmanager get-secret-value --secret-id /eks/cluster/main/iam/roles/karpenter
```

- Test Vault plugin ( https://argocd-vault-plugin.readthedocs.io/en/stable )

```bash
curl -sL https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v1.17.0/argocd-vault-plugin_1.17.0_linux_arm64 -o argocd-vault-plugin
chmod +x argocd-vault-plugin
```

```yaml
cat <<EOF > secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: aws-example
stringData:
  sample-secret: <path:/eks/cluster/20231126/main/iam/roles#karpenter#AWSCURRENT>
type: Opaque
EOF
```

```yaml
cat <<EOF > serviceaccount.yaml
apiVersion: v1
automountServiceAccountToken: true
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.con/role-arn: '<path:/eks/cluster/20231126/main/iam/roles#argocd#AWSCURRENT>'
  name: argo-cd-argocd-repo-server
  namespace: argocd
EOF
```

```bash
export AVP_TYPE=awssecretsmanager
cat serviceaccount.yaml | argocd-vault-plugin generate --verbose-sensitive-output -
```

## EKS cluster management

### Upgrade cluster

- Cluster version

```bash
aws --profile dev eks update-cluster-version --name "main" --kubernetes-version "1.28"
aws --profile dev eks list-updates --name "main"
aws --profile dev eks describe-update --name "main" --update-id "94de69c1-9486-4e77-802b-99f56edad178"
```

- Nodegroups

```bash
aws --profile dev eks list-nodegroups --cluster-name main
aws --profile dev eks update-nodegroup-version --cluster-name main --nodegroup-name main-system
```

- Check events

```bash
kubectl events -A -w
```

## ArgoCD management

- Login

```bash
argocd login --grpc-web --name dev --skip-test-tls \
    --username admin \
    --password $(kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n argocd | base64 -d) \
    argocd.dev.lokalise.cloud:443
```

- Get root app status

```bash
argocd context dev
argocd app get root -o json | jq -r '.status.sync.status'
```

- Destroy infra

    - Disable sync
    ```bash
    argocd app patch root --type json --patch='[{"op": "replace", "path": "/spec/syncPolicy", "value": null}]'
    ```

    - Delete root app
    ```bash
    argocd app delete --cascade --propagation-policy foreground root
    ```

    **This will remove the `root` app and all the child applications**

## Kubernetes resources

- Get nodes

```bash
kubectl get nodes -o go-template='
{{ range .items }}
    {{ $node_name := .metadata.name }}
    {{ if .metadata.labels }}
        {{ range $key, $value := .metadata.labels }}
            {{ if or ( eq $key "app" ) ( eq $key "role" ) }}
                {{- printf "%s %s" $node_name $value -}}
            {{ end }}
        {{ end }}
    {{ end }}
{{ end }}'
```

## Cluster cleanup

### Karpenter nodes

After cluster removal, there could be some Karpenter nodes still running, so we would need to remove them

```bash
IDS=($(aws --profile dev ec2 describe-instances --filter "Name=tag:karpenter.sh/managed-by,Values=main" --query 'Reservations[].Instances[].[InstanceId]' --output text | tr '\n' ' '))

for ID in ${IDS[@]}; do echo "Deleting instance $ID"; aws --profile dev ec2 terminate-instances --instance-ids "$ID"; done
```

### Load balancers

```bash
aws --profile dev elbv2 describe-load-balancers --filter "Name=tag:elbv2.k8s.aws/cluster,Values=main"

TG_ARNS=$(aws --profile dev elbv2 describe-target-groups | jq -r '.TargetGroups[]|select(.LoadBalancerArns|length == 0)|.TargetGroupArn')
for ARN in ${TG_ARNS[@]}; do echo "Deleting TG $ID"; aws --profile dev elbv2 delete-target-group --target-group-arn "$ARN"; done
```

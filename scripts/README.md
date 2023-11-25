
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

## Kubernetes resources

- Get nodes

```bash
kubectl get nodes -o go-template='
{{ range .items }}
    {{ $node_name := .metadata.name }}
    {{ if .metadata.labels }}
        {{ range $key, $value := .metadata.labels }}
            {{ if or ( eq $key "app" ) ( eq $key "role" ) }}
                {{ printf "%s %s" $node_name $value }}
            {{ end }}
        {{ end }}
    {{ end }}
{{ end }}'
```

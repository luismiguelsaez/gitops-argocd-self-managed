
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

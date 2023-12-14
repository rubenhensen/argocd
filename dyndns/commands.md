## Change config
- `kubectl create configmap go-transip-dyndns-configmap --from-file=dyndns/go-transip-dyndns.toml --dry-run=client -o yaml > dyndns/dyndns-configmap.yml`

## Ssh to sealed secret
- `kubectl create secret generic transip-key --from-file=dyndns/transip.key -o yaml --dry-run=client > dyndns/transip-secret.key`  
- `kubeseal -f ./dyndns/transip-secret.key -w ./dyndns/transip-key-sealed.yml`
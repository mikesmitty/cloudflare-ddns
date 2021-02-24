# Cloudflare DDNS
A kubernetes cronjob that runs a docker-container every 5 minutes to check & update your external ip.

## Execute kubernetes cronjob
```
# 1. Clone the repo
git clone https://github.com/mirioeggmann/cloudflare-ddns.git

# 2. Navigate into it
cd cloudflare-ddns

# 3. Edit the k8s-cronjob.yaml according to "Values for k8s-cronjob.yaml"

# 4. Execute the cronjob
kubectl apply -f k8s-cronjob.yaml
```

## Add values for k8s-cronjob.yaml to vault

- Get the ZONE_ID under dash.cloudflare.com -> Your domain -> Overview -> API -> Zone ID
- Get the RECORD_ID: https://api.cloudflare.com/#dns-records-for-a-zone-list-dns-records
- Get your AUTH_KEY (Create a key with "DNS Edit" permissions): https://support.cloudflare.com/hc/en-us/articles/200167836-Where-do-I-find-my-Cloudflare-API-key-
- Specify your DNS record

Assuming your kv store is mounted at secret/
```
#example
vault kv put secret/cloudflare-ddns \
    ZONE_ID=023e105f4ecef8ad9ca31a8372d0c353 \
    RECORD_ID=372e67954025e0ba6aaa6d586b9e0b59 \
    AUTH_KEY=c2547eb745049flc9320b638f5e225cf483cc5cfdda41 \
    NAME=example.com
```
## Create and apply an app access policy for the cronjob to read the secret
```
cat <<EOF > app-policy.hcl
path "secret*" {
  capabilities = ["read"]
}
EOF

vault policy write app app-policy.hcl
```
## Add role for the CronJob to access the secret
```
#example
vault write auth/kubernetes/role/cloudflare-ddns \
    bound_service_account_names=default \
    bound_service_account_namespaces=<NAMESPACE> \
    policies=app \
    ttl=1m
```

For more Cloudflare API information read the [Cloudflare API documentation v4](https://api.cloudflare.com/)

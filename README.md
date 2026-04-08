# traefik-wildcard-test

Configuration for testing wildcard certificate generation

## load environment variables

```shell
export KUBECONFIG=$(pwd)/.kubeconfig
export CLUSTERNAME=""
export DOMAINNAME=""
export HUB_TOKEN=""
export GANDIV5_PERSONAL_ACCESS_TOKEN=
```

or

```shell
source .env
```

## Deploy Traefik Hub

### Create Traefik Hub namespace

```shell
kubectl create ns traefik
```

### Set Traefik Hub token

```shell
kubectl create secret generic hub-license --from-literal=token="${HUB_TOKEN}" -n traefik
```

### Set DNS Token for LE DNS Challenge

```shell
kubectl create secret generic dnsapikey --namespace traefik --from-literal=token=${GANDIV5_PERSONAL_ACCESS_TOKEN}
```

```shell
helm upgrade --install traefik traefik/traefik -n traefik --values hub/values.yaml
```

### Add CName record

```shell
ADDRECORD='{
  "rrset_type": "CNAME",
  "rrset_name": "*.'$CLUSTERNAME'",
  "rrset_ttl": "1800",
  "rrset_values": [
    "'$(kubectl get svc/traefik -n traefik --no-headers | awk {'print $4'})'."
  ]
}'
curl -s -X POST -d $ADDRECORD \
  -H "Authorization: Bearer $GANDIV5_PERSONAL_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.gandi.net/v5/livedns/domains/$DOMAINNAME/records
```

### Deploy dashboard

```shell
envsubst < hub/dashboard.yaml | kubectl apply -f -
```

## Deploy app

```shell
envsubst < whoami/whoami.yaml | kubectl apply -f -
```

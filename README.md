# aws-redis-simple

A very simple container that serves 2 purposes:

* demonstrate golang as an aws elasticache client
* validate elasticache secure config / connectivity

When deployed as a pod for environment validation, the /liveness and /readiness endpoints attempt to set a key with the current timestamp, then get the key to make sure it was written.

# tls

TLS is available in aws and uses certs created by a public AWS CA (unlike RDS which is a special non-public bundle) - so public bundles from any distro should work.  The [Dockerfile](./Dockerfile) in this repo uses certs from `alpine:latest`.

**TLS on mac >= 12.x** - the AWS certs do not comply with SCT verification - this requires TLS verification be disabled (the connection is still TLS, just without cert verification).  A darwin check has been hard-coded into this example [here](https://github.com/bensolo-io/aws-redis-simple/blob/7a3e33dbf4df8436342961c21544c1a12e155967/main.go#L51-L56).

# requirements

An elasticache redis instance with:

* in clustered mode (required to enable redis auth in aws)
* auth token configured
* tls in transist
* network connectivity from client pod (security groups, subnet placement, etc.)

TODO - link to terraform examples repo

# usage

Create a secret containing elasticache redis endopint and auth token:

```bash
kubectl apply --context $REMOTE_CONTEXT1 -f- <<EOF
kind: Secret
apiVersion: v1
metadata:
  name: redis-config
  namespace: default
stringData:
  token: ${REDIS_PASSWORD}
  address: ${REDIS_ADDR}
EOF
```
Deploy as a pod (attempts to set and get a key each time readiness or liveness is requested; pod will not be healthy if anything fails):

```bash
kubectl apply -k ./deploy/kustomize --context $REMOTE_CONTEXT1
```
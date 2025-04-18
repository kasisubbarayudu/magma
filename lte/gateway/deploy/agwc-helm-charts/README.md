# AGW Helm Deployment - Experimental

This folder contains a Helm Chart for the containerized AGW.

Currently working on arm64 kvm-qemu VM host.

## Configuration

The following table list the configurable parameters of the agw chart and their default values.

| Parameter            | Description                            | Default              |
| -------------------- | -------------------------------------- | -------------------- |
| `secret.certs`       | Secret name containing agwc certs.     | `agwc-secrets-certs` |
| `image.repository`   | Repository for agwc images.            | ``                   |
| `image.pullPolicy`   | Pull policy for agwc images.           | ``                   |
| `image.tag`          | Tag for agwc image.                    | `latest`             |
| `image.arch`         | Tag for node architecture (e.g., arm). | ``                   |
| `config.domainName`  | Orchestrator domain name.              | ``                   |
| `persistant.name`    | Secret name containing agwc certs.     | `agwc-claim`         |
| `config.gwChallenge` | Challenge key.                         | ``                   |

## Prerequisites

- A Kubernetes cluster with the following components(For private cluster):
    - MetalLB
    - OpenEBS
- `rootCA.pem` certificate file.
- `snowflake` file containing uuid.

## Generate `snowflake` file

```bash
sudo uuidgen > /etc/snowflake
```

## Generate new Challenge key

```bash
# Generate Private key
sudo openssl ecparam -name secp384r1 -genkey -noout -out gw_challenge.key
sudo chmod 644 gw_challenge.key

# Generate Public key
openssl ec -in gw_challenge.key -pubout -out gw_challenge.pem
GW_CHALLENGE=$(cat gw_challenge.pem | sed '5d' | sed '1d' | tr -d '\n')
echo ${GW_CHALLENGE}
```
Copy this challenge key and paste in `config.gwChallenge` field.

For `image.arch`, use `arm` or leave blank for x86

## Installation

# Create magma namespace

```sh
~ kubectl create namespace magma
```

# Create rootca certificate secret needed to communicate with orc8r

```sh
~ kubectl create secret generic agwc-secret-certs --from-file=rootCA.pem --namespace magma
```

# Deploy an AGW after updating values.yaml

```sh
~ cd lte/gateway/deploy/agwc-helm-charts
~ kubectl apply -f templates/agwc-pv.yaml
~ helm --debug install agwc --namespace magma . --values=values.yaml
```

# Delete the AGW if needed using

```sh
~ helm uninstall agwc --namespace magma
~ kubectl delete -n magma secret agwc-secret-certs
~ kubectl delete namespace magma
```

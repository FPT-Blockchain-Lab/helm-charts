## Helm Charts Repo Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

  helm repo add fpt-blc-lab https://fpt-blockchain-lab.github.io/helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
fpt-blc-lab` to see the charts.

To install the <chart-name> chart:

    helm install my-<chart-name> fpt-blc-lab/<chart-name>

To uninstall the chart:

    helm delete my-<chart-name>

# Helm Charts

Each helm chart that you can use has the following keys and you need to set them. The `cluster.provider` is used as a key for the various cloud features enabled. Also you only need to specify one cloud provider, **not** both if deploying to cloud. As of writing this doc, AWS and Azure are fully supported.

```bash
# dict with what features and the env you're deploying to
cluster:
  provider: local  # choose from: local | aws | azure
  cloudNativeServices: false # set to true to use Cloud Native Services (SecretsManager and IAM for AWS; KeyVault & Managed Identities for Azure)

aws:
  # the aws cli commands uses the name 'quorum-node-secrets-sa' so only change this if you altered the name
  serviceAccountName: quorum-node-secrets-sa
  # the region you are deploying to
  region: ap-southeast-2

azure:
  # the script/bootstrap.sh uses the name 'quorum-pod-identity' so only change this if you altered the name
  identityName: quorum-pod-identity
  # the clientId of the user assigned managed identity created in the template
  identityClientId: azure-clientId
  keyvaultName: azure-keyvault
  # the tenant ID of the key vault
  tenantId: azure-tenantId
  # the subscription ID to use - this needs to be set explictly when using multi tenancy
  subscriptionId: azure-subscriptionId

```

Setting the `cluster.cloudNativeServices: true` will:

- Keys are stored in KeyVault or Secrets Manager
- We make use of Managed Identities or IAMs for access

You are encouraged to pull these charts apart and experiment with options to learn how things work.

## Local Development:

Minikube defaults to 2 CPU's and 2GB of memory, unless configured otherwise. We recommend you starting with at least 16GB, depending on the amount of nodes you are spinning up - the recommended requirements for each besu node are 4GB

```bash
minikube start --memory 16384 --cpus 2
# or with RBAC
minikube start --memory 16384 --cpus 2 --extra-config=apiserver.Authorization.Mode=RBAC

# enable the ingress
minikube addons enable ingress

# optionally start the dashboard
minikube dashboard &
```

Verify kubectl is connected to Minikube with: (please use the latest version of kubectl)

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:18:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

### Production with k3s with single master

***NOTE:*** For the detail, please check the quick start [link](https://rancher.com/docs/k3s/latest/en/quick-start/)
On the master/control plane node, run

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC='--disable="traefik,local-path"' sh -
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config
# After finish find the token
sudo cat /var/lib/rancher/k3s/server/node-token
```

On the worker node, run

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
# with myserver is the IP/Domain address of the master node, mynodetoken is the /var/lib/rancher/k3s/server/node-token value
```

## Usage
### _Install longhorn distributed block storage:_

Firstly, running check requirements script for the cluster

```bash
curl -sfL https://raw.githubusercontent.com/longhorn/longhorn/v1.3.0/scripts/environment_check.sh | bash -
```

After the checking succesfully executed, please follow this guide [link](https://longhorn.io/docs/1.3.0/deploy/install/install-with-helm/). For the node dependencies requirements, please check this [link](https://longhorn.io/docs/1.3.0/deploy/install/#installation-requirements)

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm upgrade --install longhorn longhorn/longhorn --version 1.3.0 --namespace admin --create-namespace --values ./values/longhorn.yml 
# To check the deployment succeeded, run
kubectl -n admin get pod -w
```

### _Spin up prometheus-stack for metrics and loki for logs: (Optional but recommended)_

Firstly, the loki values has `persistence.enabled: true`, to enable persistent logs.
Secondly, uses charts from prometheus-community. Grafana by default already disable in `./values/prometheus-stack.yml`. Please configure this as per your requirements and policies. 

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install loki-stack grafana/loki-stack \
    --version 2.6.5 \
    --namespace=monitoring --create-namespace \
    --values ./values/loki-stack.yml \
    --set loki.fullnameOverride=loki,logstash.fullnameOverride=logstash-loki

# NOTE: please refer to values/prometheus-stack.yml to configure the alerts per your requirements ie slack, email etc
helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack --version 34.10.0 --namespace=monitoring --create-namespace --values ./values/prometheus-stack.yml --atomic --debug > manifest-prometheus-stack.logs
```

Install grafana support both Prometheus and Loki

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
kubectl create secret generic admin-auth --from-literal=admin-user="${ADMIN_USER}" --from-literal=admin-password="${ADMIN_PASSWORD}" -n monitoring 
helm upgrade --install grafana grafana/grafana --version 6.32.7 --namespace=monitoring --create-namespace --values ./values/grafana.yml --atomic --debug

### To get admin user password
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Install postgres cluster (not recommend, please use the hosted services)

```bash
helm repo add postgresql bitnami/postgresql
helm repo update
helm install postgresql bitnami/postgresql --version 11.6.18 --namespace=quorum --create-namespace --values ./values/postgresql.yml --atomic --debug
```

```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace quorum postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo $POSTGRES_PASSWORD
kubectl exec -it postgresql-0  -n quorum -- /opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash
PGPASSWORD="$POSTGRES_PASSWORD" --host postgresql -U postgres -d postgres -p 5432
```

```sql
CREATE DATABASE blockscout;
CREATE USER blockscout WITH ENCRYPTED PASSWORD 'blockscout';
GRANT ALL PRIVILEGES ON DATABASE blockscout TO blockscout;

CREATE DATABASE subgraph;
CREATE USER subgraph WITH ENCRYPTED PASSWORD 'subgraph';
GRANT ALL PRIVILEGES ON DATABASE subgraph TO subgraph;
```

### Blockchain Explorer

#### Blockscout

**Notes** Blockscout use subchart of postgresql. The default value of subchart is configured at helm/charts/blockscout/values.yaml postgresql

```bash
helm dependency update ./charts/blockscout

# For GoQuorum
helm upgrade --install blockscout ./charts/blockscout --namespace quorum --values ./values/blockscout-goquorum.yml

# For Besu
helm install blockscout ./charts/blockscout --namespace quorum --values ./values/blockscout-besu.yml
``` 

#### Quorum Explorer

You may optionally deploy our lightweight Quorum Explorer, which is compatible for both Besu and GoQuorum. The Explorer can give an overview over the whole network, such as querying each node on the network for block information, voting or removing validators from the network, demonstrating a SimpleStorage smart contract with privacy enabled, and sending transactions between wallets in one interface.

**Note:** It will be necessary to update the `quorum-explorer-config` configmap after deployment (if you do not choose to modify the `explorer-besu.yaml` or `explorer-goquorum.yaml` files before deploying) to provide the application endpoints to the nodes on the network. You may choose to either use internal k8s DNS or through ingress (your preference and needs). Please see the `values/explorer-besu.yaml` or `values/explorer-goquorum.yaml` to see some examples.

**Going into Production**

If you would like to use the Quorum Explorer in a production environment, it is highly recommended to enable OAuth or, at the minimum, local username and password authentication which is natively supported by the Explorer. The `explorerEnvConfig` section of the `explorer-besu.yaml` and `explorer-goquorum.yaml` files contain the variables that you may change. By default `DISABE_AUTH` is set to `true`, which means authentication is disabled. Change this to `false` if you would like to enable authentication. If this is set to `false`, you must also provide at least one authentication OAuth method by filling the variables below (supports all of those listed in the file).

You may find out more about the variables [here](https://github.com/ConsenSys/quorum-explorer#going-into-production).

```
To deploy for Besu:

```bash
helm upgrade --install quorum-explorer ./charts/explorer --namespace quorum --create-namespace --values ./values/explorer-besu.yaml --atomic
```

To deploy for GoQuorum:
```bash
helm upgrade --install quorum-explorer ./charts/explorer --namespace quorum --create-namespace --values ./values/explorer-goquorum.yaml --atomic

After modifying configmap with node details, you will need to restart the pod to get the config changes. Deleting the existing pod will force the deployment to recreate it:

```bash
kubectl delete pod <quorum-explorer-pod-name>
```

### _For Besu:_

```bash
helm install genesis ./charts/besu-genesis --namespace quorum --create-namespace --values ./values/genesis-besu.yml

# bootnodes - optional but recommended
helm install bootnode-1 ./charts/besu-node --namespace quorum --values ./values/bootnode.yml
helm install bootnode-2 ./charts/besu-node --namespace quorum --values ./values/bootnode.yml

# !! IMPORTANT !! - If you use bootnodes, please set `quorumFlags.usesBootnodes: true` in the override yaml files
# for validator.yml, txnode.yml, reader.yml
helm install validator-1 ./charts/besu-node --namespace quorum --values ./values/validator.yml
helm install validator-2 ./charts/besu-node --namespace quorum --values ./values/validator.yml
helm install validator-3 ./charts/besu-node --namespace quorum --values ./values/validator.yml
helm install validator-4 ./charts/besu-node --namespace quorum --values ./values/validator.yml

# spin up a besu and tessera node pair
helm install member-1 ./charts/besu-node --namespace quorum --values ./values/txnode.yml

# spin up a quorum rpc node
helm install rpc-1 ./charts/besu-node --namespace quorum --values ./values/reader.yml
```

### _For GoQuorum:_

Create storageClass with name `quorum-node-storage` and namespace `quorum`. This storage class here use longhorn.
```
cat <<EOF | kubectl apply -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: quorum-node-storage
  namespace: quorum
provisioner: driver.longhorn.io
reclaimPolicy: "Delete"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
parameters:
  numberOfReplicas: "1"
  dataLocality: "best-effort"
  staleReplicaTimeout: "2880" # 48 hours in minutes
  fromBackup: ""
  #  diskSelector: "ssd,fast"
  #  nodeSelector: "storage,fast"
  #  recurringJobSelector: '[
  #   {
  #     "name":"snap",
  #     "isGroup":true,
  #   },
  #   {
  #     "name":"backup",
  #     "isGroup":false,
  #   }
  #  ]'
EOF
```

```bash
### Replace value name with the desired environments
helm install genesis ./charts/goquorum-genesis --namespace quorum --create-namespace --values ./values/genesis-goquorum.test.yml --wait-for-jobs

helm upgrade --install validator-1 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-2 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-3 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-4 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-5 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-6 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml & \
helm upgrade --install validator-7 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml
```

### spin up a quorum and tessera node pair (optional)
```bash
helm install member-1 ./charts/goquorum-node --namespace quorum --values ./values/txnode.yml
```

### spin up a quorum rpc node

```bash
helm upgrade --install rpc-1 ./charts/goquorum-node --namespace quorum --values ./values/reader.yml --atomic
```

### Deploy enhanced permission contract

Generating config
```Bash
helm install enhanced-permission ./charts/goquorum-enhanced-permission --namespace quorum --values ./values/enhanced-permission.yml --wait-for-jobs --atomic --timeout 30s
```

Edit `./values/validator.yml`. To turn on copy mounted volumes files into quorum data files. 
```yaml
---

quorumFlags:
  ...
  permissioned: true
  enhancedPermissioned: true
  ...
```

Take turn to validator apply new config to prevent network going down.

```bash
helm upgrade --install validator-1 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-2 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-3 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-4 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-5 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-6 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
helm upgrade --install validator-7 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml --atomic
```

### Add New ORG

Deploy new node
```
helm install new-org-1 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml
```

Get node details
```
kubectl get secret goquorum-node-new-org-1-keys -nquorum -ojson | jq -r ".data[\"nodekey.pub\"]" | base64 -d
kubectl get secret goquorum-node-new-org-1-keys -nquorum -ojson | jq -r ".data[\"accountAdddress\"]" | base64 -d
```

Create new org with network admin
https://consensys.net/docs/goquorum/en/latest/reference/api-methods/#quorumpermission_addorg

Approve org with network admin
https://consensys.net/docs/goquorum/en/latest/reference/api-methods/#quorumpermission_approveorg


### External Validator

#### FPT Blockchain Lab cluster

NodePort all validator services to connect from external cluster

Export 4 configmap

```
goquorum-genesis
goquorum-enhanced-permission-config
goquorum-peers (with cluster public ip & port from NodePort)
goquorum-networkid
```

Add new external node to permission contract

```
quorumPermission.addNode("ADMINORG", "enode://nodekey@nodeip:nodeport?discport=0", {from: eth.accounts[0]})
```

#### External Validator cluster

Import 4 configmaps

```
goquorum-genesis
goquorum-enhanced-permission-config
goquorum-peers
goquorum-networkid
```

helm install external-validator-1 ./charts/goquorum-node --namespace quorum --values ./values/goquorum-external-validator.yml

### _Using Cert Manager for Ingress: (Optional but recommnended)_

***NOTE:** only necessary if ingress used
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.8.2 \
  --set installCRDs=true
```

### Ingress

***NOTES*** for host based approach, please set DNS A record to the Cluster IP Address

Optionally deploy the ingress controller for the network and nodes like so:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress ingress-nginx/ingress-nginx \
  --namespace admin --create-namespace \
  --version 4.2.0 \
  --values ./values/ingress-nginx.yml
```

Once complete, view the IP address listed under the `Ingress` section if you're using the Kubernetes Dashboard
or on the command line `kubectl get services -A`.

Deploy RPC ingress
```
kubectl apply -f ../ingress/ingress-rules-goquorum.yml
```

Those three command deploy the IngressClass at cluster scope, and 3 Ingress for the given namespace. This is required because TLS sercret must be avaiable in the same namespace. And, remember to view the host for the ingress with `kubectl get ingress -A`

You can then access Grafana on: 
```bash
# For Besu's grafana address:
http://<INGRESS_HOST>/d/XE4V0WGZz/besu-overview?orgId=1&refresh=10s

# For GoQuorum's grafana address:
http://<INGRESS_HOST>/d/a1lVy7ycin9Yv/goquorum-overview?orgId=1&refresh=10s
```

You can access Kibana on:
```bash
http://<INGRESS_HOST>/kibana
```

If you've deployed the Ingress from the previous step, you can access the Quorum Explorer on:

```bash
http://<INGRESS_HOST>/explorer
```

### Once deployed, services are available as follows on the IP/ of the ingress controllers:

API Calls to either client

```bash

# HTTP RPC API:
curl -v -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://<INGRESS_HOST>/rpc

# which should return (confirming that the node running the JSON-RPC service is syncing):
{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : "0x4e9"
}

# HTTP GRAPHQL API:
curl -X POST -H "Content-Type: application/json" --data '{ "query": "{syncing{startingBlock currentBlock highestBlock}}"}' http://<INGRESS_HOST>/graphql/
# which should return
{
  "data" : {
    "syncing" : null
  }
}
```

### LC protocol

```
helm upgrade --install lc ./charts/lc --namespace quorum --atomic --debug --dry-run
```
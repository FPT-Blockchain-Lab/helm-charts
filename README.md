## **Helm Charts Repo Usage**

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

## **Helm Charts**

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

### Staging with k3s with single master/control plane

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

### Production

It's is recommend to run master/control plane with at least 3 nodes to provide high availability

### Blockchain Explorer

#### Blockscout

**Notes** Blockscout use subchart of postgresql. The default value of subchart is configured at helm/charts/blockscout/values.yaml postgresql

```bash
helm dependency update ./charts/blockscout

# For GoQuorum
helm upgrade --install blockscout ./charts/blockscout --namespace quorum --values ./values/blockscout-goquorum.yml
``` 

#### Quorum Explorer

You may optionally deploy our lightweight Quorum Explorer, which is compatible for both Besu and GoQuorum. The Explorer can give an overview over the whole network, such as querying each node on the network for block information, voting or removing validators from the network, demonstrating a SimpleStorage smart contract with privacy enabled, and sending transactions between wallets in one interface.

**Note:** It will be necessary to update the `quorum-explorer-config` configmap after deployment (if you do not choose to modify the `explorer-besu.yaml` or `explorer-goquorum.yaml` files before deploying) to provide the application endpoints to the nodes on the network. You may choose to either use internal k8s DNS or through ingress (your preference and needs). Please see the `values/explorer-besu.yaml` or `values/explorer-goquorum.yaml` to see some examples.

**Going into Production**

If you would like to use the Quorum Explorer in a production environment, it is highly recommended to enable OAuth or, at the minimum, local username and password authentication which is natively supported by the Explorer. The `explorerEnvConfig` section of the `explorer-besu.yaml` and `explorer-goquorum.yaml` files contain the variables that you may change. By default `DISABE_AUTH` is set to `true`, which means authentication is disabled. Change this to `false` if you would like to enable authentication. If this is set to `false`, you must also provide at least one authentication OAuth method by filling the variables below (supports all of those listed in the file).

You may find out more about the variables [here](https://github.com/ConsenSys/quorum-explorer#going-into-production).

To deploy for GoQuorum:
```bash
helm upgrade --install quorum-explorer ./charts/explorer --namespace quorum --create-namespace --values ./values/explorer-goquorum.yaml --atomic

After modifying configmap with node details, you will need to restart the pod to get the config changes. Deleting the existing pod will force the deployment to recreate it:

```bash
kubectl delete pod <quorum-explorer-pod-name>
```

### Deploy validators:

```bash
### Replace value name with the desired environments
helm install genesis fpt-blc-lab/goquorum-genesis --namespace quorum --create-namespace --values ./values/genesis-goquorum.test.yml --wait-for-jobs

helm upgrade --install validator-1 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31101 & \
helm upgrade --install validator-2 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31102 & \
helm upgrade --install validator-3 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31103 & \
helm upgrade --install validator-4 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31104 & \
helm upgrade --install validator-5 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31105 & \
helm upgrade --install validator-6 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31106 & \
helm upgrade --install validator-7 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set node.goquorum.p2p.nodePort=31107
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
helm install enhanced-permission fpt-blc-lab/goquorum-enhanced-permission --namespace quorum --values ./values/enhanced-permission.yml --wait-for-jobs
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
helm upgrade --install validator-1 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-2 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-3 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-4 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-5 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-6 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
helm upgrade --install validator-7 fpt-blc-lab/goquorum-node --namespace quorum --values ./values/goquorum-validator.yml --set quorumFlags.permissioned=true --set quorumFlags.enhancedPermissioned=true
```

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

For testnet:
```
# goquorum-enhanced-permission-config
kubectl create configmap goquorum-enhanced-permission-config --from-file values/testnet/goquorum-enhanced-permission-config.yml -o yaml --dry-run | kubectl apply -f -
# goquorum-genesis
kubectl create configmap goquorum-genesis --from-file values/testnet/goquorum-genesis.yml -o yaml --dry-run | kubectl apply -f -
# goquorum-networki
kubectl create configmap goquorum-networkid --from-file values/testnet/goquorum-networkid.yml -o yaml --dry-run | kubectl apply -f -
# goquorum-peers
kubectl create configmap goquorum-peers --from-file values/testnet/goquorum-peers.yml -o yaml --dry-run | kubectl apply -f -
```

For mainnet: (VPN setup requirement)
t
Then create validator char
```
helm install external-validator-1 ./charts/goquorum-node --namespace quorum --values ./values/goquorum-external-validator.yml
```

### LC protocol

```
helm upgrade --install lc ./charts/lc --namespace quorum --atomic --debug --dry-run
```

### LC graph

```
helm upgrade --install lc-graph ./charts/lc-graph --namespace quorum --atomic --debug --dry-run
```
## Quorum Permission operations

### Add new ORG

- Install org node
```
helm upgrade --install new-org-1 ./charts/goquorum-node  --namespace quorum --values ./values/validator.yml --set storage.storageClass=longhorn --set node.goquorum.serviceType=NodePort
``` 
- Get node details
```
kubectl get secret goquorum-node-new-org-1-keys -nquorum -ojson | jq -r ".data[\"nodekey.pub\"]" | base64 -d
kubectl get secret goquorum-node-new-org-1-keys -nquorum -ojson | jq -r ".data[\"accountAdddress\"]" | base64 -d
```

- Propose addNode, access to validator have network admin account
```
kubectl exec -it goquorum-node-validator-1-0 -nquorum -- geth attach /data/quorum/geth.ipc
```
```
quorumPermission.addOrg(<orgId>,<enodeId>,<accountId>,{from: eth.accounts[0]})
```
Ref: https://consensys.net/docs/goquorum/en/latest/reference/api-methods/#quorumpermission_addorg
- approveOrg from > 50% network admins
```
quorumPermission.approveOrg(<orgId>,<enodeId>,<accountId>,{from: eth.accounts[0]})
```
Ref: https://consensys.net/docs/goquorum/en/latest/reference/api-methods/#quorumpermission_approveorg

0xf8b9a00000000000000000000000000000000000000000000000000000000000000000f89394386f3dbcdc2c52d67e1f3827e408ff26af33e95
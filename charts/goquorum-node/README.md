# goquorum-node

![Version: 0.2.16](https://img.shields.io/badge/Version-0.2.16-informational?style=flat-square) ![AppVersion: v22.7.4-fpt.blockchain.lab-0.5.0](https://img.shields.io/badge/AppVersion-v22.7.4--fpt.blockchain.lab--0.5.0-informational?style=flat-square)

Quorum node for a POA network using IBFT for consensys

**Homepage:** <https://github.com/FPT-Blockchain-Lab>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Truong Ma Phi | <trmaphi@gmail.com> |  |

## Source Code

* <https://github.com/FPT-Blockchain-Lab/quorum>

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| aws.region | string | `"ap-southeast-2"` |  |
| aws.serviceAccountName | string | `"quorum-node-secrets-sa"` |  |
| azure.identityClientId | string | `"azure-clientId"` |  |
| azure.identityName | string | `"quorum-pod-identity"` |  |
| azure.keyvaultName | string | `"azure-keyvault"` |  |
| azure.subscriptionId | string | `"azure-subscriptionId"` |  |
| azure.tenantId | string | `"azure-tenantId"` |  |
| cluster.cloudNativeServices | bool | `false` |  |
| cluster.provider | string | `"local"` |  |
| cluster.reclaimPolicy | string | `"Delete"` |  |
| image.goquorum.repository | string | `"fptblockchainlabbot/quorum"` |  |
| image.goquorum.tag | string | `"v23.4.0-fpt.blockchain.lab-0.5.4"` |  |
| image.hooks.pullPolicy | string | `"IfNotPresent"` |  |
| image.hooks.repository | string | `"consensys/quorum-k8s-hooks"` |  |
| image.hooks.tag | string | `"qgt-0.2.3"` |  |
| image.tessera.repository | string | `"quorumengineering/tessera"` |  |
| image.tessera.tag | string | `"23.4.0"` |  |
| node.goquorum.account.password | string | `"password"` |  |
| node.goquorum.account.passwordPath | string | `"/keys/accountPassword"` |  |
| node.goquorum.account.unlock | int | `0` |  |
| node.goquorum.dataPath | string | `"/data/quorum"` |  |
| node.goquorum.genesisFilePath | string | `"/etc/genesis/genesis.json"` |  |
| node.goquorum.graphql.addr | string | `"0.0.0.0"` |  |
| node.goquorum.graphql.corsDomain | string | `"*"` |  |
| node.goquorum.graphql.enabled | bool | `true` |  |
| node.goquorum.graphql.port | int | `8547` |  |
| node.goquorum.graphql.vHosts | string | `"*"` |  |
| node.goquorum.keysPath | string | `"/keys"` |  |
| node.goquorum.log.verbosity | int | `5` |  |
| node.goquorum.metrics.enabled | bool | `true` |  |
| node.goquorum.metrics.pprofaddr | string | `"0.0.0.0"` |  |
| node.goquorum.metrics.pprofport | int | `9545` |  |
| node.goquorum.metrics.serviceMonitorEnabled | bool | `false` |  |
| node.goquorum.metrics.serviceMonitorRelease | string | `"rancher-monitoring"` |  |
| node.goquorum.miner.blockPeriod | int | `5` |  |
| node.goquorum.miner.threads | int | `1` |  |
| node.goquorum.network.extIp | string | `"0.0.0.0"` |  |
| node.goquorum.p2p.addr | string | `"0.0.0.0"` |  |
| node.goquorum.p2p.enabled | bool | `true` |  |
| node.goquorum.p2p.port | int | `30303` |  |
| node.goquorum.privacy.pubkeyFile | string | `"/tessera/tm.pub"` |  |
| node.goquorum.privacy.pubkeysPath | string | `"/tessera"` |  |
| node.goquorum.privacy.url | string | `"http://localhost:9101"` |  |
| node.goquorum.qlightServer.addr | string | `"0.0.0.0"` |  |
| node.goquorum.qlightServer.enabled | bool | `false` |  |
| node.goquorum.qlightServer.port | int | `30303` |  |
| node.goquorum.replicaCount | int | `1` |  |
| node.goquorum.resources.cpuLimit | int | `1` |  |
| node.goquorum.resources.cpuRequest | float | `0.1` |  |
| node.goquorum.resources.memLimit | string | `"2G"` |  |
| node.goquorum.resources.memRequest | string | `"1G"` |  |
| node.goquorum.rpc.addr | string | `"0.0.0.0"` |  |
| node.goquorum.rpc.api | string | `"admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul"` |  |
| node.goquorum.rpc.authenticationEnabled | bool | `false` |  |
| node.goquorum.rpc.corsDomain | string | `"*"` |  |
| node.goquorum.rpc.enabled | bool | `true` |  |
| node.goquorum.rpc.port | int | `8545` |  |
| node.goquorum.rpc.vHosts | string | `"*"` |  |
| node.goquorum.serviceType | string | `"ClusterIP"` |  |
| node.goquorum.ws.addr | string | `"0.0.0.0"` |  |
| node.goquorum.ws.api | string | `"admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul"` |  |
| node.goquorum.ws.authenticationEnabled | bool | `false` |  |
| node.goquorum.ws.enabled | bool | `true` |  |
| node.goquorum.ws.origins | string | `"*"` |  |
| node.goquorum.ws.port | int | `8546` |  |
| node.tessera.dataPath | string | `"/data/tessera"` |  |
| node.tessera.keysPath | string | `"/keys"` |  |
| node.tessera.password | string | `"password"` |  |
| node.tessera.passwordPath | string | `"/keys/tm.password"` |  |
| node.tessera.port | int | `9000` |  |
| node.tessera.q2tport | int | `9101` |  |
| node.tessera.resources.cpuLimit | float | `0.7` |  |
| node.tessera.resources.cpuRequest | float | `0.5` |  |
| node.tessera.resources.memLimit | string | `"2G"` |  |
| node.tessera.resources.memRequest | string | `"1G"` |  |
| node.tessera.tmkey | string | `""` |  |
| node.tessera.tmpub | string | `""` |  |
| node.tessera.tpport | int | `9080` |  |
| quorumFlags.enhancedPermissioned | bool | `false` |  |
| quorumFlags.permissioned | bool | `false` |  |
| quorumFlags.privacy | bool | `false` |  |
| quorumFlags.removeKeysOnDelete | bool | `false` |  |
| quorumFlags.updatePeersConfigMap | bool | `false` |  |
| serviceAccount.autoGenerate | bool | `true` |  |
| storage.pvcSizeLimit | string | `"5Gi"` |  |
| storage.reclaimPolicy | string | `"Delete"` |  |
| storage.sizeLimit | string | `"5Gi"` |  |
| storage.storageClass | string | `nil` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.0](https://github.com/norwoodj/helm-docs/releases/v1.11.0)

# goquorum-genesis

![Version: 0.1.4](https://img.shields.io/badge/Version-0.1.4-informational?style=flat-square) ![AppVersion: latest](https://img.shields.io/badge/AppVersion-latest-informational?style=flat-square)

Quorum Genesis generator with Helm chart in Kubernetes

**Homepage:** <https://github.com/FPT-Blockchain-Lab>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Truong Ma Phi | <trmaphi@gmail.com> |  |

## Source Code

* <https://github.com/FPT-Blockchain-Lab/helm-charts/charts/goquorum-genesis>

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
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"consensys/quorum-k8s-hooks"` |  |
| image.tag | string | `"qgt-0.2.3"` |  |
| quorumFlags.nodePublicIp | string | `nil` |  |
| quorumFlags.onlyGenerateKeys | bool | `false` |  |
| quorumFlags.removeGenesisOnDelete | bool | `true` |  |
| rawGenesisConfig.blockchain.accountPassword | string | `"password"` |  |
| rawGenesisConfig.blockchain.nodes.count | int | `4` |  |
| rawGenesisConfig.blockchain.nodes.generate | bool | `true` |  |
| rawGenesisConfig.genesis.coinbase | string | `"0x0000000000000000000000000000000000000000"` |  |
| rawGenesisConfig.genesis.config.algorithm.blockperiodseconds | int | `10` |  |
| rawGenesisConfig.genesis.config.algorithm.consensus | string | `"qbft"` |  |
| rawGenesisConfig.genesis.config.algorithm.emptyBlockPeriod | int | `60` |  |
| rawGenesisConfig.genesis.config.algorithm.epochlength | int | `30000` |  |
| rawGenesisConfig.genesis.config.algorithm.requesttimeoutseconds | int | `20` |  |
| rawGenesisConfig.genesis.config.chainId | int | `1337` |  |
| rawGenesisConfig.genesis.difficulty | string | `"0x1"` |  |
| rawGenesisConfig.genesis.gasLimit | string | `"0x47b760"` |  |
| rawGenesisConfig.genesis.nonce | string | `"0x0"` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.0](https://github.com/norwoodj/helm-docs/releases/v1.11.0)
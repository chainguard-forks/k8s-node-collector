
# k8s-node-collector

> [!IMPORTANT]
>
> This is a supported replacement of the original [aquasecurity/k8s-node-collector](https://github.com/aquasecurity/k8s-node-collector) repository.
>
> Community contributions are not being accepted at this time. The documentation has been carried over directly from the original repository and may not reflect recent changes.
>
> We will make a best-effort attempt to address publicly known security vulnerabilities, including CVEs in dependencies and certain source code vulnerabilities when remediation can be achieved safely and with minimal risk. If mitigating a vulnerability would require extensive code changes (for example, adapting to a new API or significant refactoring), we will generally not make that change in order to avoid introducing regressions.
>
> Interested in a CVE-free container image of this project? [Contact Chainguard](https://www.chainguard.dev/contact).
---

[![GitHub Release][release-img]][release]
[![Build Action][action-build-img]][action-build]
[![Release snapshot Action][action-release-snapshot-img]][action-release-snapshot]
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/aquasecurity/k8s-node-collector/blob/main/LICENSE)

The k8s-Node-collector is an open-source collector that gathers Node information (file system and process data) from Kubernetes nodes and outputs it in a JSON format.

## Installation

```sh
git clone git@github.com:aquasecurity/k8s-node-collector.git
cd k8s-node-collector/cmd/node-collector
GOOS=linux GOARCH=arm64 go build -o node-collector main.go
```

## Executing node-collector binary

```sh
Usage:
  node-collector [flags]
  node-collector [command]

Examples:
node-collector k8s [flags]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  k8s         k8s-node-collector extract file system info from cluster Node

Flags:
  -c, --cluster-version string   cluser version. example 1.23.0
  -h, --help                     help for node-collector
      --kubelet-config string    kubelet config via api /api/v1/nodes/<>/proxy/configz encoded in base64
  -n, --node string              node name
  -o, --output string            Output format. One of table|json (default "json")
  -s, --spec-name string         spec name. example: k8s-cis
  -v, --spec-version string      spec version. example 1.23.0
  ```

```sh
./node-collector k8s
```

## Executing Collector specifications

The node-collector executes a collector specification,example [k8s-cis-1.23.0](./pkg/collector/config/specs/k8s-cis-1.23.0.yaml).
Each specification must include:

- `name:` any other platfrom used, example (k8s-cis, aks-cis, gke-cis and etc)

- `version:` of the cis-benchmark it represent (example: 1.23.0)

for executing a specific spec need to pass the `--spec-name k8s-cis` and `--spec-version 1.23.0` flags

If no collector spec has been specified. the node-collector will try to auto detect the matching spec by platform type and version as define in [version_mapping data](./pkg/collector/config/config.yaml)
example:

```yaml
k8s:
  - op: "="
    cluster_version: "1.21"
    spec: k8s-cis-1.21.0
  - op: ">"
    cluster_version: "1.21"
    spec: k8s-cis-1.23.0
```

you can use the `cluster-version` flag in case you do not know what cis spec is supported for you cluster.
this option must be used in conjantion with `spec-name` flag and the matching spec version will be auto detected
example:|
`--spec-name k8s-cis` `--cluster-version 1.23.1`

In the example provided, there are two rules; the first matching rule will obtain the appropriate specification.
Any native k8s cluser with version equal to 1.21 will obtain the `k8s-cis-1.21.0` collector specification it no match found
any native k8s cluser with version grather to 1.21 will obtain the `k8s-cis-1.23.0`

## Adding new collector specifications

In order to add a new specifications, put a new yaml file to this path : `.pkg/collector/config/specs/`
with the following file naming convesion <`platform`-`cis`-`spec_version`>
example: `gke-cis-1.24.0`

Each collector specification audit includes the following fields

```yaml
---
version: "1.23.0"
name: aks-cis
title: Node Specification for AKS info collector
collectors:
  - key: < name to hold the audit command output>
    title: <title of the audit command>
    nodeType: <node type - master | worker>
    audit: <audit shell command>
```

### General spec data

`name`    - name of the spec (example: `aks-cis`)

`version` - version of the spec (example: `1.23.0`)

`title`   - short description of the overall spec

### Specific audit data

`key`      - parameter name to hold the audit shell command output

`title`    - title of the audit shell command

`nodeType` - define the node type on which shell command should be executed (master | worker)

`audit`    - a shell command that collect information and return the result (errors must be supressed)

## Config file

The k8s-node-collector use a config file which help to obtain binaries and config files path based on different platfrom (rancher, native k8s and etc)
for example:

```yaml
kubelet:
    bins:
      - kubelet
      - hyperkube kubelet
    confs:
      - /etc/kubernetes/kubelet-config.yaml
      - /var/lib/kubelet/config.yaml
```

The node collector will obtain the kubelet binary name and config file path based on the platfrom it runs on.
when writing an `audit` shell command the params from config files can be used to collect the appropriate data via config params
example, collect the kubelet config.yaml configuration file ownership:

```sh
stat -c %U:%G $kubelet.confs
```

## Run s k8s job

- simple k8s cluster run following job

```sh
kubectl apply -f job.yaml
```

- Check k8s pod status

```sh
kubectl get pods

NAME                                     READY   STATUS      RESTARTS   AGE
node-collector-ng2z7                          0/1     Completed   0          6m13s
```

- Check k8s pod audit output

```sh
kubectl logs node-collector-ng2z7
```

## k8s-node-collector output

- json output

```json
{
  "apiVersion": "v1",
  "kind": "NodeInfo",
  "metadata":{
    "creationTimestamp":"2023-01-04T11:37:11+02:00"
    },
  "type": "master",
  "info": {
    "adminConfFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "adminConfFilePermissions": {
      "values": [
        600
      ]
    },
    "certificateAuthoritiesFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "certificateAuthoritiesFilePermissions": {
      "values": [
        644
      ]
    },
    "containerNetworkInterfaceFileOwnership": {
      "values": [
        "root:root",
        "root:root"
      ]
    },
    "containerNetworkInterfaceFilePermissions": {
      "values": [
        700,
        775
      ]
    },
    "controllerManagerConfFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "controllerManagerConfFilePermissions": {
      "values": [
        600
      ]
    },
    "etcdDataDirectoryOwnership": {
      "values": [
        "root:root"
      ]
    },
    "etcdDataDirectoryPermissions": {
      "values": [
        700
      ]
    },
    "kubeAPIServerSpecFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeAPIServerSpecFilePermission": {
      "values": [
        600
      ]
    },
    "kubeControllerManagerSpecFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeControllerManagerSpecFilePermission": {
      "values": [
        600
      ]
    },
    "kubeEtcdSpecFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeEtcdSpecFilePermission": {
      "values": [
        600
      ]
    },
    "kubePKIDirectoryFileOwnership": {
      "values": [
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root",
        "root:root"
      ]
    },
    "kubePKIKeyFilePermissions": {
      "values": [
        600,
        600,
        600,
        600,
        600,
        600,
        600,
        600,
        600,
        600,
        600
      ]
    },
    "kubeSchedulerSpecFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeSchedulerSpecFilePermission": {
      "values": [
        600
      ]
    },
    "kubeconfigFileExistsOwnership": {
      "values": [

      ]
    },
    "kubeconfigFileExistsPermissions": {
      "values": [

      ]
    },
    "kubeletAnonymousAuthArgumentSet": {
      "values": [

      ]
    },
    "kubeletAuthorizationModeArgumentSet": {
      "values": [
        "Node",
        "RBAC"
      ]
    },
    "kubeletClientCaFileArgumentSet": {
      "values": [
        "/etc/kubernetes/pki/ca.crt"
      ]
    },
    "kubeletConfFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeletConfFilePermissions": {
      "values": [
        600
      ]
    },
    "kubeletConfigYamlConfigurationFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeletConfigYamlConfigurationFilePermission": {
      "values": [
        644
      ]
    },
    "kubeletEventQpsArgumentSet": {
      "values": [

      ]
    },
    "kubeletHostnameOverrideArgumentSet": {
      "values": [

      ]
    },
    "kubeletMakeIptablesUtilChainsArgumentSet": {
      "values": [

      ]
    },
    "kubeletOnlyUseStrongCryptographic": {
      "values": [

      ]
    },
    "kubeletProtectKernelDefaultsArgumentSet": {
      "values": [

      ]
    },
    "kubeletReadOnlyPortArgumentSet": {
      "values": [

      ]
    },
    "kubeletRotateCertificatesArgumentSet": {
      "values": [

      ]
    },
    "kubeletRotateKubeletServerCertificateArgumentSet": {
      "values": [

      ]
    },
    "kubeletServiceFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "kubeletServiceFilePermissions": {
      "values": [
        644
      ]
    },
    "kubeletStreamingConnectionIdleTimeoutArgumentSet": {
      "values": [

      ]
    },
    "kubeletTlsCertFileTlsArgumentSet": {
      "values": [
        "/etc/kubernetes/pki/apiserver.crt"
      ]
    },
    "kubeletTlsPrivateKeyFileArgumentSet": {
      "values": [
        "/etc/kubernetes/pki/apiserver.key"
      ]
    },
    "kubernetesPKICertificateFilePermissions": {
      "values": [
        644,
        644,
        644,
        644,
        644,
        644,
        644,
        644,
        644,
        644
      ]
    },
    "schedulerConfFileOwnership": {
      "values": [
        "root:root"
      ]
    },
    "schedulerConfFilePermissions": {
      "values": [
        600
      ]
    }
  }
}
```

### job cleanup

```sh
kubectl delete -f job.yaml
```

[release-img]: https://img.shields.io/github/release/aquasecurity/k8s-node-collector.svg?logo=github
[release]: https://github.com/aquasecurity/k8s-node-collector/releases
[action-build-img]: https://github.com/aquasecurity/k8s-node-collector/actions/workflows/build.yaml/badge.svg
[action-build]: https://github.com/aquasecurity/k8s-node-collector/actions/workflows/build.yaml
[action-release-snapshot-img]: https://github.com/aquasecurity/k8s-node-collector/actions/workflows/release-snapshot.yaml/badge.svg
[action-release-snapshot]: https://github.com/aquasecurity/k8s-node-collector/actions/workflows/release-snapshot.yaml

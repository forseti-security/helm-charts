# Config Validator

[Forseti Security](https://forsetisecurity.org/) is a suite of Open-source security tools for GCP.  [Config Validator](https://github.com/forseti-security/config-validator) is a Golang library which provides functionality to evaluate GCP resources against Rego-based policies

## Prerequisites

1. Kubernetes Cluster 1.12+.

## Quick start
Install a Config Validator service by executing:

```bash
helm install forseti-security/config-validator
```

## Installing the Config Validator Chart

### With Tiller (Option 1)

#### Installing
The forseti-security Helm chart can be installed using the following as an example:
```bash
helm install --set production=true \
             --name forseti  \
             --set-string serverKeyContents="$(cat forseti-server.json | base64 - -w 0)" \
             --set-string orchestratorKeyContents="$(cat forseti-client.json | base64 - -w 0)" \
             --set-string serverConfigContents="$(gsutil cat gs://<BUCKET_NAME>/configs/forseti_conf_server.yaml | base64 -)" \
             --values=forseti-values.yaml \
             forseti-security/forseti-security
```
Note that certain values are required.  See the [configuration](#configuration) for details.

Also note that if running on *MacOS*, the `-w 0` flag is not supported for the `base64` command and should be ommitted from the above command.

#### Upgrading

The forseti-security Helm chart can be easily upgraded via the ```helm upgrade``` command.  For example:
```bash
helm upgrade -i forseti forseti-security/forseti-security \
    --set production=true \
    --recreate-pods \
    --set-string serverKeyContents="$(cat forseti-server.json | base64 - -w 0)" \
    --set-string orchestratorKeyContents="$(cat forseti-client.json | base64 - -w 0)" \
    --set-string serverConfigContents="$(gsutil cat gs://<BUCKET_NAME>/configs/forseti_conf_server.yaml | base64 -)" \
    --values=forseti-values.yaml
```

#### Uninstalling

To uninstall/delete the `<RELEASE_NAME>` deployment:

```bash
helm delete <RELEASE_NAME> --purge
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

### Without Tiller (Option 2)

If Tiller is not present in the environment, the charts can still be deployed.

#### Installing or Upgrading

First fetch the chart and download it locally.

```bash
helm fetch forseti-security/config-validator
```

Next, render the template and pipe it into `kubectl`. 

```bash
helm template config-validator-[VERSION].tgz | kubectl apply -f -
```

#### Uninstalling

Similar to installing or upgrading, the Config Validator components can be uninstalled leveraging Helm's `template` sub-command.

```bash
helm template config-validator-[VERSION].tgz | kubectl delete -f -
```

## Configuration

As a best practice, a YAML file that specifies the values for the chart parameters should be provided to configure the chart:

**Copy the default [`config-validator-values.yaml`](values.yaml) value file.**

```bash
helm install -f config-validator-values.yaml <RELEASE_NAME> forseti-security/config-validator
```

See the [All configuration options](#all-configuration-options) section to discover all possibilities offered by the Config Validator chart.

### Pod NetworkPolicy

Optionally, the config-validator helm chart can be deployed with a [Kubernetes Pod NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).  NetworkPolicies provide controls over how pods communicate with one another.  In GKE, [network policy enforcement](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy#using_network_policy_enforcement) must be enabled for Pod NetworkPolicies to take effect.

In this implementation, the NetworkPolicy allows any pod labeled with `configValidator: client` to access the config-validator service.  All other ingrress is denied.

## All Configuration Options

The following table lists the configurable parameters of the Forseti Security chart and their default values. Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
helm install forseti-security/config-validator \
    --name config-validator
    
```

| Parameter                                | Description                                    | Default|
| ----------------------------- | ------------------------------------ |------------------------------------------- |
| image          | This is the container image used by the config-validator  | `gcr.io/forseti-containers/config-validator` |
| imageTag       | This is the tag for the config-validator image.           | `latest` |
| gitSync.image  | This is the container image used by the config-validator git-sync side-car | `gcr.io/google-containers/git-sync` |
| gitSync.imageTag               | This is the container image tag used by the config-validator git-sync side-car | `v3.1.2` |
| gitSync.privateSSHKey          | This is the private OpenSSH key generated to allow the git-sync to clone the policy library repository over SSH. Omitting this value will sync the policy-library over HTTPS. | `nil` |
| gitSync.wait                   | This is the time number of seconds between git-syncs      | `30` |
| loadBalancer                  | Deploy a Load Balancer allowing access to the Forseti server ['none', 'internal', 'external'] | `none` |
| networkPolicy.enable           | Enable pod network policy to limit the connectivty to the server. | `false` |
| networkPolicy.ingressCidr      | A list of CIDR's from which to allow communication to the server.  This is only relevant for client connectivity from outside the Kubernetes cluster. | `[]` |
| nodeSelectors                 | A list of strings in the form of label=value describing on which nodes to run the Forseti on-GKE pods. | `nil` |
| policyLibrary.repositoryURL    | The Git repository containing the policy-library. | `https://github.com/forseti-security/policy-library` |
| policyLibrary.repositoryBranch | The specific git branch containing the policies. | `master` |
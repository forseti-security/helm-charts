# Forseti Security

[Forseti Security](https://forsetisecurity.org/) is a suite of Open-source security tools for GCP.

## Prerequisites

1. Kubernetes Cluster 1.12+ with the [workload-identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) addon enabled.
2. A Forseti environment.  This can be created via the Forseti Security [Terraform module](https://forsetisecurity.org/docs/latest/setup/install.html).
3. A GCP project IAM policy binding tying the Kubernetes Service account for the server (created by this chart) to the GCP IAM Forseti server service account.  This binding is created via the Terraform module or can be created [manually](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity#enable_workload_identity_on_a_new_cluster).
4. A GCP project IAM policy binding tying the Kubernetes Service account for the orchestrator (created by this chart) to the GCP IAM Forseti client service account.  This binding is created via the Terraform module or can be created [manually](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity#enable_workload_identity_on_a_new_cluster).

## Production Configuration
Whether or not to deploy the forseti-security helm chart in a production configuration is controlled by the **production** value.  By default, this is set to **false**.  A production configuration presumes the existence of [Forseti infrastructure](https://forsetisecurity.org/docs/latest/concepts/architecture.html).  The required components are deployed via the Forseti Terrafom Module.  

In a non-production configuration (the default) there are no infrastructure requirements, save Kubernetes.  The only service enabled in the server is the *inventory* service.  The Forseti Orchestrator CronJob is not deployed.  The purpose is to demonstrate a simple deployment allowing for a ```forseti inventory list``` from a client with the CLI.

|                           | Production         | Non-Production (default)  |
|---------------------------|--------------------|---------------------------|
| Forseti Server Pod Container Images | <ul><li>gcr.io/forseti-containers/forseti:latest</li><li>gcr.io/cloudsql-docker/gce-proxy:latest</li></ul> | <ul><li>gcr.io/forseti-containers/forseti:latest</li><li>docker.io/mysql:5.7</li></ul> |
| SQL Data Persistence      | CloudSQL Instance       | EmptyDir Volume Mount |
| Server Configuration File | Value of **serverConfigContents** | files/forseti_conf_server.yaml.sample |
| Load Balancer             | <ul><li>none</li><li>internal</li><li>external</li></ul> | <ul><li>none</li></ul> |

## Quick start

The forseti-security Helm chart by default, deploys the Forseti server in a container running in a [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).  This allows for an external Forseti client to access the server for operations such as ```forseti explain```.  The chart will also deploy a [CloudSQL Proxy container](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine#proxy), in the same pod (and deployment) as the Forseti server.  This allows the Forseti server deployment to access the CloudSQL instance containing Forseti's database.

Optionally, a Forseti orchestrator can be deployed.  This is essentially a container with the Forseti client CLI installed.  It is deployed as a [Kubernetes CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/).  The schedule is user definable in the values.yaml.

## Installing the Forseti Security Chart

### With Tiller (Option 1)

#### Installing
The forseti-security Helm chart can be installed using the following as an example:
```bash
helm install --set production=true \
             --name forseti  \
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
helm fetch forseti-security/forseti-security
```

Next, render the template and pipe it into `kubectl`.  Take note to change the **[SERVER_BUCKET]** and **[VERSION]** values in the command below.

```bash
helm template --set production=true \
              --set-string serverConfigContents="$(gsutil cat gs://[SERVER_BUCKET]/configs/forseti_conf_server.yaml | base64 -)" \
              --values=forseti-values.yaml \
              forseti-security-[VERSION].tgz | kubectl apply -f -
```
Also note that if running on *MacOS*, the `-w 0` flag is not supported for the `base64` command and should be ommitted from the above command.

#### Uninstalling

Similar to installing or upgrading, the Forseti Security components can be uninstalled leveraging Helm's `template` sub-command.

```bash
helm template --set production=true \
              --set-string serverConfigContents="$(gsutil cat gs://[SERVER_BUCKET]/configs/forseti_conf_server.yaml | base64 -)" \
              --values=forseti-values.yaml \
              forseti-security-[VERSION].tgz | kubectl delete -f -
```
Also note that if running on *MacOS*, the `-w 0` flag is not supported for the `base64` command and should be ommitted from the above command.

## Configuration

As a best practice, a YAML file that specifies the values for the chart parameters should be provided to configure the chart:

**Copy the default [`forseti-security-values.yaml`](values.yaml) value file.**

```bash
helm install -f forseti-security-values.yaml <RELEASE_NAME> forseti-security/forseti-security
```

See the [All configuration options](#all-configuration-options) section to discover all possibilities offered by the Forseti Security chart.

### Pod NetworkPolicy

Optionally, the forseti-security helm chart can be deployed with a [Kubernetes Pod NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).  NetworkPolicies provide controls over how pods communicate with one another.  In GKE, [network policy enforcement](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy#using_network_policy_enforcement) must be enabled for Pod NetworkPolicies to take effect.

In this implementation, the NetworkPolicy allows forseti-orchestrator to communicate the forseti-server and only the forseti-server.  It also for the forseti-server to receive traffic from the forseti-orchestrator and only the forseti-orchestrator.  However, if client CLI accesses the server from outside the Kubernetes cluster, then the **networkPolicyIngressCidr** must be defined.  Each item in this list is a CIDR range from which to allow communications to the forseti-server.

![Forseti Pod NetworkPolicy](images/forseti_network_policy.png)

## All Configuration Options

The following table lists the configurable parameters of the Forseti Security chart and their default values. Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
helm install forseti-security/forseti-security \
    --name forseti \
    --set production=true
    --set-string serverConfigContents="$(gsutil cat gs://<BUCKET_NAME>/configs/forseti_conf_server.yaml | base64 -)" \
    --values forseti-values.yaml
    
```

| Parameter                                | Description                                    | Default|
| ----------------------------- | ------------------------------------ |------------------------------------------- |
| **server.cloudsqlConnection**        | This is the connection to the CloudSQL instance.          | `nil`|
| configValidator.enabled               | This sets whether or not to deploy config-validator       | `false` |
| networkPolicy.enabled           | Enable pod network policy to limit the connectivty to the server. | `false` |
| networkPolicy.ingressCidr      | A list of CIDR's from which to allow communication to the server.  This is only relevant for client connectivity from outside the Kubernetes cluster. | `[]` |
| nodeSelectors                 | A list of strings in the form of label=value describing on which nodes to run the Forseti on-GKE pods. | `nil` |
| orchestrator.enabled            | Whether or not to deploy the orchestrator.                | `true`|
| orchestrator.image             | The container image used by the orchestrator.             | `gcr.io/forseti-security-containers/forseti`|
| orchestrator.imageTag          | The tag for the orchestrator container image.              | `v2.23.0` |
| **orchestrator.workloadIdentity**  | the GCP IAM Service account for the Forseti client/orchestrator. | `nil` |
| production                    | Deploy in a production configuration.                      | `false`|
| server.cloudProfilerEnabled           | enables the forseti-server to send metrics to Cloud Profiler | `false` |
| server.loadBalancer                  | Deploy a Load Balancer allowing access to the Forseti server ['none', 'internal', 'external'] | `none` |
| server.rules.bucket                   | The GCS bucket containing the rules.  Often this is the same as the serverBucket.  Ommit the "gs://".| server.bucket |
| server.rules.bucketFolder             | The Folder inside the rulesBucket containing all the rules.| `rules`|
| **server.config.bucket**              | The GCS bucket used by the Forseti server.  Omit the "gs://" | `nil`|
| server.config.bucketFolder      | The folder in the server bucket containing the server configs. | `configs` |
| **server.config.contents**      | The Base64 encoded contents of the forseti_conf_server.yaml file.| `nil`|
| server.image                   | The container image used by the server.                   | `gcr.io/forseti-security-containers/forseti`|
| server.imageTag                | The tag for the server container image.              | `v2.23.0` |
| server.logLevel                | The log level for the server.                             | `info` |
| server.runFrequency               | The cron schedule for the server.  The default is every 60 minute.    | `"*/60 * * * *"` Every 60 minutes|
| **server.workloadIdentity**        | The GCP IAM Service account for the Forseti server.       | `nil` |

**NOTE:** Bolded parameters denotes a required value.
**NOTE 2:** Please see the config-validator chart for input values.
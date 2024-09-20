# Victoria Metrics Agent Helm Chart

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 0.1.0](https://img.shields.io/badge/AppVersion-0.1.0-informational?style=flat-square)

A custom Helm chart for for vm-agent for Amnic
This Helm chart provides a deployment solution for the Victoria Metrics Agent alongside kube-state-metrics and Prometheus Node Exporter. It is designed to collect and forward metrics from Kubernetes clusters to Amnic metrics ingester.

## Table of Contents

- [Overview](#overview)
- [Components](#components)
- [Configuration](#configuration)
  - [Scrape Config](#scrape-config)
  - [Relabeling Config](#relabeling-config)
  - [Dependencies](#dependencies)
- [Installation](#installation)
- [Testing](#testing)
- [Chart Values](#chart-values)
- [License](#license)

## Overview

This Helm chart deploys the following components:

- **Victoria Metrics Agent**: For scraping and forwarding metrics.
- **kube-state-metrics**: Exposes Kubernetes resource metrics.
- **Prometheus Node Exporter**: Collects hardware and OS metrics.

The chart uses ConfigMaps to manage the configuration of these components and supports flexible customization through Helm values.

## Configuration

### Scrape Config

- The scraping config for the vm-agent is stored in `scrape.yml` in `vmagent-config` configMap installed by this chart. 
- By default, this chart installs node-exporter and kube-state-metrics (as dependencies). If you don't require them, you can turn them off in the`values.yaml`. 

> #### **Note:** Disabling them requires you to provide the values (labels and namespaces) for the agent to discover these services. Configure service discovery for node exporter and kube-state-metrics in the `sdConfig` section of the `values.yaml` if you disabled them.

### Relabeling Config

The `global-relabel.yml` in the `vm-agent` ConfigMap allows for relabeling of metrics for Amnic ingesters.

### Dependencies


| Repository | Name | Version |
|------------|------|---------|
| https://prometheus-community.github.io/helm-charts | kube-state-metrics | 5.25.1 |
| https://prometheus-community.github.io/helm-charts | prometheus-node-exporter | 4.39.0 |
| https://victoriametrics.github.io/helm-charts/ | victoria-metrics-agent | 0.12.2 |


Dependencies are managed and installed automatically if `enabled` in `values.yaml`.

## Installation

To install the Helm chart, use the following command:

```bash

helm upgrade -i amnic-agent . \
--set kube-state-metrics.enabled=false \
--set prometheus-node-exporter.enabled=false \
--set victoria-metrics-agent.extraArgs.remoteWrite\\.bearerToken="<TOKEN>" \
--set 'sdConfig.nodeExporter.namespaces[0]'=<nodeExporter_name> \
--set sdConfig.nodeExporter.name=<nodeExporter_name> \
--set sdConfig.kubeStateMetrics.instance=<kubeStateMetrics_instance> \
--set 'sdConfig.kubeStateMetrics.namespaces[0]'=<kubeStateMetrics_namespace> \
--set amnic_relabel.installationID="<installationID>" \
--set amnic_relabel.clusterID="<clusterID>"
-n amnic-agent \
--create-namespace
````
## Testing

Before deploying to production, validate your Helm chart using:

```bash
helm template ./path-to-chart --debug --dry-run --output-dir=<output-to-a-dir>
```
## Chart Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| amnic_clusterID | string | `""` |  |
| amnic_installationID | string | `""` |  |
| amnic_metricsRegex | string | `"(container.*allocation|container_cpu_cfs_.*|container_cpu_usage_seconds_total|container_fs_limit_bytes|container_fs_usage_bytes|container_memory_working_set_bytes|container_network_receive_bytes_total|container_network_transmit_bytes_total|coredns_dns_request_duration_seconds_bucket|coredns_dns_requests_total|coredns_dns_responses_total|coredns_forward_requests_total|karpenter.*|kube_.*_info|.*_created|kube_.*labels|kube_daemonset.*|kube_deployment_.*|kube_node.*|kube_persistentvolume.*|kube_pod.*|kube_replicaset.*|kube_statefulset.*|kubecost.*|kubelet_active_pods|kubelet_desired_pods|kubelet_node_name|kubelet_pod_start_duration_seconds_sum|kubelet_restarted_pods_total|kubelet_running_.*|kubelet_running_container_count|kubelet_running_pod_count|kubelet_started_pods_total|kubelet_volume_stats.*|kubelet_working_pods|kubernetes_build_info|kubernetes_feature_enabled|machine_.*|node_cpu_seconds_total|node_disk_info|node_disk_io_time_seconds_total|node_disk_io_time_weighted_seconds_total|node_filesystem_avail_bytes|node_filesystem_size_bytes|node_gpu_count|node_load.*|node_load1|node_memory_Buffers_bytes|node_memory_Cached_bytes|node_memory_MemAvailable_bytes|node_memory_MemFree_bytes|node_memory_MemTotal_bytes|node_memory_Slab_bytes|node_network_receive_bytes_total|node_network_transmit_bytes_total|node_vmstat_oom_kill|node_vmstat_pgmajfault|node_disk_read_bytes_total|node_disk_written_bytes_total|opencost.*|.*_cost|DCGM.*).*"` |  |
| kube-state-metrics.enabled | bool | `true` |  |
| prometheus-node-exporter.enabled | bool | `false` |  |
| sdConfig.kubeStateMetrics.instance | string | `nil` |  |
| sdConfig.kubeStateMetrics.name | string | `nil` |  |
| sdConfig.kubeStateMetrics.namespaces | list | `[]` |  |
| sdConfig.nodeExporter.metricsPortName | string | `nil` |  |
| sdConfig.nodeExporter.name | string | `nil` |  |
| sdConfig.nodeExporter.namespaces | list | `[]` |  |
| victoria-metrics-agent.configMap | string | `"vmagent-config"` |  |
| victoria-metrics-agent.extraArgs."promscrape.maxScrapeSize" | string | `"33554432"` |  |
| victoria-metrics-agent.extraArgs."remoteWrite.bearerToken" | string | `""` |  |
| victoria-metrics-agent.extraArgs."remoteWrite.relabelConfig" | string | `"/tmp/relabel/global-relabel.yml"` |  |
| victoria-metrics-agent.extraVolumeMounts[0].mountPath | string | `"/tmp/relabel"` |  |
| victoria-metrics-agent.extraVolumeMounts[0].name | string | `"amnic-relabel"` |  |
| victoria-metrics-agent.extraVolumes[0].configMap.name | string | `"vmagent-config"` |  |
| victoria-metrics-agent.extraVolumes[0].name | string | `"amnic-relabel"` |  |
| victoria-metrics-agent.remoteWriteUrls[0] | string | `"https://metrics.amnic.com/z/api/v1/push"` |  |

## License


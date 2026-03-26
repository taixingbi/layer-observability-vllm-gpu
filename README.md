# GPU node — DCGM metrics

NVIDIA GPU metrics for Prometheus/Grafana via [DCGM Exporter](https://github.com/NVIDIA/dcgm-exporter).

## Run

Prerequisites: Docker with Compose plugin, NVIDIA driver, [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

```bash
docker compose up -d
```

Metrics: `http://<gpu-node-ip>:9400/metrics` (example node: `http://192.168.86.173:9400/metrics`).

See [plan.md](plan.md) for metric categories and compose details.

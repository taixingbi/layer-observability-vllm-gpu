# vLLM GPU Observability Stack

This repository provides a runnable setup for monitoring two GPU vLLM nodes from a central Prometheus/Grafana monitoring host.

## Topology

- `gpu-node-1` (`192.168.86.173`)
- `gpu-node-2` (`192.168.86.176`)
- `monitoring-node` (Prometheus + Grafana)

Each GPU node runs:

- `vllm` on `:8000`
- `dcgm-exporter` on `:9400`
- `node-exporter` on `:9100`

## Prerequisites (GPU Nodes)

- Docker + Docker Compose plugin
- NVIDIA driver
- NVIDIA Container Toolkit
- Open ports: `8000`, `9100`, `9400` from monitoring node

Quick checks:

```bash
nvidia-smi
docker --version
docker compose version
```

## 1) Start Services on Each GPU Node

Copy this repo to both GPU nodes, then on each node:

```bash
cp .env.example .env
# Edit .env for this node
```

Update per-node values:

- `NODE_NAME=gpu-node-1` (or `gpu-node-2`)
- `VLLM_MODEL=Qwen/Qwen2.5-7B-Instruct` (or your model)
- Optional `HUGGING_FACE_HUB_TOKEN` if your model requires auth

Bring up services:

```bash
docker compose -f docker-compose.gpu-node.yml up -d
```

Validate endpoints:

```bash
curl http://localhost:8000/v1/models
curl http://localhost:8000/metrics
curl http://localhost:9400/metrics
curl http://localhost:9100/metrics
```

## 2) Configure Prometheus on Monitoring Node

Use `prometheus/prometheus.yml` and update targets if IPs differ.

Start Prometheus (example):

```bash
docker run -d \
  --name prometheus \
  --restart unless-stopped \
  -p 9090:9090 \
  -v "$(pwd)/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro" \
  prom/prometheus:latest
```

Start Grafana (example):

```bash
docker run -d \
  --name grafana \
  --restart unless-stopped \
  -p 3000:3000 \
  grafana/grafana:latest
```

## 3) Operations

On GPU nodes:

```bash
docker compose -f docker-compose.gpu-node.yml ps
docker compose -f docker-compose.gpu-node.yml logs -f vllm
docker compose -f docker-compose.gpu-node.yml logs -f dcgm-exporter
docker compose -f docker-compose.gpu-node.yml logs -f node-exporter
docker compose -f docker-compose.gpu-node.yml restart
docker compose -f docker-compose.gpu-node.yml down
```

## Expected Metrics Coverage

- GPU: utilization, memory, temperature, power (DCGM exporter)
- vLLM: request and latency metrics from `/metrics`
- Host: CPU, memory, disk, network (Node exporter)

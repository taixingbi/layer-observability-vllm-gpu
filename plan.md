
gpu-node-1
gpu-node-2



vLLM
dcgm-exporter
node-exporter


README.md
# GPU Nodes with Docker for vLLM Monitoring

This setup runs on two GPU machines:

- `gpu-node-1` → `192.168.86.173`
- `gpu-node-2` → `192.168.86.176`

Each GPU node runs Docker containers for:

1. **vLLM** — model serving
2. **DCGM Exporter** — NVIDIA GPU metrics
3. **Node Exporter** — host metrics (CPU / memory / disk / network)

A separate monitoring machine runs:

- **Prometheus** — scrapes metrics
- **Grafana** — dashboards

---

## Architecture

```text
gpu-node-1 (192.168.86.173)
  ├─ vLLM              -> :8000
  ├─ dcgm-exporter     -> :9400
  └─ node-exporter     -> :9100

gpu-node-2 (192.168.86.176)
  ├─ vLLM              -> :8000
  ├─ dcgm-exporter     -> :9400
  └─ node-exporter     -> :9100

monitoring node
  ├─ Prometheus        -> scrape all metrics
  └─ Grafana           -> dashboards
Goals

Each GPU node should:

serve vLLM traffic
expose GPU metrics
expose host metrics

This allows Prometheus and Grafana to monitor:

GPU utilization
GPU memory used
GPU temperature
GPU power usage
vLLM request metrics
TTFT
tokens/sec
host CPU / memory / disk / network
Node Names

Use these logical names consistently:

gpu-node-1 = 192.168.86.173
gpu-node-2 = 192.168.86.176

These names are useful in Prometheus labels and Grafana panels.

Requirements

Install on each GPU node:

Docker
NVIDIA driver
NVIDIA Container Toolkit

Check GPU visibility:

nvidia-smi

Check Docker:

docker --version
Ports

Each GPU node exposes:

8000 → vLLM API + /metrics
9400 → dcgm-exporter metrics
9100 → node-exporter metrics

Make sure these ports are reachable from the monitoring machine.

Run on gpu-node-1
1. Start vLLM

Example:

docker run -d \
  --name vllm \
  --restart unless-stopped \
  --gpus all \
  -p 8000:8000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  vllm/vllm-openai:latest \
  --model Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 8000
2. Start DCGM Exporter
docker run -d \
  --name dcgm-exporter \
  --restart unless-stopped \
  --gpus all \
  --cap-add SYS_ADMIN \
  -p 9400:9400 \
  nvcr.io/nvidia/k8s/dcgm-exporter:latest
3. Start Node Exporter
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  prom/node-exporter
Run on gpu-node-2

Repeat the same commands on gpu-node-2.

1. Start vLLM
docker run -d \
  --name vllm \
  --restart unless-stopped \
  --gpus all \
  -p 8000:8000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  vllm/vllm-openai:latest \
  --model Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 8000
2. Start DCGM Exporter
docker run -d \
  --name dcgm-exporter \
  --restart unless-stopped \
  --gpus all \
  --cap-add SYS_ADMIN \
  -p 9400:9400 \
  nvcr.io/nvidia/k8s/dcgm-exporter:latest
3. Start Node Exporter
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  prom/node-exporter
Verify on Each GPU Node
vLLM
curl http://localhost:8000/v1/models
curl http://localhost:8000/metrics
DCGM Exporter
curl http://localhost:9400/metrics | grep DCGM
Node Exporter
curl http://localhost:9100/metrics
Example Prometheus Config

Run Prometheus on a separate monitoring machine and scrape both GPU nodes.

global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "vllm"
    static_configs:
      - targets: ["192.168.86.173:8000"]
        labels:
          node: "gpu-node-1"
      - targets: ["192.168.86.176:8000"]
        labels:
          node: "gpu-node-2"

  - job_name: "dcgm"
    static_configs:
      - targets: ["192.168.86.173:9400"]
        labels:
          node: "gpu-node-1"
      - targets: ["192.168.86.176:9400"]
        labels:
          node: "gpu-node-2"

  - job_name: "node"
    static_configs:
      - targets: ["192.168.86.173:9100"]
        labels:
          node: "gpu-node-1"
      - targets: ["192.168.86.176:9100"]
        labels:
          node: "gpu-node-2"
Recommended Grafana Panel Names
GPU
GPU Util - gpu-node-1
GPU Util - gpu-node-2
GPU Memory Used - gpu-node-1
GPU Memory Used - gpu-node-2
GPU Temp - gpu-node-1
GPU Temp - gpu-node-2
GPU Power - gpu-node-1
GPU Power - gpu-node-2
vLLM
Completion RPS - gpu-node-1
Completion RPS - gpu-node-2
Latency P95 - gpu-node-1
Latency P95 - gpu-node-2
TTFT - gpu-node-1
TTFT - gpu-node-2
Tokens/sec - gpu-node-1
Tokens/sec - gpu-node-2
Host
CPU Usage - gpu-node-1
CPU Usage - gpu-node-2
Memory Usage - gpu-node-1
Memory Usage - gpu-node-2
Container Management

List containers:

docker ps

View logs:

docker logs -f vllm
docker logs -f dcgm-exporter
docker logs -f node-exporter

Restart:

docker restart vllm
docker restart dcgm-exporter
docker restart node-exporter

Remove:

docker rm -f vllm dcgm-exporter node-exporter
Notes
Keep the same container names on both GPU nodes for simplicity:
vllm
dcgm-exporter
node-exporter
Distinguish nodes by IP and Prometheus label:
node="gpu-node-1"
node="gpu-node-2"
Do not run separate Prometheus instances on each GPU node unless specifically needed.
The GPU nodes are service nodes.
Prometheus and Grafana should run centrally.
Summary

Each GPU node is responsible for:

serving vLLM
exposing GPU metrics
exposing host metrics

Prometheus is responsible for:

scraping all nodes

Grafana is responsible for:

visualizing all metrics

---



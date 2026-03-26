# GPU Node Metrics - DCGM Exporter

This node exposes NVIDIA GPU metrics for Prometheus and Grafana.

## Service

`dcgm-exporter` is NVIDIA’s Prometheus exporter for GPU metrics.

It exposes metrics on:

- `http://<gpu-node-ip>:9400/metrics`

Example:

- `http://192.168.86.173:9400/metrics`

## Purpose

This container is used to monitor GPU health and usage on the GPU node.

Typical metrics include:

- GPU utilization
- GPU memory used / free
- GPU temperature
- GPU power usage
- SM clock
- memory clock
- XID errors

This service is lightweight and is intended to run directly on each GPU machine.

## Docker Compose

```yaml
services:
  dcgm-exporter:
    image: nvcr.io/nvidia/k8s/dcgm-exporter:latest
    container_name: dcgm-exporter
    restart: unless-stopped
    cap_add:
      - SYS_ADMIN
    ports:
      - "9400:9400"
    gpus: all
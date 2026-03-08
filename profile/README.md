# VECTR

A platform for remotely controlling and autonomously operating drones.

## Architecture

```
Drone
├── Flight Controller (Betaflight)   ←— MSP over Serial
└── Raspberry Pi
    ├── node          (Go)           ←— commands / telemetry
    └── reticulum-bridge (Python)    ←— Unix socket ↕ WiFi

                    ↕ WiFi / Reticulum mesh

Base Station
├── reticulum-bridge (Python)        ←— WiFi ↕ Unix socket
└── base             (Go)            ←— gRPC ↓

                    ↕ gRPC / TCP

Infrastructure
└── gateway          (Go)            ←— browser UI + WebSocket bridge
```

The **Reticulum bridge** (`rns_bridge.py` in `proto`) is a Python process running on both the drone and the base station. It handles all wireless mesh networking over the local WiFi link and exposes a Unix socket to the Go binaries on each side. This keeps the Go code transport-agnostic — UDP and Reticulum are both supported today.

The **`DroneControl` gRPC service** defines the full operator API: streaming stick inputs, flight commands (arm, disarm, set mode, return home), emergency stop, and a live telemetry stream (attitude, position, battery, flight mode).

## Repository

All edge and infrastructure code lives in a single monorepo:

| Repo | Description |
|------|-------------|
| [edge](https://github.com/vectr-ch/edge) | Monorepo containing `node/` (drone), `base/` (ground station), `gateway/` (operator UI), `proto/` (shared protobuf + transport), and `deploy/` (ops scripts). |

## Hardware (Phase 1)

| Role | Hardware |
|------|----------|
| Flight controller | Betaflight FC |
| On-board computer | Raspberry Pi (possible upgrade to Jetson Nano for on-board inference) |
| Wireless link | WiFi antenna via Reticulum mesh |
| Base station | Linux machine with WiFi antenna |

## Roadmap

| Phase | Focus |
|-------|-------|
| 1 — now | Manual remote control with real-time stick inputs |
| 2 | Waypoint missions, richer FC protocol (MAVLink / PX4) |
| 3 | Autonomous operation with on-board inference |

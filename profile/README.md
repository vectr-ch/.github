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

## Repositories

| Repo | Description |
|------|-------------|
| [node](https://github.com/vectr-ch/node) | On-board Go binary running on the Raspberry Pi. Talks to the flight controller over MSP/serial and to the base via the Reticulum bridge. |
| [base](https://github.com/vectr-ch/base) | Ground base station Go binary. Receives telemetry and forwards commands between the Reticulum bridge and the gateway. |
| [gateway](https://github.com/vectr-ch/gateway) | Operator-facing gateway. Serves a browser-based control UI and bridges WebSocket connections to the base station's gRPC API. |
| [proto](https://github.com/vectr-ch/proto) | Shared Protobuf definitions and generated Go code used across all services. Includes the `DroneControl` gRPC service and pluggable transport layer (UDP today, Reticulum when ready). |
| reticulum-bridge | Python bridge (`rns_bridge.py`) providing Reticulum mesh networking. Runs on both the drone and the base station, connected to the Go binaries via Unix socket. Lives in `proto/transport/`. |
| [deploy](https://github.com/vectr-ch/deploy) | Idempotent setup scripts for the drone Pi and base station: WiFi AP/client configuration, Reticulum and bridge systemd services, and a Makefile for checking link status and logs across nodes over SSH. |

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

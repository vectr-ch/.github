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
└── gateway          (Go)            ←— connects to ops tooling
```

The **Reticulum bridge** is a Python process on both the drone and the base station. It handles all wireless mesh networking and exposes a Unix socket to the Go binaries on each side. This keeps the Go code transport-agnostic.

## Repositories

| Repo | Description |
|------|-------------|
| [node](https://github.com/vectr-ch/node) | On-board Go binary running on the Raspberry Pi. Talks to the flight controller over MSP/serial and to the base via the Reticulum bridge. |
| [base](https://github.com/vectr-ch/base) | Ground base station Go binary. Receives telemetry and forwards commands between the Reticulum bridge and the gateway. |
| [gateway](https://github.com/vectr-ch/gateway) | Infrastructure gateway. Exposes the control and telemetry interface to operators and tooling. |
| [proto](https://github.com/vectr-ch/proto) | Shared Protobuf definitions and generated Go code used across all services. |
| reticulum-bridge *(in progress)* | Python bridge providing Reticulum mesh networking. Runs on both the drone and the base station, connected to the Go binaries via Unix socket. |

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

# Katomato 
Modular automation system for indoor plant environments, built with Arduino and Python. 

[![License: MIT](https://img.shields.io/badge/License-MIT-F2BF42.svg)](https://opensource.org/licenses/MIT)
[![Python 3.12+](https://img.shields.io/badge/python-3.12+-3872E0.svg)](https://www.python.org/downloads/)
[![Poetry](https://img.shields.io/badge/managed_by-poetry-3872E0.svg)](https://python-poetry.org/)
[![Type Checked: mypy](https://img.shields.io/badge/type%20checked-mypy-3872E0.svg)](http://mypy-lang.org/)
[![Tests: pytest](https://img.shields.io/badge/tests-pytest-58A65D.svg)](https://pytest.org/)

[![C++](https://img.shields.io/badge/c++-arduino-2D6569.svg?logo=c%2B%2B&logoColor=white)](https://www.arduino.cc/)
[![PlatformIO](https://img.shields.io/badge/embeded_env-platformio-D57A2D.svg?logo=platformio&logoColor=%23f5822a)](https://platformio.org/)

## Features

- **Climate and Soil Moisture Control** — Maintains target temperature, humidity, and soil moisture through closed-loop
  feedback between sensors and actuators.
- **Data Logging & Visualization** — Persists all sensor readings to InfluxDB, with interactive time-series dashboards
  available through its built-in UI.
- **Plug-and-Play Architecture** — Sensors, actuators, controllers, and devices attach through well-defined integration
  points. Adding or removing a component requires no changes to the core logic on either side.
- **Python Control App** — Receives sensor data from Arduino, applies control rules, and sends commands back to
  actuators. Supports CLI-based manual override.
- **Dockerized Deployment** — Python app and InfluxDB run in isolated containers.

## Project Structure

- `app/` — Python application that communicates with the Arduino, drives actuator logic, and logs data to InfluxDB.
- `arduino/` — PlatformIO-based firmware for Arduino-compatible boards; reads sensors and executes control commands.
- `docs/` — Project documentation.
- `Makefile` — Unified dev commands across both environments.

## Quick Start

### Prerequisites
- Arduino UNO
- Any sensor and any actuator, for example DHT22 and LED
- Docker
- PlatformIO

### Installation

```bash
# Install dependencies
pip3 install platformio
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Clone the repository
git clone https://github.com/<your-repo>/katomato.git
cd katomato

# Build and run
make build-arduino
make upload-arduino
make up
```

> See [Setup guide](docs/setup-guide.md) for detailed instructions.

## Architecture

<p align="center">
    <img src="assets/architecture-diagram.png" alt="architecture diagram" width="1000"/>
</p>

Arduino sensors sample the physical environment and stream structured JSON to the Python server over serial.
The server validates each reading, routes it to the appropriate controller, and the controller decides what action
the connected devices should take. Devices translate those actions into hardware commands and place them on an
internal queue; the server core drains the queue and sends the commands back to Arduino, which forwards them to the
corresponding actuator. Every sensor reading is also published to InfluxDB for historical analysis and dashboarding.

Both sides — firmware and server — follow the same principle: a generic core handles data flow, while
domain-specific modules (sensors, controls, controllers, devices) plug in through a registration mechanism.
Nothing in the core needs to change when a module is added or removed.

> See the [Architecture overview](docs/architecture-overview.md) for a detailed breakdown.

## Electric circuit
> See [Circuit and specification](docs/electrical-circuit.md).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Copyright (c) 2025 Alexander Suvorov

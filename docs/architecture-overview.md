# Architecture

<p align="center">
    <img src="../assets/architecture-diagram.png" alt="architecture diagram" width="1000"/>
</p>

Katomato is a plug-and-play automation framework. Although it was designed for indoor plant growing, the
architecture is intentionally generic — the same core can drive any IoT automation scenario.

The system is split into two cooperating halves: Arduino firmware that interfaces with the physical world, and a
Python server that makes decisions. Each half follows the same structural pattern — a thin, generic core surrounded
by domain-specific modules — but they play complementary roles.

---

## Arduino Firmware (C++ / PlatformIO)

The firmware sits closest to the hardware. It reads sensor values, encodes them as structured JSON, and streams
the data to the server over serial. In the other direction, it receives command JSON from the server and drives the
appropriate actuator output.

Internally, the firmware has three layers:

### **Core**
The core loop iterates over every registered sensor, collects its reading, and forwards the JSON to the server. When
a command arrives, the core dispatches it to the matching control output. It never interprets what a reading means or
what an actuator does — that knowledge lives entirely in the modules.

### **Sensors**
Each sensor implements a unified `Sensor` interface. Registering a new implementation with the core is enough to
start streaming its data — no other wiring is needed.

### **Controls**
Each actuator type (digital, analog, PWM, etc.) implements a `Control` interface. The core calls into it whenever
a matching command arrives, without knowing what the actuator physically does.

---

## Server Application (Python)

The Python server is the decision-making layer. It receives sensor JSON from the Arduino, runs it through
control logic, and sends actuator commands back. It also publishes every reading to InfluxDB.

Its architecture mirrors the firmware's three-layer model, but the middle layer is split into two distinct
concepts — controllers and devices — to separate *what should happen* from *how to make it happen*.

### **Core**
The server core listens for sensor data on the serial connection, parses it into `SensorData` objects, and
hands each one to the controller registered for that sensor type. After controllers and devices have produced
their commands, the core drains the command queue and transmits the results back to the Arduino.

### **Controllers**
A controller encodes a single environmental rule — for instance, "keep temperature within a target range."
It receives a `SensorData` reading, evaluates it against configured thresholds, and decides on an `Action`
(`UP` or `DOWN`). The action is then forwarded to one or more devices listed in the sensor message. Controllers
are unaware of which physical sensor produced the data or which actuator will ultimately carry out the action.

### **Devices**
A device translates an abstract action into a concrete hardware command. A `DOWN` action on an exhaust fan device,
for example, might produce a JSON command that sets a specific PWM pin to a calculated duty cycle. The resulting
command is placed on the internal message queue. Devices have no knowledge of which controller triggered them or
how the command reaches the hardware.

---

## Shared Infrastructure

### **Message Queue**
All device commands flow through an in-process `asyncio.Queue`. The server core consumes this queue in a
dedicated coroutine and forwards each command to the Arduino over serial.

### **InfluxDB**
Every sensor reading is intercepted and published to InfluxDB, providing time-series storage and visualization
through its built-in dashboard UI.

---

## Extensibility

Both sides rely on runtime registration to discover modules. On the Arduino, sensors and controls register
with the core at startup. On the Python side, controllers and devices self-register through decorators
(`@controller_registry`, `@device_registry`), and their packages are auto-imported so that dropping a new
module file into the right directory is all it takes.

Because the core on each side is completely agnostic to what modules are present, adding or removing a
sensor, control, controller, or device never requires a core change.

> See the [Development guide](development-guide.md) for concrete examples.

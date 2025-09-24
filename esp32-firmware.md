## Embedded Backend: ESP32 Firmware (`BLE-Connect`)

The firmware running on the ESP32 is the heart of the data collection system. It is responsible for collecting sensor data, processing it, and serving it over BLE to the mobile application. The firmware is built using the ESP-IDF framework and C.

### Supported Targets:

|   |   |   |   |   |
|---|---|---|---|---|
|**ESP32**|**ESP32-C2**|**ESP32-C3**|**ESP32-S2**|**ESP32-S3**|

### Key Features & Technical Details:

- **Dual Data-Source Architecture:** The firmware features a modular data acquisition system, controlled by the `USE_I2C_DATA_SOURCE` macro in `data_config.h`. This allows the firmware to be compiled to use either a `mock_data` module for a sophisticated physics-based simulation or an `i2c_data` module to read data from a real I2C sensor.
- **BLE GATT Server:** The firmware implements a custom BLE GATT service using the high-performance **NimBLE** stack. This service is designed for low power consumption and efficient data transfer.
- **Physics-Based Simulation:** The `mock_data.c` module provides a realistic simulation of the heat exchange system. It's not just a simple mock; it implements a thermal model that simulates heat transfer, cooling dynamics, and the behavior of system components like the pump and heat exchanger. It also models system degradation over time, including pump wear, thermal stress, and heat exchanger fouling.
- **I2C Communication:** When configured to use a real sensor, the `i2c_data.c` module implements the I2C slave logic to communicate with a master controller. It parses incoming CSV data from the master and can send control commands back to it.
- **Historical Data Queue:** The `queue_system` implements a circular buffer that stores the last 100 sensor data points. This feature ensures that no data is lost if the mobile app disconnects. Upon reconnection, the app can read the data from this queue.
- **Concurrent Operation with FreeRTOS:** The firmware leverages FreeRTOS to run the BLE stack and the data generation/processing tasks as separate, concurrent tasks. This ensures that the BLE communication remains responsive even while the system is performing other operations.

### BLE Service Structure:

The firmware defines a custom BLE service with the following structure:

- **Service UUID:** `0x1230`

- **Characteristics:**
    - **`0xFEF4` (Read):** A simple characteristic that provides basic server information. Used for verifying the connection.
    - **`0xDEAD` (Write):** The control characteristic. The mobile app can write to this characteristic to send commands to the ESP32, such as turning the heater on or off.
    - **`0x1234` (Read/Notify):** The main data characteristic. It streams the current sensor data in JSON format. The mobile app subscribes to notifications on this characteristic to receive real-time updates.
    - **`0x1235` (Read):** The historical data characteristic. The mobile app can read this characteristic to retrieve data from the historical data queue.

### JSON Data Format:

Sensor data is serialized into a compact JSON object for efficient transmission over BLE.

```json
{
  "t": 12345,
  "oilReservoirLevel": 75.50,
  "heatingElementLevel": 60.25,
  "oilTankTemperature": 45.30,
  "tempPumpToHeater": 42.15,
  "tempHeaterToHx": 85.20,
  "tempHeatingElement1": 95.80,
  "tempHeatingElement2": 94.25,
  "tempHeatingElement3": 96.10,
  "tempHeatingElement4": 93.75,
  "tempBeforeHx": 80.45,
  "ambientTemperature": 25.60,
  "pumpFlowRate": 6.80,
  "systemVoltage": 24.15,
  "systemCurrent": 11.30
}
```

### Control Commands:

Clients can write to characteristic `0xDEAD` to control the system. The following commands are supported:

- `HEATER:ON`: Turns on the simulated heating elements.
- `HEATER:OFF`: Turns off the simulated heating elements.
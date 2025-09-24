## Development & Setup

This document outlines the setup process for the development environment for both the Flutter mobile application and the ESP32 firmware.

### Flutter Application

The mobile application is built with Flutter. The project requires the Flutter SDK to be installed.

1.  **Project Location:**
    The Flutter project is located in the `D:\heat-monitor` directory.

2.  **Install dependencies:**
    Navigate to the project directory and run the following command to fetch the required packages:
    ```bash
    flutter pub get
    ```

3.  **Run the application:**
    To run the application on a connected device or an emulator, use the following command:
    ```bash
    flutter run
    ```

4.  **Toggle Simulation Mode:**
    The application can be run in a simulation mode that doesn't require a physical hardware connection. To enable this, open `lib/main.dart` and change the `useBle` flag to `false`, or ensure the `SimulatedSensorDataService` is instantiated instead of `BleSensorDataService`. This will make the app use the physics-based simulated data provider.

### ESP32 Firmware

The ESP32 firmware is built using the Espressif IoT Development Framework (ESP-IDF).

1.  **ESP-IDF Installation:**
    The project requires the ESP-IDF to be installed and configured. The `idf.py` tool should be in the system's PATH.

2.  **Firmware Location:**
    The firmware code is located in the `D:\heat-monitor\BLE-Connect` directory.

3.  **Firmware Configuration (Optional):**
    To configure the firmware, use the `menuconfig` tool. This allows for changing settings like the data source (mock vs. I2C), GPIO pins, and other parameters.
    ```bash
    cd D:\heat-monitor\BLE-Connect
    idf.py menuconfig
    ```

4.  **Build the Firmware:**
    To compile the firmware, run the following command. This will produce a binary file that can be flashed to the ESP32.
    ```bash
    idf.py build
    ```

5.  **Flash the Firmware:**
    To flash the compiled firmware to a connected ESP32 device, use the following command, replacing `(PORT)` with the device's serial port (e.g., `COM3` on Windows, `/dev/ttyUSB0` on Linux).
    ```bash
    idf.py -p (PORT) flash
    ```

6.  **Monitor the Output:**
    To view the serial output from the ESP32, use the `monitor` command:
    ```bash
    idf.py -p (PORT) monitor
    ```
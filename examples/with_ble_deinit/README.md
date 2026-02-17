# WiFi Manager - BLE Soft Deinit Example (NimBLE)

Demonstrates the "service-only" BLE mode where the application owns the NimBLE stack. The WiFi Manager registers its GATT service for provisioning, then cleanly removes it on deinit while the NimBLE stack keeps running for the application's own BLE services.

## Use Case

Many applications need BLE for more than just WiFi provisioning — sensor data streaming, device control, OTA updates, etc. With the default BLE mode, calling `wifi_manager_deinit()` tears down the entire NimBLE stack, making BLE unusable. Service-only mode solves this by letting the application own the stack lifecycle.

## Flow

```
1. App initializes NimBLE stack
2. wifi_manager_init() detects NimBLE already running
   → registers 0xFFE0 GATT service only (service-only mode)
3. User provisions WiFi via BLE
4. wifi_manager_deinit() removes 0xFFE0 service, leaves NimBLE running
5. App registers its own 0xAA00 GATT service and advertises
```

## Build & Flash

```bash
cd examples/with_ble_deinit
idf.py build flash monitor
```

This example is configured for ESP32-S3 with NimBLE. To change the target:

```bash
idf.py set-target esp32c3  # or esp32, esp32c6, etc.
```

## Usage

1. Flash the device and open the serial monitor
2. Use the [Python BLE CLI](../../tools/wifi_ble_cli/) to provision WiFi:

```bash
cd tools/wifi_ble_cli
python wifi_ble_cli.py scan
python wifi_ble_cli.py add "MyWiFi" "password123"
python wifi_ble_cli.py connect
```

3. Once WiFi connects (or after 60s timeout), the WiFi Manager deinitializes and the app takes over BLE
4. The device now advertises as "ESP32-App" with a custom 0xAA00 GATT service
5. Use any BLE scanner app to verify the 0xFFE0 service is gone and 0xAA00 is present

## Expected Output

```
I (xxx) ble_deinit_example: === BLE Soft Deinit Example ===
I (xxx) ble_deinit_example: Step 1: Initializing NimBLE stack (app-owned)
I (xxx) ble_deinit_example: NimBLE stack running
I (xxx) ble_deinit_example: Step 2: Initializing WiFi Manager (service-only BLE mode)
I (xxx) wifi_mgr_ble_nb: NimBLE stack already running, registering service only
I (xxx) ble_deinit_example: WiFi Manager initialized (BLE service-only mode)
I (xxx) ble_deinit_example: Step 3: Waiting for WiFi connection...
  ... user provisions WiFi via BLE ...
I (xxx) ble_deinit_example: WiFi connected!
I (xxx) ble_deinit_example: Step 4: Deinitializing WiFi Manager (BLE service removed, stack stays)
I (xxx) ble_deinit_example: WiFi Manager deinitialized — NimBLE stack still running
I (xxx) ble_deinit_example: Step 5: Registering app GATT service and advertising
I (xxx) ble_deinit_example: App GATT service 0xAA00 registered
I (xxx) ble_deinit_example: App BLE advertising started: ESP32-App
I (xxx) ble_deinit_example: WiFi: MyWiFi (78%) | BLE: app service active
```

## Key Differences from `with_ble` Example

|                  | `with_ble`             | `with_ble_deinit`    |
|------------------|------------------------|----------------------|
| Stack owner      | WiFi Manager           | Application          |
| Host stack       | Bluedroid (default)    | NimBLE               |
| After deinit     | BLE completely stopped | BLE keeps running    |
| App BLE services | Not possible           | Yes (0xAA00 example) |

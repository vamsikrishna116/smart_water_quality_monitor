# Smart Water Quality Monitoring System (ESP32 + Arduino IoT Cloud)

An ESP32-based IoT system that measures **water temperature, electrical conductivity (EC) and TDS (Total Dissolved Solids)**, shows the readings on an OLED display, and streams them live to an **Arduino IoT Cloud** dashboard viewable on a laptop or phone.

> Course project: *Electronic System Automation (22SDEC02R)*, B.Tech ECE, KL University.

![Hardware setup](images/hardware_setup.jpg)

## Features

- Real-time measurement of temperature, EC and TDS
- Temperature-compensated EC calculation
- 16-bit ADC (ADS1115) for more precise analog readings than the ESP32's built-in ADC
- Local display on a 0.96" I2C OLED
- Remote monitoring through an Arduino IoT Cloud dashboard (web + mobile)
- Serial-command calibration of the TDS/EC probe

## Hardware Required

| Component | Purpose |
|---|---|
| ESP32 Dev Kit V1 | Main controller with Wi-Fi |
| TDS sensor (analog, e.g. DFRobot Gravity TDS meter) | Measures water conductivity |
| DS18B20 waterproof temperature sensor + terminal adaptor | Measures water temperature (1-Wire) |
| ADS1115 16-bit ADC module | Digitizes the TDS sensor's analog output |
| 0.96" I2C OLED (SSD1306, 128x64) | Local display |
| 4.7 kΩ resistor | Pull-up for the DS18B20 data line (if your adaptor doesn't include one) |
| Breadboard and jumper wires | Connections |

## Pin Connections

| Device | Pin | ESP32 / Connects to |
|---|---|---|
| TDS sensor | VCC / GND | Vin / GND |
| TDS sensor | Analog out | ADS1115 **A0** |
| ADS1115 | SDA / SCL | GPIO 21 / GPIO 22 |
| ADS1115 | VCC / GND | 3.3V / GND |
| OLED (SSD1306) | SDA / SCL | GPIO 21 / GPIO 22 |
| OLED (SSD1306) | VCC / GND | 3.3V / GND |
| DS18B20 | Data | GPIO 18 (with 4.7 kΩ pull-up to 3.3V) |
| DS18B20 | VCC / GND | 3.3V / GND |

The ADS1115 and the OLED share the same I2C bus (OLED address `0x3C`, ADS1115 default `0x48`).

![Circuit diagram](docs/circuit_diagram.png)

## Software and Libraries

Built with the Arduino IDE (or Arduino Cloud editor). Install the ESP32 board package, then these libraries:

- `ArduinoIoTCloud` and `Arduino_ConnectionHandler`
- `OneWire`
- `DallasTemperature`
- `Adafruit ADS1X15`
- `DFRobot_ESP_EC`
- `Adafruit GFX Library`
- `Adafruit SSD1306`

## Setup

1. **Create the cloud Thing.** In [Arduino Cloud](https://cloud.arduino.cc/), create a new Thing and add three **float, read-only** variables named exactly:
   - `temperature`
   - `ecValue`
   - `tDS`
2. **Add your device** (ESP32) to the Thing and enter your Wi-Fi name and password. Arduino Cloud generates `thingProperties.h` and `secrets.h` for you.
3. **Keep secrets private.** Do **not** upload `secrets.h`. Copy `firmware/secrets_example.h` to `secrets.h` and fill in your own values.
4. **Wire the hardware** as shown in the pin table above.
5. **Upload the sketch** in `firmware/` to the ESP32.
6. **Build a dashboard** with gauges for Temperature, EC and TDS, plus a chart for TDS.
7. Open the Serial Monitor at **115200 baud** to see readings.

## Calibration

Calibration uses the DFRobot EC library over the Serial Monitor:

1. Type `enterec` to enter calibration mode.
2. Place the probe in a standard solution (for example 1413 µS/cm) and type `calec`.
3. Type `exitec` to save the calibration to EEPROM.

Recalibrate periodically, as probe readings drift over time.

## How It Works

1. The TDS probe outputs an analog voltage proportional to conductivity.
2. The ADS1115 converts it to a digital value over I2C.
3. The DS18B20 measures water temperature over 1-Wire.
4. The ESP32 converts voltage to EC with temperature compensation, then calculates `TDS (ppm) = EC (µS/cm) / 2`.
5. Values are shown on the OLED, printed to Serial, and synced to Arduino Cloud.

## Sample Output

Typical readings from testing: temperature around 31 °C and TDS around 270 to 310 ppm.

![Dashboard on laptop](images/dashboard_laptop.png)
![Dashboard on mobile](images/dashboard_mobile.png)

## Limitations

- Only **temperature, EC and TDS** are measured. pH, dissolved oxygen and turbidity are not implemented.
- TDS is estimated from EC using a fixed factor of 0.5, so it is approximate and not lab-grade.
- Accuracy depends on calibration and probe condition.
- No machine-learning or anomaly detection is implemented yet.

## Future Work

- Add pH, turbidity and dissolved oxygen sensors
- Threshold alerts (notifications when TDS is too high)
- Data logging and trend analysis, then anomaly detection with ML
- Solar or battery power with deep sleep for remote deployment
- Waterproof enclosure and field testing


# Accident Detection System with GPS & GSM Alert

An Arduino-based accident detection and alert system. It detects a crash or rollover with an MPU6050 motion sensor, sounds a local alarm, and, unless the rider cancels it, sends an SMS with a Google Maps link of the location using a SIM800L GSM module and a NEO-6M GPS module.

Built and simulated in [BenchTwin](https://benchtwin.com).

## Features

- Detects hard impacts (high g-force) and rollovers (sustained tilt)
- 10-second cancel window with a flashing red LED and buzzer
- Push button cancels false alarms
- Reads GPS coordinates with TinyGPSPlus
- Sends an SMS with a Google Maps location link over GSM
- Cooldown after each event to avoid repeated alerts

## Components

| Component | Purpose |
|---|---|
| Arduino Uno R3 | Main controller |
| MPU6050 | Accelerometer / gyroscope for crash detection |
| NEO-6M GPS module | Location |
| SIM800L GSM module | Sends the SMS alert |
| Push button | Cancel false alarm |
| Red LED + 220 Ω resistor | Visual alert |
| Buzzer (active) | Audible alert |
| 1 kΩ + 2 kΩ resistors | Voltage divider for SIM800L RX |
| Separate 3.7–4.2 V supply | Powers the SIM800L (up to 2 A peaks) |

## Wiring

| Part | Pin | Arduino |
|---|---|---|
| MPU6050 | SDA / SCL | A4 / A5 |
| MPU6050 | VCC / GND | 5V / GND |
| NEO-6M | TX | D4 |
| NEO-6M | RX | D3 |
| NEO-6M | VCC / GND | 5V / GND |
| SIM800L | TX | D7 |
| SIM800L | RX | D8 (through voltage divider) |
| SIM800L | VCC / GND | External supply / common GND |
| Push button | One leg / other leg | D2 / GND |
| Red LED | Anode (via 220 Ω) / cathode | D5 / GND |
| Buzzer | + / − | D6 / GND |

Notes:
- The SIM800L RX pin needs a 5 V to 3.3 V divider: D8 → 1 kΩ → SIM800L RX, and 2 kΩ from SIM800L RX → GND.
- Do not power the SIM800L from the Arduino 5 V pin. Add a 1000 µF capacitor across its supply.
- All grounds must be connected together.

## How It Works

1. The MPU6050 is read continuously.
2. A crash is flagged when total acceleration is at least 2.5 g, or when the board stays tilted more than 60° for 2 seconds.
3. The LED and buzzer flash for 10 seconds. Pressing the button cancels the alert.
4. If not cancelled, the GPS position is read and the SIM800L sends an SMS to the emergency contact with a Google Maps link.
5. The system waits 30 seconds before it can trigger again.

## Setup

1. Install the Arduino IDE (or use the BenchTwin code editor).
2. Install the **TinyGPSPlus** library (Mikal Hart), plus an MPU6050 library if you use real hardware.
3. Open `accident_detection.ino`.
4. Replace the phone number in the code:
   ```cpp
   const char PHONE[] = "+91XXXXXXXXXX";
   ```
5. Choose your mode at the top of the sketch:
   ```cpp
   #define IMU_MODE 0     // 0 = fake crash from Serial monitor, 1 = real MPU6050
   #define SIMULATION 1   // 1 = fake GPS coordinates if no fix, 0 = real hardware
   ```
6. Upload the sketch to the Arduino Uno.

## Testing in BenchTwin

- With `IMU_MODE 0`, open the Serial monitor (9600 baud) and send `c` to simulate a crash.
- Do nothing for 10 seconds to see the alert flow end with "SMS sent".
- Press the push button during the countdown to see "Cancelled by user".
- With `IMU_MODE 1`, raise the IMU gravity slider above 2.5 g or tilt it past 60° for 2 seconds.

BenchTwin does not simulate the SIM800L or send real SMS messages, so the SMS step only prints to the Serial monitor in simulation.

## Configuration

| Constant | Default | Meaning |
|---|---|---|
| `IMPACT_G` | 2.5 | g-force that counts as a crash |
| `ROLLOVER_DEG` | 60 | tilt angle that counts as a rollover |
| `ROLLOVER_MS` | 2000 | time tilted before rollover is flagged |
| `CANCEL_WINDOW_MS` | 10000 | time to press the cancel button |
| `COOLDOWN_MS` | 30000 | pause after an event |

## Buzzer Note

The sketch is written for an **active** buzzer (`digitalWrite`). For a passive buzzer, use `tone(BUZ, 2000)` and `noTone(BUZ)` instead.

## Limitations

- Accelerometer thresholds need tuning on real vehicles to avoid false alarms.
- GPS needs a clear view of the sky and can take minutes for a first fix.
- SIM800L requires a SIM with SMS credit and 2G network coverage.

## License

MIT License. Free to use and modify

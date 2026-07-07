# Automatic Fan Speed Control Using Temperature Sensor

An embedded systems project that automatically regulates DC fan speed based on ambient temperature, using an **Arduino UNO** and a **TMP36 temperature sensor**.

## Overview

Manual fan regulation is inefficient — fans are often left running at full speed regardless of actual cooling needs, or forgotten and left on, wasting power. This project replaces manual control with a closed-loop system: a TMP36 sensor continuously measures ambient temperature, the Arduino converts that reading into a temperature value, and the fan speed is adjusted automatically via PWM, switched through an NPN transistor.

As temperature rises, fan speed increases in defined steps; as it falls, the fan slows down or switches off — with no manual intervention required.

## Objectives

- Sense ambient temperature in real time using an embedded system.
- Automatically control DC fan speed proportional to sensed temperature using PWM.
- Eliminate manual fan adjustment and reduce power wastage at low temperatures.
- Simulate and validate circuit behavior across multiple temperature test cases.

## How It Works

The TMP36 sensor outputs an analog voltage linearly proportional to temperature, read on Arduino analog pin `A0`. The Arduino converts this to a Celsius value (0.488 °C per ADC unit) and applies the following control logic to set the PWM duty cycle on the fan:

| Temperature | Fan State | PWM Value | Duty Cycle |
|---|---|---|---|
| Below 23 °C | OFF | 0 | 0% |
| 23 °C – 33 °C | Medium speed | 128 | ≈ 50% |
| 33 °C and above | Full speed | 255 | 100% |

Since an Arduino I/O pin can only safely source ~20–40 mA — far less than a DC motor draws — an **NPN transistor** is used as a current-controlled switch: the PWM signal drives the transistor's base (through a current-limiting base resistor), and the transistor's collector-emitter path switches power to the motor. A **flyback diode** across the motor protects the transistor from the voltage spike generated when the motor's inductive coil is switched off.

The sense → decide → actuate cycle repeats every 500 ms in `loop()`, with each reading logged to the Serial Monitor.

### Why an NPN Transistor

An NPN transistor is used (rather than PNP) because the motor is wired high-side (to +5V) and the transistor switches the low side (to ground). This means a HIGH/PWM signal from the Arduino directly turns the transistor on — matching a microcontroller output that idles LOW and pulses HIGH, with no signal inversion needed.

### Why a Flyback Diode

The motor is an inductive load. When the transistor switches off, the collapsing magnetic field induces a large reverse voltage spike that can destroy the transistor. The flyback diode, wired in reverse bias across the motor, provides a safe path for this stored current to dissipate instead of forcing it through the transistor.

## Components

- Arduino UNO
- TMP36 analog temperature sensor
- NPN transistor (switching stage)
- Flyback (PN-junction) diode
- Base resistor
- DC motor (fan)

## Circuit

The TMP36 sensor connects to Arduino pin `A0`; the PWM output on digital pin `9` drives the transistor base through a current-limiting resistor, which switches the motor circuit, protected by a flyback diode across the motor terminals.

## Arduino Source Code

```cpp
const int tempPin = A0;
const int fanPin = 9;
int tempC;
int pwmValue;

void setup() {
  pinMode(fanPin, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int sensorValue = analogRead(tempPin);
  tempC = sensorValue * 0.488;

  if (tempC < 23) {
    pwmValue = 0;
  }
  else if (tempC >= 23 && tempC < 33) {
    pwmValue = 128;
  }
  else {
    pwmValue = 255;
  }

  analogWrite(fanPin, pwmValue);

  Serial.print("Temperature: ");
  Serial.print(tempC);
  Serial.print(" C | PWM: ");
  Serial.println(pwmValue);

  delay(500);
}
```

## Simulation & Test Results

The circuit was simulated across three temperature conditions to validate each branch of the control logic:

| Condition | Sensed Temperature | Fan State | PWM Value | Duty Cycle |
|---|---|---|---|---|
| Below threshold (cool) | 21 °C | OFF | 0 | 0% |
| Moderate temperature | 26 °C | ON — Medium speed | 128 | ≈ 50% |
| High temperature | 46 °C | ON — Full speed | 255 | 100% |

All three tests confirmed correct transitions between OFF, medium-speed, and full-speed states, matching the Serial Monitor output exactly (e.g. `Temperature: 26 C | PWM: 128`).

## Applications

- Automatic cooling for electronic enclosures and control panels
- Smart home climate control and energy-saving ventilation
- CPU/server cabinet cooling based on real-time temperature
- Industrial machine and motor cooling systems
- Greenhouse and storage room temperature regulation

## Advantages

- Fully automatic — no manual switching required
- Energy efficient — fan runs only when needed, at a speed proportional to demand
- Simple, low-cost hardware built around a widely available microcontroller and sensor
- Easily extendable with additional sensors, an LCD display, or IoT connectivity

## Future Scope

- Add an LCD/OLED display for live temperature and fan status
- Integrate IoT connectivity (e.g., ESP8266/ESP32) for remote monitoring and logging
- Implement PID-based closed-loop control for smoother, continuous speed variation
- Add multiple sensor nodes for zone-wise cooling in larger environments

## Conclusion

This project demonstrates a practical application of embedded systems in real-time environmental monitoring and actuation. By combining a TMP36 sensor, an Arduino UNO, and a transistor-driven PWM stage, the system automatically adjusts fan speed across three temperature bands, validated through simulation at 21 °C, 26 °C, and 46 °C.

## Author

**Pathan Mohammad Shoaib Khan**
B.Tech, Electronics and Communication Engineering

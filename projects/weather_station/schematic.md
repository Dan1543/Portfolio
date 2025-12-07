---
layout: post
title: "Schematic"
parent: "Weather Station"
nav_order: 2
project_url: https://github.com/Dan1543/Weather-Monitor
---
## Hardware Design

For this system, the **ESP32-WROOM-32D** was selected as the main controller. It was chosen not only for its cost effectiveness,but also for its integrated **Wi-Fi and Bluetooth** capabilities and low power **Deep Sleep modes**, which are critical for IoT edge devices sending data to cloud services like **AWS IoT Core**.

For data acquisition, I used the **Bosch BME280** sensor. Unlike simpler alternatives (e.g., DHT11), the BME280 offers superior accuracy and simultaneous measurement of **temperature, humidity, and barometric pressure** via the I2C bus.

![Schematic]({{ site.baseurl }}/assets/images/schematic.png)
*Figure 1: Schematic for the connection of the ESP32-WROOM-32D with the BME280*
{: .text-center .d-block .mt-2 .text-delta }

### Wiring and Configuration
The communication uses the standard I2C protocol. It is important to note that the BME280 operates strictly at **3.3V logic levels**. Connecting it to 5V sources requires a logic level shifter to avoid damage.

| ESP32 Pin | BME280 Pin | Description |
| :--- | :--- | :--- |
| 3V3 | VCC | Power Supply |
| GND | GND | Ground |
| GPIO 21 | SDA | I2C Data |
| GPIO 22 | SCL | I2C Clock |

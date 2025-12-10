---
layout: default
title: "Weather Station"
nav_order: 1
parent: "Projects"
project_url: https://github.com/Dan1543/Weather-Monitor
has_toc: false
---

# Intelligent IoT Weather Monitor
{: .fs-9 }

An end to end IoT solution that captures local environmental data, processes it in the cloud using AWS, and predicts weather conditions using a custom probability model.

![Project Hero Image]({{ site.baseurl }}/assets/images/hero.jpeg){: width="30%" .d-block .mx-auto }

*Figure 1: ntfy.sh mobile app displaying real-time weather alerts from the IoT system*
{: .text-center .d-block .mt-2 .text-delta }

---

## Project Overview
This project bridges the gap between **Embedded Systems** and **Data Science**. Unlike standard weather apps that provide regional forecasts, this system collects hyper local data (temperature, humidity, pressure) to predict micro climate changes specifically for the device location.

The goal was to design a scalable, serverless architecture that could ingest real time sensor data and deliver alerts to the end user with minimal latency.

## Key Features
* **Real time sensing:** High precision environmental monitoring using the **BME280** sensor.
* **Edge Connectivity:** **ESP32** microcontroller handling secure MQTT communication (TLS/SSL).
* **Serverless Backend:** Fully managed **AWS** infrastructure (IoT Core, DynamoDB, Lambda) to reduce maintenance and costs.
* **Predictive Analysis:** Custom Python logic running in the cloud to calculate weather probabilities.
* **Smart Alerts:** Push notifications via **ntfy.sh** only when critical weather changes are detected.

---

## Technical Documentation
Explore the detailed engineering design behind the system:

<div class="d-flex flex-wrap gap-3 my-4">
  <a href="{{ site.baseurl }}/projects/weather_station/hardware_design" class="btn btn-primary fs-5 mr-2">
    Hardware Design
  </a>
  <a href="{{ site.baseurl }}/projects/weather_station/aws_structure" class="btn btn-purple fs-5 mr-2">
    AWS Structure
  </a>
  <a href="{{ site.baseurl }}/projects/weather_station/model_design" class="btn btn-blue fs-5">
    Model Design
  </a>

</div>

---

## Tech Stack

| Domain | Technology Used |
| :--- | :--- |
| **Hardware** | ESP32 WROOM 32D, BME280 (I2C) |
| **Firmware** | Python / MicroPython |
| **Cloud Provider** | Amazon Web Services (AWS) |
| **Cloud Services** | IoT Core, Lambda, DynamoDB, IAM |
| **Protocols** | MQTT, HTTPS, JSON |
| **Notification** | ntfy.sh (Pub/Sub) |

## Results & Performance
The system successfully predicts rain with an accuracy window of 1 hour based on the **XGBoost** model imported to the Lambda function on AWS. By using the **Deep Sleep** capabilities of the ESP32 and the **Event Driven** nature of AWS Lambda, the energy consumption and cloud costs were kept to a minimum, qualifying entirely for the AWS Free Tier.


### Future Improvements
**Hardware Improvements**
* Design a 3D-printed enclosure to house the ESP32, BME280 sensor, and battery pack.
* Implement modular compartments for easy battery replacement and component maintenance.
* Optimize thermal management within the enclosure for sensor accuracy.
* Add weatherproofing features (gaskets, vents) to protect against environmental elements.

**Cloud & Software Improvements**
* Search for a more accurate model for the prediction
* Upgrade to **SageMaker** for more advanced historical data training.
* Change the ntfy.sh for an app for better user interaction
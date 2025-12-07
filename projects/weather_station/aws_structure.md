---
layout: post
title: "AWS Architecture"
parent: "Weather Station"
nav_order: 1
project_url: https://github.com/Dan1543/Weather-Monitor
---

## Cloud Computing Design
For the backend infrastructure, **AWS (Amazon Web Services)** was selected due to its robustness and scalability. The entire architecture is designed to operate within the **AWS Free Tier**, making it cost-effective for personal deployment.

While the current prediction model runs directly on AWS Lambda using a pre-trained Python script, the architecture is designed to be easily upgradeable to **Amazon SageMaker** for more complex model training and deployment in the future.

## Technologies Used
* **AWS IoT Core:** Acts as the MQTT Broker to securely receive data from the ESP32.
* **Amazon DynamoDB:** A NoSQL database used for high-speed storage of time-series weather data.
* **AWS Lambda:** Serverless compute service that processes database streams, runs the prediction model, and handles notifications.
* **ntfy.sh:** An external HTTP-based pub-sub notification service.

## System Architecture
The system follows an **Event-Driven Architecture**. Data flows asynchronously from the edge device (ESP32) to the user notification, triggered by data arrival rather than polling.

![Architecture]({{ site.baseurl }}/assets/images/architecture.png)
*Figure 1: Cloud architecture and data flow*
{: .text-center .d-block .mt-2 .text-delta }

## 1. IoT Core (Ingestion)
IoT Core is the entry point for our data. The ESP32 connects securely using mutual authentication (certificates) and publishes the weather data via **MQTT**.

The data is sent to a specific topic defined in the microcontroller's configuration (e.g., `ESP32/weather-data`).

## 2. DynamoDB (Storage)
Instead of writing code to save data, we use an **IoT Rule** with a DynamoDBv2 action. This rule listens to the MQTT topic and automatically inserts the payload into the database.

**The SQL Selection Query:**
```sql
SELECT *, timestamp() as timestamp FROM 'ESP32/weather-data'
```

> **Note:** The `timestamp()` function adds the server-side time of arrival, which is crucial for time-series analysis.

**Data Requirements:**
The ESP32 payload **must** include a `device_id`. This is used as the **Partition Key** in DynamoDB to efficiently organize data, while the timestamp serves as the **Sort Key**.
* **Required Fields:** `device_id`, `pressure`, `temperature`, `humidity` and all the deltas.
* **Optional Fields:** `dew_point`

> **Note:** The `dew_point` is optional and can be calculated either on the ESP32 or in the Lambda function, depending on your preference.

## 3. AWS Lambda (Processing & Prediction)
The Lambda function is the "brain" of the cloud logic. It is not invoked via HTTP; instead, it is configured with a **DynamoDB Stream Trigger**. This means the function executes automatically **only** when a new record is successfully committed to the database.

### Executing the Model
The Lambda environment includes a custom `model.py` file containing the pre-trained logic. Once triggered:
1.  The function parses the `NewImage` (the latest data) from the DynamoDB Stream.
2.  It inputs the data into the model.
3.  A **Softmax** function calculates the probability of weather conditions for the next hour.

### Sending Notifications (ntfy)
To keep the project lightweight and free, I integrated **ntfy.sh** for push notifications. The user subscribes to a topic generated using the microcontroller's MAC address (e.g., `WeatherStation_A1B2C3`), ensuring privacy and preventing cross-talk between different devices.

**Notification Logic:**
The system assigns priority levels based on the prediction severity:

* **Normal (Clear/Cloudy):** Sends a low-priority notification. It appears in the history but does not trigger a sound/vibration.
* **Critical (Rain/Storm):** Sends a high-priority notification (Push), actively alerting the user to take precautions.
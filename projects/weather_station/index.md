---
layout: post
title: "Weather Station"
nav_order: 1
parent: "Projects"
permalink: /projects/weather-station
project_url: https://github.com/Dan1543/Weather-Monitor
---
# Weather Station Monitoring

```mermaid
graph LR
A[ESP32] -- MQTT --> B((AWS IoT Core))
B --> C{Regla}
C -->|Trigger| D[Lambda]
D --> E[(DynamoDB)]
```
---
layout: post
title: "Weather Station"
parent: "Projects"
nav_order: 1
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
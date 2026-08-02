# iot-anomaly-detector

Simulated IoT sensor pipeline that streams device data through Kafka, detects anomalies in real time using Redis-backed rolling statistics, and pushes alerts through AWS SNS.

## Status

🚧 In development — this repo is being built incrementally. See [Roadmap](#roadmap) for current progress.

## Overview

This project simulates a fleet of IoT devices producing sensor readings (e.g. temperature, vibration) at high frequency, then processes that stream to catch anomalous readings in near real time — without recomputing statistics from scratch on every event.

It's a personal project built to get hands-on with distributed streaming systems and AWS, going beyond framework tutorials into a design with real tradeoffs: partitioning strategy, why a cache belongs in a streaming pipeline, and least-privilege cloud permissions.

## Architecture

```
IoT simulator → Kafka (device-partitioned topic) → Anomaly detector
                                                          │
                                                     reads/writes
                                                          │
                                                        Redis
                                                (rolling stats per device)
                                                          │
                                              ┌───────────┼───────────┐
                                              ▼           ▼           ▼
                                             S3          SNS      CloudWatch
                                        (raw + alert   (anomaly   (metrics
                                            logs)        alerts)    & logs)
```

Everything runs in Docker Compose on a single EC2 instance, scoped with an IAM role limited to the one S3 bucket and one SNS topic this project uses.

## Tech stack

- **Java / Spring Boot** — application services
- **Apache Kafka** — event streaming, partitioned by device ID
- **Redis** — rolling window stats (mean/stddev) per device, avoids recomputing from history on every message
- **AWS** — EC2 (host), S3 (log storage), SNS (alerting), CloudWatch (metrics/logs), IAM (scoped access)
- **Docker Compose** — local/single-instance orchestration

## Project structure

```
src/main/java/com/github/ravikr309/iotanomalydetector/
├── config/           # Kafka, Redis, and AWS client configuration
├── controller/        # REST endpoints (live stats, simulation control)
├── service/           # Interfaces
│   └── impl/           # Implementations
├── kafka/             # Producer and consumer
├── repository/        # Redis-backed stats repository
├── model/              # SensorReading, DeviceStats, AnomalyAlert
├── exception/          # Custom exceptions + global handler
└── util/               # Rolling stats calculation
```

## Getting started

Prerequisites: Java 17+, Docker, Docker Compose, an AWS account with an IAM user/role scoped to S3 + SNS.

```bash
git clone https://github.com/ravikr309/iot-anomaly-detector.git
cd iot-anomaly-detector
docker-compose up -d      # starts Kafka + Redis
./mvnw spring-boot:run
```

Configuration (Kafka bootstrap servers, Redis host, AWS region/bucket/topic) lives in `application.yml` — copy `application-example.yml` and fill in your own values.

## Roadmap

- [ ] Device simulator producing to Kafka
- [ ] Kafka consumer + Redis rolling stats
- [ ] Anomaly detection logic
- [ ] SNS alerting integration
- [ ] S3 log archival
- [ ] CloudWatch metrics via Spring Boot Actuator
- [ ] Load test with concurrent simulated devices

## License

MIT

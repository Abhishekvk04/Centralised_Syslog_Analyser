# Centralized Syslog Analyzer for Multiple Linux Systems 
## OpenSearch Syslog Pipeline

A comprehensive, production-ready logging pipeline using Docker Compose, integrating syslog-ng, Filebeat, Logstash, OpenSearch, and OpenSearch Dashboards. This stack ingests syslog messages (TCP/UDP), parses and enriches them, and stores them in OpenSearch for search and visualization.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Components](#components)
- [Directory Structure](#directory-structure)
- [Prerequisites](#prerequisites)
- [Setup & Usage](#setup--usage)
- [Testing the Pipeline](#testing-the-pipeline)
- [Key Configuration Files](#key-configuration-files)
- [OpenSearch Access](#opensearch-access)
- [Troubleshooting](#troubleshooting)
- [Notes](#notes)

---

## Architecture Overview

Syslog Clients  
│  
▼  
**syslog-ng** (UDP/TCP 514)  
│  
▼  
**filebeat** (reads JSON logs)  
│  
▼  
**logstash** (parsing, enrichment)  
│  
▼  
**OpenSearch** (storage, search)  
│  
▼  
**OpenSearch Dashboards** (visualization)


---

## Components

- **syslog-ng**: Receives syslog messages over TCP/UDP, writes them as JSON for Filebeat.
- **Filebeat**: Reads JSON logs, forwards to Logstash.
- **Logstash**: Parses, enriches, and forwards logs to OpenSearch.
- **OpenSearch**: Stores and indexes logs.
- **OpenSearch Dashboards**: Web UI for searching and visualizing logs.

---

## Directory Structure

```text
/opensearch-pipeline/
├── docker-compose.yml
├── filebeat/
│   └── filebeat.yml
├── logstash/
│   ├── config/
│   │   └── logstash.yml
│   ├── pipeline/
│   │   └── logstash.conf
│   ├── templates/
│   │   └── syslog-template.json
│   └── data/
├── syslog-ng/
│   └── config/
│       └── syslog-ng.conf
├── logs/
├── shared-logs/
│   └── filebeat-input.json
├── Dockerfile
├── Dockerfile.client
└── test_syslog.py
```


---

## Prerequisites

- Docker & Docker Compose installed
- Ports 514 (TCP/UDP), 5044, 5000, 5601, 9200, 9600 available
- At least 4GB RAM recommended

---

## Setup & Usage

1. **Build and Start the Stack**
  ```
  docker-compose down && docker-compose up -d --build
  ```

2. **Send Test Syslog Messages**

- **UDP:**
  ```
  logger -n 127.0.0.1 -P 514 -d "UDP test log message from logger"
  ```
- **TCP:**
  ```
  logger -T -n 127.0.0.1 -P 514 "TCP test log message from logger"
  ```

3. **Check syslog-ng Output**

- View syslog-ng logs:
  ```
  docker exec syslog-ng cat /var/log/messages.log | grep test
  ```
- View JSON logs for Filebeat:
  ```
  docker exec syslog-ng cat /var/log/syslog-ng-for-filebeat/filebeat-input.json | grep test
  ```

4. **Verify Log Ingestion in OpenSearch**

- Search for test logs:
  ```
  curl -XGET 'https://localhost:9200/syslog-/_search?q=message:*test&pretty' -u admin:MyStrongPassword123! --insecure
  ```
- Count logs:
  ```
  docker exec -it opensearch curl -k https://localhost:9200/syslog-\*/_count -u admin:MyStrongPassword123!
  ```
- Get all logs:
  ```
  curl -XGET 'https://localhost:9200/syslog-*/_search?pretty' -u admin:MyStrongPassword123! --insecure
  ```

5. **Continuous Log Testing**

- Activate Python virtual environment and run test script:
  ```
  source venv/bin/activate
  python3 test_syslog.py
  ```
- Use the above steps to verify logs.

---

## Key Configuration Files

- **docker-compose.yml**: Defines all services, networks, and volumes.
- **filebeat/filebeat.yml**: Filebeat input/output configuration.
- **logstash/pipeline/logstash.conf**: Logstash pipeline for parsing and enriching logs.
- **logstash/templates/syslog-template.json**: OpenSearch index template for syslog logs.
- **syslog-ng/config/syslog-ng.conf**: syslog-ng sources, destinations, and log paths.
- **test_syslog.py**: Python script to send test syslog messages via TCP/UDP.

---

## OpenSearch Access

- **OpenSearch**:  
URL: `https://localhost:9200/`  
User: `admin`  
Password: `MyStrongPassword123!`

- **OpenSearch Dashboards**:  
URL: `http://localhost:5601/`  
User: `admin`  
Password: `MyStrongPassword123!`

---

## Troubleshooting

- **Health Checks**: All containers have health checks; use `docker-compose ps` to view status.
- **Log Files**:  
- syslog-ng logs: `/var/log/messages.log`  
- JSON logs for Filebeat: `/var/log/syslog-ng-for-filebeat/filebeat-input.json`
- **Dead Letter Queue**: Logstash writes failed documents to `/usr/share/logstash/data/dead_letter_queue/`.

---

## Notes

- The pipeline supports both TCP and UDP syslog messages.
- All logs are indexed in OpenSearch with the pattern `syslog-*`.
- The system is designed for extensibility and production-readiness.
- For custom parsing or enrichment, modify `logstash.conf` or the syslog-ng template.

---


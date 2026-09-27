# Floci Architecture

Floci is an AWS-compatible local runtime built as a Java application. It provides local cloud simulation for defensive security and cloud infrastructure testing.

## Ports and Services
- **AWS API:** `http://127.0.0.1:4566`
- **Console:** `http://127.0.0.1:4500/console/aws`
- **RDS Proxy Ports:** `7001-7099`
- **Elasticsearch/OpenSearch Ports:** `9200-9299`
- **Redis/ElastiCache Ports:** `6379-6399`

## Docker Compose Configuration
The lab environment is orchestrated via Docker Compose:

```yaml
services:
  floci:
    build:
      context: .
      dockerfile: docker/Dockerfile
    tty: true
    ports:
      - "4566:4566"
      - "6379-6399:6379-6399"
      - "7001-7099:7001-7099"
      - "9200-9299:9200-9299"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/app/data
    environment:
      FLOCI_SERVICES_DOCKER_NETWORK: floci_default
      FLOCI_SERVICES_RDS_PROXY_BASE_PORT: "7001"
      # FLOCI_STORAGE_HOST_PERSISTENT_PATH is no longer needed.
      # Floci now uses named Docker volumes by default.
      # If you were using this env var for persistence, migrate: docker volume ls -f label=floci=true
      FLOCI_HOSTNAME: floci
      FLOCI_BASE_URL: http://floci:4566
      # localhost.floci.io resolves everywhere: public DNS -> 127.0.0.1 on the host
      # (reaching the published proxy ports), the network alias below inside Compose.
      FLOCI_SERVICES_ELASTICACHE_CLUSTER_ANNOUNCE_HOSTNAME: localhost.floci.io
      FLOCI_SERVICES_LAMBDA_HOT_RELOAD_ENABLED: "true"
      # Match the compatibility workflow (.github/workflows/compatibility.yml) so the
      # local Docker test run exercises the same Floci configuration as CI.
      # TLS proxy serves HTTP and HTTPS on the same port 4566 (first-byte sniffing),
      # so no extra port mapping is needed — this is what makes the TLS/HTTPS suites pass.
      FLOCI_TLS_ENABLED: "true"
    networks:
      floci_default:
        aliases:
          - localhost.floci.io

networks:
  floci_default:
    name: floci_default

```

## Backend Services
The backend is primarily written in Java (e.g. `CloudWatchLogsService.java`) which simulates the AWS APIs locally.

# Telemetry Monitoring Stack

A comprehensive monitoring solution using Prometheus, Thanos, Grafana, and Node Exporter. This stack provides metrics collection, long-term storage, and visualization capabilities.

## Stack Components

- **Prometheus**: Metrics collection and storage
- **Thanos**: Long-term metrics storage and global query view
- **Grafana**: Metrics visualization and dashboarding
- **Node Exporter**: Host metrics collection
- **[Sigil](https://github.com/Landfall-SMP/Sigil)**: Custom Velocity metrics plugin (external)

## Prerequisites

- Docker
- Docker Compose
- At least 4GB of RAM
- Linux host (for Node Exporter)
- Sufficient disk space for metrics storage (recommend at least 50GB)

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/Landfall-SMP/Landfall-Telemetry.git
   cd Landfall-Telemetry
   ```

2. Create the data directories:
   ```bash
   mkdir -p data/prometheus data/grafana
   ```

3. Set up environment variables (optional):
   ```bash
   export GF_SECURITY_ADMIN_PASSWORD=your_secure_password  # Default: admin
   ```

4. Start the stack:
   ```bash
   docker-compose up -d
   ```

The stack will automatically:
- Initialize data directories with correct permissions
- Start all services in the correct order
- Begin collecting metrics

## Storage Configuration

This stack uses host filesystem storage for better performance and scalability:

- **Prometheus data**: `./data/prometheus`
  - Contains TSDB data
  - Automatically configured with correct permissions (uid 65534)
  - Recommend monitoring disk usage and growth

- **Grafana data**: `./data/grafana`
  - Contains dashboards, users, and plugins
  - Automatically configured with correct permissions (uid 472)

The initialization of data directories and permissions is handled automatically by the `prometheus-init` service in the Docker Compose stack.

Benefits of filesystem storage:
- Better I/O performance
- Easier backup and restore
- Independent scaling of storage
- Persistence across container rebuilds
- Direct access for maintenance

## Access Points

- **Grafana**: `http://localhost:2000`
  - Default credentials: 
    - Username: `admin`
    - Password: `admin` (or value of `GF_SECURITY_ADMIN_PASSWORD`)

- **Prometheus**: `http://localhost:9090`
- **Thanos Query**: `http://localhost:19090`
- **Node Exporter**: `http://localhost:9100/metrics`

## Component Details

### Prometheus
- Scrapes metrics
- Configured targets:
  - Prometheus itself
  - Node Exporter
  - Sigil (external service)
- Data stored in host filesystem: `./data/prometheus`

### Thanos
Consists of two components:
1. **Sidecar**:
   - Connects to Prometheus
   - Handles long-term storage
   - Exposes StoreAPI
   - Ports: 19191 (HTTP), 10901 (gRPC)

2. **Query**:
   - Global view of metrics
   - Deduplicates metrics from Prometheus/sidecar
   - Ports: 19090 (HTTP), 19091 (gRPC)

### Node Exporter
- Collects host metrics
- Access to host filesystem through volume mounts
- Exposes metrics on port 9100

### Grafana
- Modern visualization platform
- Pre-configured with Prometheus/Thanos datasources
- Runs on port 2000
- Data stored in host filesystem: `./data/grafana`

## Network Architecture

All services are connected through a dedicated Docker network named `monitoring`. The stack uses:
- Internal service discovery (Docker DNS)
- Host machine access via `host.docker.internal`
- Bridge network for container communication

## Storage Architecture

The stack uses host filesystem storage for better performance and scalability:

### Data Directories
- `./data/prometheus/`: Prometheus TSDB storage
  - Contains all metrics data
  - Accessed by both Prometheus and Thanos Sidecar
  - Initialized with UID 65534 (nobody user)

- `./data/grafana/`: Grafana storage
  - Contains dashboards, users, and plugins
  - Initialized with UID 472 (grafana user)

### Backup Process
```bash
# Stop the stack first
docker-compose down

# Backup data directories
tar czf backup-$(date +%Y%m%d).tar.gz data/

# Restart the stack
docker-compose up -d
```

## Configuration Files

- `prometheus/prometheus.yml`: Prometheus scrape configuration
- `thanos/bucket.yaml`: Thanos object storage configuration
- `grafana/`: Grafana provisioning and configuration files

## Maintenance

### Storage Management
Monitor disk usage of data directories:
```bash
du -sh data/prometheus
du -sh data/grafana
```

### Backup
Back up the data directories:
```bash
# Stop the stack first
docker-compose down

# Backup data directories
tar czf backup-$(date +%Y%m%d).tar.gz data/

# Restart the stack
docker-compose up -d
```

### Troubleshooting

1. **Prometheus not scraping:**
   ```bash
   docker logs prometheus
   ```

2. **Node Exporter issues:**
   ```bash
   docker logs node-exporter
   ```

3. **Thanos connectivity:**
   ```bash
   docker logs thanos-sidecar
   docker logs thanos-query
   ```

## Security Notes

- All ports are exposed on localhost only
- Grafana password should be changed from default
- Prometheus admin API is enabled for Thanos integration
- Volume permissions are handled by an init container

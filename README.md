# Blockchain Node Monitor

A comprehensive monitoring tool for blockchain nodes that supports EVM-compatible chains, Cosmos-based networks, and Ethereum Beacon Chain. This tool provides Prometheus metrics for monitoring node health, block processing delays, endpoint response times, and connection status.

## Features

- **Multi-chain Support**: Monitor EVM, Cosmos/CometBFT, and Ethereum Beacon Chain nodes
- **Prometheus Metrics**: Export detailed metrics for monitoring and alerting
- **Real-time Monitoring**: Subscribe to new blocks and track processing delays
- **Health Checks**: Monitor endpoint availability and response times
- **Grafana Integration**: Pre-built dashboard for visualization

## Supported Chains

### EVM Compatible Chains
- Ethereum Mainnet/Testnets
- Polygon
- Binance Smart Chain
- Arbitrum
- Optimism
- Any EVM-compatible chain with standard JSON-RPC interface

### Cosmos Ecosystem
- Cosmos Hub
- Osmosis
- Juno
- Any Cosmos SDK-based chain with CometBFT consensus

### Ethereum Beacon Chain
- Ethereum 2.0 Beacon Chain
- Compatible with major consensus clients (Lighthouse, Prysm, Teku, Nimbus)

## Metrics Overview

The tool exports the following Prometheus metrics:

### Block Processing Metrics
- `blockchain_node_last_block_timestamp_seconds`: Timestamp of the last processed block
- `blockchain_node_block_processing_delay_seconds`: Delay between block creation and processing
- `blockchain_node_block_processing_delay_histogram_seconds`: Histogram of block processing delays

### Node Health Metrics
- `blockchain_node_health_status`: Health status of node endpoints (1=healthy, 0=unhealthy)
- `blockchain_node_endpoint_response_time_milliseconds`: Current response time for endpoints
- `blockchain_node_endpoint_response_time_histogram_milliseconds`: Histogram of response times

### Connection Metrics
- `blockchain_node_rpc_connections_count`: Total number of RPC connection attempts

All metrics include labels for:
- `chain_name`: Name of the blockchain
- `hostname`: Node identifier
- `chain_id`: Chain ID (for informational metrics)
- `node_version`: Node software version (for informational metrics)
- `endpoint_type`: Type of endpoint being monitored
- `connection_type`: Type of connection (http/ws)
- `result`: Result of connection attempt (success/fail)

## Installation

### Prerequisites
- Go 1.22 or later
- Access to blockchain node RPC endpoints

### Build from Source
```bash
git clone https://github.com/ronnynth/chainmonitor.git
cd chainmonitor
go mod download
go build -o chainmonitor main.go
```

### Using Docker
```bash
docker build -t chainmonitor .
docker run -v $(pwd)/config.yaml:/app/config.yaml -p 3002:3002 chainmonitor
```

## Configuration

Create a `config.yaml` file based on the provided example:

```yaml
# EVM-compatible chains
evm:
  - hostname: "ethereum-mainnet"
    http_url: "https://mainnet.infura.io/v3/YOUR_PROJECT_ID"
    ws_url: "wss://mainnet.infura.io/ws/v3/YOUR_PROJECT_ID"
    chain_name: "ethereum"
    protocol_name: "ethereum
    check_second: 5

# Cosmos-based chains
cometbft:
  - hostname: "cosmos-hub"
    http_url: "https://rpc.cosmos.network:443"
    chain_name: "cosmos"
    protocol_name: "cosmos"
    ws_endpoint: "/websocket"
    check_second: 5

# Ethereum Beacon Chain
beacon:
  - hostname: "ethereum-beacon"
    http_url: "https://beacon-api-endpoint"
    chain_name: "ethereum"
    protocol_name: "ethereum"
    check_second: 15
```

### Configuration Parameters

#### Common Parameters
- `hostname`: Unique identifier for the node
- `chain_name`: Name of the blockchain network
- `chain_id`: Chain identifier (auto-detected if empty)
- `node_version`: Node software version (auto-detected if empty)
- `check_second`: Health check interval in seconds

#### EVM-specific Parameters
- `http_url`: HTTP JSON-RPC endpoint
- `ws_url`: WebSocket JSON-RPC endpoint

#### Cosmos-specific Parameters
- `http_url`: CometBFT RPC endpoint
- `ws_endpoint`: WebSocket endpoint path (default: "/websocket")

#### Beacon Chain-specific Parameters
- `http_url`: Beacon API endpoint

## Usage

### Running the Monitor

```bash
# Using default config file (./config.yaml)
./chainmonitor

# Using custom config file
./chainmonitor -conf /path/to/config.yaml -logtostderr=true -v=5
```

### Command Line Options
- `-conf`: Path to configuration file (default: "./config.yaml")

### Accessing Metrics

Once running, metrics are available at:
- **Metrics endpoint**: `http://localhost:3002/metrics`
- **Health check**: Monitor logs for connection status
- **Debug endpoint**: `http://localhost:6062/debug/pprof/` (pprof profiling)

## Monitoring Setup

### Prometheus Configuration

Add the following to your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'blockchain-nodes'
    static_configs:
      - targets: ['localhost:3002']
    scrape_interval: 15s
    metrics_path: /metrics
```

### Alerting Rules

Example Prometheus alerting rules:

```yaml
groups:
  - name: blockchain-nodes
    rules:
      - alert: NodeDown
        expr: blockchain_node_health_status == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Blockchain node {{ $labels.hostname }} is down"
          
      - alert: HighBlockDelay
        expr: blockchain_node_block_processing_delay_seconds > 60
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High block processing delay on {{ $labels.hostname }}"
          
      - alert: HighResponseTime
        expr: blockchain_node_endpoint_response_time_milliseconds > 5000
        for: 3m
        labels:
          severity: warning
        annotations:
          summary: "High response time on {{ $labels.hostname }}"
```

## Grafana Dashboard

A pre-configured Grafana dashboard is available in `grafana-dashboard.json`. Import it into your Grafana instance to visualize:

- Node health status
- Block processing delays
- Endpoint response times
- Connection success rates
- Historical trends


### Dashboard Features
- **Overview Panel**: Summary of all monitored nodes
- **Health Status**: Real-time health indicators
- **Performance Metrics**: Response times and processing delays
- **Network Statistics**: Block heights and sync status
- **Alerts**: Visual indicators for issues

## Development

### Project Structure
```
chainmonitor/
├── base/                   # Core metrics definitions
├── beacon/                 # Ethereum Beacon Chain implementation
├── cometbft/               # Cosmos/CometBFT implementation
├── conf/                   # Configuration structures
├── evm/                    # EVM chain implementation
├── sched/                  # Scheduler and controller
├── config.yaml.example     # Configuration template
├── grafana-dashboard.json  # Grafana dashboard
└── main.go                 # Application entry point
```

### Adding New Chain Types

1. Create a new package in the project root
2. Implement the `base.CheckerTrait` interface
3. Add configuration struct to `conf/conf.go`
4. Register the new checker in `sched/sched.go`

### Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Troubleshooting

### Common Issues

**Connection Failures**
- Verify RPC endpoint URLs are correct and accessible
- Check firewall settings and network connectivity
- Ensure API keys are valid (for services like Infura)

**High Memory Usage**
- Reduce check intervals in configuration
- Monitor with pprof endpoint: `http://localhost:6062/debug/pprof/`

**Missing Metrics**
- Verify Prometheus can access the metrics endpoint
- Check logs for initialization errors
- Ensure configuration syntax is correct

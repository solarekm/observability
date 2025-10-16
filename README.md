# OpenTelemetry Observability

This repository contains a custom OpenTelemetry Collector build with specific components for observability needs.

## Project Structure

```
observability/
├── README.md                                    # This file
├── build/                                       # Build configuration files
│   ├── builder-config.yaml                     # OCB configuration
│   ├── collector-config.yaml                   # Collector runtime configuration
│   ├── Dockerfile                              # Container build definition
│   ├── .dockerignore                           # Docker build optimization
│   └── README.md                               # Build-specific documentation
├── .github/workflows/                           # GitHub Actions workflows
│   └── build-collector.yml                     # Build automation
├── Dynatrace-opentelemetry-collector-contrib/   # Dynatrace components
└── opentelemetry-collector-contrib/             # Standard OTel components
```

## Custom Collector Components

This custom collector includes the following components:

### From opentelemetry-collector-contrib (local):
- **transformprocessor** - Data transformation capabilities
- **filterprocessor** - Data filtering capabilities

### From Dynatrace-opentelemetry-collector-contrib (local):
- **dynatracereceiver** - Dynatrace metrics and data receiver

### External dependencies:
- **debugexporter** - Debug output for development and testing  
- **otlpreceiver** - OTLP protocol receiver (gRPC and HTTP)
- **batchprocessor** - Batch processing for performance

## Building the Collector

### Prerequisites
- Go 1.25.3+
## Building the Collector

### Prerequisites
- GitHub account with access to this repository
- AWS ECR repository set up  
- GitHub Secrets configured (see below)

### Cloud Build Only
This project is designed for cloud-based builds using GitHub Actions. All builds happen in the cloud - no local development environment is required.

### GitHub Actions Build
The collector is built automatically using GitHub Actions. The workflow builds both binary and Docker images.

**Required GitHub Secrets:**
- `AWS_ACCESS_KEY_ID` - AWS access key for ECR
- `AWS_SECRET_ACCESS_KEY` - AWS secret key for ECR  
- `AWS_REGION` - AWS region (e.g., `us-east-1`)
- `AWS_ECR_REGISTRY` - Full ECR URI (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/otelcol-dynatrace`)

To trigger the build:
1. Go to the "Actions" tab in GitHub
2. Select "Build OpenTelemetry Collector"
3. Click "Run workflow"
4. Choose build type: `binary`, `docker`, or `both`
**Build Outputs:**
- Binary artifacts stored in GitHub Actions (name: `otelcol-dynatrace-linux-amd64-{timestamp}`)
- Docker images pushed to AWS ECR with timestamp tags (format: `YYYYMMDD-HHMMSS`)

## Configuration

The collector configuration is located in `build/collector-config.yaml`. This includes:
- OTLP receivers on standard ports (4317 gRPC, 4318 HTTP)
- Transform and filter processors
- Debug exporter for development
- Dynatrace receiver configuration

## Usage

### Running with Docker (from ECR)
```bash
# Login to ECR (replace with your registry)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Run latest
docker run -p 4317:4317 -p 4318:4318 123456789012.dkr.ecr.us-east-1.amazonaws.com/otelcol-dynatrace:latest

# Run specific version  
docker run -p 4317:4317 -p 4318:4318 123456789012.dkr.ecr.us-east-1.amazonaws.com/otelcol-dynatrace:20251016-143022
```

### Running downloaded binary
```bash
# Download from GitHub Actions artifacts
./otelcol-dynatrace --config build/collector-config.yaml
```

### Available Build Options
When triggering the GitHub Actions workflow:
- `binary` - Build Linux binary only
- `docker` - Build and push Docker image to ECR only  
- `both` - Build both binary and Docker image (default)

## Configuration

The collector configuration is located in `build/collector-config.yaml`. This includes:
- OTLP receivers on standard ports (4317 gRPC, 4318 HTTP)
- Transform and filter processors
- Debug exporter for development
- Dynatrace receiver configuration

## Usage

### Running with Docker (from ECR)
```bash
# Login to ECR (replace with your registry)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Run latest
docker run -p 4317:4317 -p 4318:4318 123456789012.dkr.ecr.us-east-1.amazonaws.com/otelcol-custom:latest

# Run specific version
docker run -p 4317:4317 -p 4318:4318 123456789012.dkr.ecr.us-east-1.amazonaws.com/otelcol-custom:20251016-143022
```

### Running locally built image
```bash
# Using make
make run-docker

# Manual
docker run -p 4317:4317 -p 4318:4318 -p 8888:8888 otelcol-custom:latest
```

### Running locally
```bash
# Using make
cd build && make run

# Manual
cd build && ./otelcol-custom/otelcol-custom --config collector-config.yaml
```

### Available Make Targets
- `make help` - Show all available targets
- `make build` - Build the collector binary
- `make build-docker` - Build Docker image
- `make test` - Test the built binary
- `make clean` - Clean build artifacts
- `make run` - Run collector locally
- `make run-docker` - Run collector in Docker
- `make dev` - Quick development build and test
- `make all` - Build everything and run all tests

## Development

This repository is set up for developing and testing custom OpenTelemetry components. The structure allows for:
- Easy component development and testing
- Containerized deployment
- CI/CD automation
- Integration with existing observability infrastructure

## License

This project follows the licensing of the upstream OpenTelemetry projects.
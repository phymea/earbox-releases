# Earbox Analysis Pipeline

Automated maize ear phenotyping pipeline for high-throughput image analysis.

## 🚀 Quick Start

### Using Docker

```bash
# Pull the latest version
docker pull ghcr.io/phymea/earbox:latest

# Run the pipeline
docker run \
  -v $(pwd)/input:/app/input \
  -v $(pwd)/output:/app/output \
  ghcr.io/phymea/earbox:latest \
  /app/input
```

# Earbox Analysis Pipeline

Automated high-throughput phenotyping pipeline for maize ear image analysis.

---

## 📦 Latest Release

**[Download v1.0.0-rc1](https://github.com/phymea/earbox-releases/releases/latest)** | **[All Releases](https://github.com/phymea/earbox-releases/releases)** | **[Download Weights](https://github.com/phymea/earbox-releases/releases/download/weights-v1.0/mrcnn_eb_weights_31032020.h5)**

---

## 🐳 Docker Images

```bash
# GPU version (recommended for production)
docker pull ghcr.io/phymea/earbox:latest
docker pull ghcr.io/phymea/earbox:latest-gpu

# CPU version (for testing/development)
docker pull ghcr.io/phymea/earbox:latest-cpu
```

**📖 Docker usage:** See [GHCR Package Documentation](https://github.com/phymea/earbox/pkgs/container/earbox)

---

## 🚀 Quick Start (EC2 Runner)

### Prerequisites
- AWS account with configured credentials ([AWS CLI setup guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html))
- S3 buckets for input images and output results
- Python 3.10+ installed locally

### Installation & Setup (One-Time)

```bash
# 1. Download and extract
wget https://github.com/phymea/earbox-releases/releases/download/v1.0.0-rc1/earbox-runner-v1.0.0-rc1.tar.gz
tar -xzf earbox-runner-v1.0.0-rc1.tar.gz && cd runner

# 2. Set up Python environment
./setup_runner.sh
source .venv/bin/activate  # Or: .venv\Scripts\activate on Windows

# 3. Set up AWS IAM roles (auto-creates runner-config.yaml)
python setup_instance_role.py --region aws-region --s3-buckets my-input-bucket my-output-bucket
python setup_automation_role.py --config runner-config.yaml
```

### Run the Pipeline

```bash
# Basic: Run with session-specific input prefix
python run_ec2_instance.py --input-prefix earbox-session-2024-10-20/

# Override instance type (e.g., switch to CPU)
python run_ec2_instance.py \
  --input-prefix data/ \
  --instance-type t3.xlarge \
  --ami-id ami-01dad638e8f31ab9a \
  --docker-image ghcr.io/phymea/earbox:latest-cpu

# Use Docker image stored in ECR
python run_ec2_instance.py \
  --input-prefix data/ \
  --docker-image 123456789.dkr.ecr.eu-north-1.amazonaws.com/earbox:latest
```

**Note:** Model weights (~170MB) are automatically downloaded on first run. To pre-download:
```bash
mkdir -p weights
wget -O weights/mrcnn_eb_weights_31032020.h5 \
  https://github.com/phymea/earbox-releases/releases/download/weights-v1.0/mrcnn_eb_weights_31032020.h5
```

### Verify Execution

```bash
# Check CloudWatch logs
python util_scripts/view_logs.py --job-id <JOB_ID_FROM_OUTPUT>

# Download results from S3
aws s3 sync s3://your-output-bucket/results/ ./local-results/
```

---

## ⚙️ Customization

### Customize Instance Type or Region

The `setup_instance_role.py` script auto-creates `runner-config.yaml` with GPU defaults. To customize:

```bash
# Edit the auto-generated config
nano runner-config.yaml

# Change:
# - aws.region (default: eu-north-1)
# - aws.instance_type (default: g4dn.xlarge for GPU, or t3.xlarge for CPU)
# - aws.ami_id (default: ami-04020d21cabbb9d43 for GPU)
# - docker.image (default: latest-gpu, or latest-cpu)

# Then run normally
python run_ec2_instance.py --input-prefix data/
```

### Using Private Docker Images (ECR)

To mirror the public GHCR image to your private AWS ECR:

```bash
# 1. Create ECR repository (one-time)
aws ecr create-repository --repository-name earbox --region eu-north-1

# 2. Login to ECR
aws ecr get-login-password --region eu-north-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.eu-north-1.amazonaws.com

# 3. Pull from GHCR, tag for ECR, and push
docker pull ghcr.io/phymea/earbox:latest-gpu
docker tag ghcr.io/phymea/earbox:latest-gpu \
  123456789.dkr.ecr.eu-north-1.amazonaws.com/earbox:latest
docker push 123456789.dkr.ecr.eu-north-1.amazonaws.com/earbox:latest

# 4. Update runner-config.yaml:
#   docker:
#     image: 123456789.dkr.ecr.eu-north-1.amazonaws.com/earbox:latest
#     registry_type: ecr
#     ecr_auth: true
```

**Or override via CLI:**
```bash
python run_ec2_instance.py \
  --docker-image 123456789.dkr.ecr.eu-north-1.amazonaws.com/earbox:latest \
  --input-prefix data/
```

**Why use ECR?** Private ECR can improve security, control access, reduce internet egress costs, and ensure image availability.

### Custom Pipeline Settings

To customize analysis parameters (segmentation threshold, output units, etc.):

```bash
# 1. Copy example pipeline config
cp pipeline-config.example.yaml pipeline-config.yaml

# 2. Edit pipeline-config.yaml to adjust:
#   - Segmentation confidence threshold
#   - Output units (mm or pixels)
#   - PDF generation settings
#   - Reporting options

# 3. Set in runner-config.yaml:
#   pipeline:
#     config_file: "pipeline-config.yaml"
```

### AWS Credentials Setup

The runner requires AWS credentials with permissions for:
- **S3**: Read from input bucket, write to output bucket
- **EC2**: Launch instances, create security groups
- **IAM**: Pass roles to EC2 instances
- **CloudWatch**: Write logs

**Configure credentials:**
```bash
# Option 1: AWS CLI (recommended)
aws configure
# Enter: Access Key ID, Secret Access Key, Region (e.g., eu-north-1)

# Option 2: Environment variables
export AWS_ACCESS_KEY_ID="your-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="eu-north-1"
```

**Documentation:** [AWS CLI Configuration Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)

---

## 💻 System Requirements

### Local Development (Docker)
- **CPU**: 4+ cores recommended
- **RAM**: 16GB recommended
- **GPU** (optional): NVIDIA GPU with CUDA 11.8 support for faster processing
- **Storage**: ~5GB for Docker image + model weights

### EC2 Runner (Production)
- **Recommended instances:**
  - CPU: `t3.xlarge`, `c5.2xlarge`
  - GPU: `g4dn.xlarge`, `g4dn.2xlarge` (recommended for production)
- **AMI**: Amazon Linux 2 or Ubuntu 22.04
- **Region**: Any AWS region with GPU availability (check [AWS regional services](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/))

---

## 🐛 Issues & Support

**Report issues:** [GitHub Issues](https://github.com/phymea/earbox-releases/issues)




## 📄 License

This software is licensed under the BSL 1.1 license. See [LICENSE.md](LICENSE.md) for full terms.

---

## 🔗 Links

- **Docker Images:** [ghcr.io/phymea/earbox](https://github.com/phymea/earbox/pkgs/container/earbox)
- **Model Weights:** [Download](https://github.com/phymea/earbox-releases/releases/download/weights-v1.0/mrcnn_eb_weights_31032020.h5)
- **All Releases:** [Releases Page](https://github.com/phymea/earbox-releases/releases)
- **AWS Documentation:** [EC2](https://docs.aws.amazon.com/ec2/) | [S3](https://docs.aws.amazon.com/s3/) | [IAM](https://docs.aws.amazon.com/iam/)

---

**Version:** v1.0.0-rc1 | **Build Date:** 2025-10-20

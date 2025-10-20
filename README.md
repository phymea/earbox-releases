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

### Installation

```bash
# 1. Download and extract latest release
wget https://github.com/phymea/earbox-releases/releases/download/v1.0.0-rc1/earbox-runner-v1.0.0-rc1.tar.gz
tar -xzf earbox-runner-v1.0.0-rc1.tar.gz
cd runner

# 2. Download model weights
mkdir -p weights
wget -O weights/mrcnn_eb_weights_31032020.h5 \
  https://github.com/phymea/earbox-releases/releases/download/weights-v1.0/mrcnn_eb_weights_31032020.h5

# 3. Set up Python environment
./setup_runner.sh
source .venv/bin/activate  # Or: .venv\Scripts\activate on Windows

# 4. Create runner config for IAM setup
cp runner-config.example.yaml runner-config.yaml
# Edit runner-config.yaml: set your AWS region and automation_role_arn placeholder

# 5. Set up AWS IAM roles (one-time setup)
python setup_instance_role.py --config runner-config.yaml
# Copy the Role ARN from output, then:
python setup_automation_role.py --instance-profile-role-arn <ROLE_ARN_FROM_ABOVE> --update-config
# The runner-config.yaml will be updated with the automation_role_arn from this output

# 6. Run the pipeline (GPU instance, default g4dn.xlarge)
python run_ec2_instance.py \
  --input-bucket my-input-bucket \
  --input-prefix earbox-session/ \
  --output-bucket my-output-bucket
```

### Verify Execution

```bash
# Check CloudWatch logs
python util_scripts/view_logs.py --job-id <JOB_ID_FROM_OUTPUT>

# Download results from S3
aws s3 sync s3://your-output-bucket/results/ ./local-results/
```

---

## ⚙️ Configuration

### Custom Runner Settings

To customize EC2 instance type, Docker image, or region, edit `runner-config.yaml`:

```yaml
aws:
  region: eu-north-1
  instance_type: g4dn.xlarge  # Or: t3.xlarge (CPU), g4dn.2xlarge (faster GPU)
  ami_id: ami-04020d21cabbb9d43  # Deep Learning AMI for GPU (Or: ami-01dad638e8f31ab9a - Amazon Linux 2023 for CPU)

docker:
  image: ghcr.io/phymea/earbox:latest-gpu  # Or: latest-cpu

s3:
  input_bucket: my-input-bucket
  input_prefix: earbox-session/
  output_bucket: my-output-bucket

iam:
  automation_role_arn: arn:aws:iam::123456789:role/EarboxAutomationRole
```

Then run without CLI parameters:
```bash
python run_ec2_instance.py --config runner-config.yaml
```

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
- **RAM**: 8GB minimum, 16GB recommended
- **GPU** (optional): NVIDIA GPU with CUDA 11.8 support for faster processing
- **Storage**: ~5GB for Docker image + model weights

### EC2 Runner (Production)
- **Recommended instances:**
  - CPU: `t3.xlarge`, `c5.large`
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

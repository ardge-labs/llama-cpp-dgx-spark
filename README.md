# llama.cpp for NVIDIA DGX Spark (ARM64)

Pre-built Docker images of [llama.cpp](https://github.com/ggml-org/llama.cpp) optimized for **NVIDIA DGX Spark** ARM64 platform with Blackwell GPUs.

## Features

- ✅ **ARM64 native** - Built specifically for DGX Spark's aarch64 architecture
- ✅ **Blackwell optimized** - Compiled for CUDA compute capability `sm_121`
- ✅ **CUDA 13.0** - Latest CUDA toolkit with full Blackwell support
- ✅ **Auto-updated** - Automatically builds from llama.cpp upstream
- ✅ **Multiple variants** - Full, Server, and Light images available

## Quick Start

### Step 1: Download a GGUF Model

First, download a GGUF format model. Here's an example with Qwen3-0.6B:

```bash
# Create models directory
mkdir -p models
cd models

# Download Qwen3-0.6B GGUF model
wget https://huggingface.co/Qwen/Qwen3-0.6B-GGUF/resolve/main/Qwen3-0.6B-Q8_0.gguf

cd ..
```

**Popular GGUF Models:**
- [Qwen3-0.6B-GGUF](https://huggingface.co/Qwen/Qwen3-0.6B-GGUF) - Small, fast (600MB)
- [Qwen2.5-0.5B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF) - Tiny instruct model
- [Llama-3.2-1B-Instruct-GGUF](https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF) - Meta's 1B model
- [Llama-3.1-8B-Instruct-GGUF](https://huggingface.co/bartowski/Meta-Llama-3.1-8B-Instruct-GGUF) - Powerful 8B model
- [Gemma-3-12B-IT-QAT-Q4_0-GGUF](https://huggingface.co/google/gemma-3-12b-it-qat-q4_0-gguf) - Google's 12B quantized model

### Step 2: Run the Server

```bash
# Pull the image
docker pull ghcr.io/ardge-labs/llama-cpp-dgx-spark:server

# Run with your model
docker run --gpus all -p 8080:8080 \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:server \
  -m /models/Qwen3-0.6B-Q8_0.gguf
```

**Expected Output:**
```
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GB10, compute capability 12.1, VMM: yes
main: HTTP server is listening, hostname: 0.0.0.0, port: 8080
```

### Step 3: Test the API

Once the server is running, test it:

```bash
# Health check
curl http://localhost:8080/health

# Generate completion
curl http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Building a website can be done in 10 simple steps:",
    "n_predict": 128
  }'

# Chat completion (OpenAI-compatible)
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is the capital of France?"}
    ],
    "temperature": 0.7
  }'
```

## Other Image Variants

### Full (All Tools + Python)

Includes all llama.cpp tools and Python dependencies:

```bash
docker pull ghcr.io/ardge-labs/llama-cpp-dgx-spark:full

# Run CLI tool
docker run --gpus all -it \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:full \
  llama-cli -m /models/Qwen3-0.6B-Q8_0.gguf -p "Hello, world!"
```

### Light (CLI Only)

Minimal image with just the llama-cli binary:

```bash
docker pull ghcr.io/ardge-labs/llama-cpp-dgx-spark:light

docker run --gpus all \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:light \
  -m /models/Qwen3-0.6B-Q8_0.gguf -p "Hello!"
```

## Available Images

| Image | Tag | Description | Size |
|-------|-----|-------------|------|
| Full | `:full`, `:latest` | All tools + Python dependencies | ~4.4GB |
| Server | `:server` | llama-server only | ~2.5GB |
| Light | `:light` | llama-cli only | ~2.0GB |

## Platform Requirements

This image is built for and tested on:
- **Platform**: NVIDIA DGX Spark
- **Architecture**: ARM64 (aarch64)
- **GPU**: Blackwell GPUs (compute capability 12.1)
- **CUDA**: 13.0+
- **Driver**: Latest NVIDIA drivers

## Building Locally

If you want to build the image yourself on DGX Spark:

```bash
git clone https://github.com/YOUR_USERNAME/llama-cpp-dgx-spark.git
cd llama-cpp-dgx-spark

# Build full image
docker build -t llama-cpp-dgx-spark:full --target full .

# Build server image
docker build -t llama-cpp-dgx-spark:server --target server .

# Build light image
docker build -t llama-cpp-dgx-spark:light --target light .
```

### Build Arguments

You can customize the build:

```bash
# Build specific llama.cpp version
docker build --build-arg LLAMA_CPP_VERSION=b4368 -t llama-cpp-dgx-spark:custom .

# Use different CUDA architecture (default: 121)
docker build --build-arg CUDA_DOCKER_ARCH=120 -t llama-cpp-dgx-spark:custom .
```

## GitHub Actions Workflow

This repository uses GitHub Actions with self-hosted ARM64 runners to automatically build and publish images.

### Setting up Self-Hosted Runner

1. On your DGX Spark, navigate to your repository Settings > Actions > Runners
2. Click "New self-hosted runner"
3. Select "Linux" and "ARM64"
4. Follow the installation instructions
5. Add labels: `self-hosted`, `ARM64`, `Linux`

### Triggering Builds

**Automatic:**
- Push to `main`/`master` branch
- Create a new tag (e.g., `v1.0.0`)
- Open/update a pull request

**Manual:**
- Go to Actions > "Build and Publish ARM64 Docker Image" > "Run workflow"
- Optionally specify llama.cpp version (default: master)

## Configuration

### Environment Variables

The server image supports all standard llama-server environment variables:

```bash
docker run --gpus all -p 8080:8080 \
  -e LLAMA_ARG_HOST=0.0.0.0 \
  -e LLAMA_ARG_PORT=8080 \
  -e LLAMA_ARG_N_GPU_LAYERS=35 \
  ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server \
  -m /models/your-model.gguf
```

### Volumes

Mount your model directory:

```bash
-v /local/models:/models
```

## Examples

### Running with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  llama-server:
    image: ghcr.io/ardge-labs/llama-cpp-dgx-spark:server
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models
    command: ["-m", "/models/Qwen3-0.6B-Q8_0.gguf", "-c", "4096", "--port", "8080"]
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    restart: unless-stopped
```

Then run:
```bash
docker compose up -d
```

View logs:
```bash
docker compose logs -f
```

### Server Options

The server image accepts all standard llama-server command-line options. You can pass any option directly to the container:

```bash
# Run with custom context size and GPU layers
docker run --gpus all -p 8080:8080 \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:server \
  -m /models/Qwen3-0.6B-Q8_0.gguf \
  -c 8192 \
  --n-gpu-layers 35 \
  --threads 20 \
  --port 8080

# Run with verbose logging
docker run --gpus all -p 8080:8080 \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:server \
  -m /models/Qwen3-0.6B-Q8_0.gguf \
  --alias qwen3 \
  --verbose

# Enable metrics endpoint
docker run --gpus all -p 8080:8080 \
  -v ${PWD}/models:/models \
  ghcr.io/ardge-labs/llama-cpp-dgx-spark:server \
  -m /models/Qwen3-0.6B-Q8_0.gguf \
  --metrics

# Access metrics
curl http://localhost:8080/metrics
```

**Common Options:**
- `-c, --ctx-size` - Context size (default: 2048)
- `--n-gpu-layers` - Number of layers to offload to GPU
- `--threads` - Number of threads to use
- `--alias` - Model alias for API requests
- `--metrics` - Enable Prometheus-compatible metrics endpoint
- `--verbose` - Enable verbose logging
- `--port` - HTTP server port (default: 8080)
- `--host` - HTTP server hostname (default: 127.0.0.1)

For a complete list of options, see the [llama-server documentation](https://github.com/ggml-org/llama.cpp/blob/master/examples/server/README.md).

### API Usage Examples

Once the server is running, you can use the OpenAI-compatible API:

```bash
# Simple completion
curl http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "The meaning of life is",
    "n_predict": 50,
    "temperature": 0.7
  }'

# Chat completion (OpenAI-compatible endpoint)
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen",
    "messages": [
      {"role": "system", "content": "You are a helpful coding assistant."},
      {"role": "user", "content": "Write a Python function to calculate fibonacci numbers."}
    ],
    "temperature": 0.7,
    "max_tokens": 500
  }'

# Streaming chat completion
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen",
    "messages": [
      {"role": "user", "content": "Explain quantum computing in simple terms."}
    ],
    "stream": true
  }'
```

### Using with Python (OpenAI SDK)

```python
from openai import OpenAI

# Point to your local llama.cpp server
client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="not-needed"  # llama.cpp doesn't require auth by default
)

# Chat completion
response = client.chat.completions.create(
    model="qwen",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"}
    ],
    temperature=0.7,
    max_tokens=100
)

print(response.choices[0].message.content)
```

## Technical Details

### Build Configuration

- **Base Image**: `nvidia/cuda:13.0.0-devel-ubuntu24.04` (ARM64)
- **Runtime Image**: `nvidia/cuda:13.0.0-runtime-ubuntu24.04` (ARM64)
- **CUDA Architecture**: `sm_121` (Blackwell)
- **CMake Flags**:
  - `-DGGML_CUDA=ON` - Enable CUDA support
  - `-DCMAKE_CUDA_ARCHITECTURES=121` - Target Blackwell GPUs
  - `-DCMAKE_EXE_LINKER_FLAGS="-lcuda"` - Link CUDA driver library

### Key Differences from x86 Build

1. **Simplified CMake flags** - Removed `-DGGML_CPU_ALL_VARIANTS` to avoid ARM compiler issues
2. **CUDA 13.0** - Required for Blackwell GPU support on ARM
3. **Ubuntu 24.04** - Newer base for better ARM64 support
4. **Explicit sm_121** - Optimized for DGX Spark's specific GPU architecture

## Troubleshooting

### GPU not detected

Ensure NVIDIA Container Toolkit is installed:
```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### Image won't start

Check Docker logs:
```bash
docker logs <container-id>
```

Verify GPU access:
```bash
docker run --rm --gpus all nvidia/cuda:13.0.0-base-ubuntu24.04 nvidia-smi
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes on DGX Spark
4. Submit a pull request

## License

This project packages llama.cpp which is licensed under MIT License.
See [llama.cpp license](https://github.com/ggml-org/llama.cpp/blob/master/LICENSE) for details.

## Acknowledgments

- [llama.cpp](https://github.com/ggml-org/llama.cpp) - The underlying inference engine
- NVIDIA for DGX Spark platform
- The GGML community

## Links

- [llama.cpp Documentation](https://github.com/ggml-org/llama.cpp/blob/master/README.md)
- [NVIDIA DGX Platform](https://www.nvidia.com/en-us/data-center/dgx-platform/)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

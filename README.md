# llama.cpp for NVIDIA DGX Spark (ARM64)

Pre-built Docker images of [llama.cpp](https://github.com/ggml-org/llama.cpp) optimized for **NVIDIA DGX Spark** ARM64 platform with Blackwell GPUs.

## Features

- ✅ **ARM64 native** - Built specifically for DGX Spark's aarch64 architecture
- ✅ **Blackwell optimized** - Compiled for CUDA compute capability `sm_121`
- ✅ **CUDA 13.0** - Latest CUDA toolkit with full Blackwell support
- ✅ **Auto-updated** - Automatically builds from llama.cpp upstream
- ✅ **Multiple variants** - Full, Server, and Light images available

## Quick Start

### Server (Recommended)

Run the llama.cpp server with GPU acceleration:

```bash
docker pull ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server

docker run --gpus all -p 8080:8080 \
  -v /path/to/models:/models \
  ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server \
  -m /models/your-model.gguf
```

### Full (All Tools)

Includes all llama.cpp tools and Python dependencies:

```bash
docker pull ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:full

docker run --gpus all -it \
  -v /path/to/models:/models \
  ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:full \
  llama-cli -m /models/your-model.gguf -p "Hello, world!"
```

### Light (CLI Only)

Minimal image with just the llama-cli binary:

```bash
docker pull ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:light

docker run --gpus all \
  -v /path/to/models:/models \
  ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:light \
  -m /models/your-model.gguf -p "Hello!"
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

```yaml
version: '3.8'
services:
  llama-server:
    image: ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models
    command: ["-m", "/models/llama-2-7b.Q4_K_M.gguf", "-c", "4096"]
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

### API Usage

Once the server is running:

```bash
# Generate completion
curl http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Building a website can be done in 10 simple steps:",
    "n_predict": 128
  }'

# Chat completion
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
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

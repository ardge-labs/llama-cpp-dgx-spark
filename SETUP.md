# Setup Guide for llama.cpp DGX Spark Repository

This guide will help you set up the GitHub repository and configure the self-hosted ARM64 runner on your DGX Spark.

## Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `llama-cpp-dgx-spark` (or your preferred name)
3. Description: "Pre-built llama.cpp Docker images for NVIDIA DGX Spark ARM64"
4. Choose: **Public** (required for ghcr.io public access)
5. **Do NOT** initialize with README (we already have one)
6. Click "Create repository"

## Step 2: Push Code to GitHub

From this directory (`/home/timwu/llama-cpp-dgx-spark`):

```bash
# Add GitHub remote (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/llama-cpp-dgx-spark.git

# Rename branch to main (if needed)
git branch -M main

# Push to GitHub
git push -u origin main
```

## Step 3: Configure GitHub Packages

1. Go to your repository on GitHub
2. Click "Settings" > "Actions" > "General"
3. Scroll to "Workflow permissions"
4. Select "Read and write permissions"
5. Check "Allow GitHub Actions to create and approve pull requests"
6. Click "Save"

## Step 4: Set up Self-Hosted ARM64 Runner on DGX Spark

### Option A: Using GitHub's Runner (Recommended)

1. In your GitHub repository, go to "Settings" > "Actions" > "Runners"
2. Click "New self-hosted runner"
3. Select:
   - **OS**: Linux
   - **Architecture**: ARM64
4. Follow the installation commands (example below):

```bash
# Create a folder for the runner
mkdir -p ~/actions-runner && cd ~/actions-runner

# Download the latest runner package (ARM64)
curl -o actions-runner-linux-arm64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-arm64-2.311.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-arm64-2.311.0.tar.gz

# Configure the runner
./config.sh --url https://github.com/YOUR_USERNAME/llama-cpp-dgx-spark \
  --token YOUR_REGISTRATION_TOKEN

# IMPORTANT: Add these labels when prompted:
# Labels: self-hosted,ARM64,Linux

# Install as a service (optional but recommended)
sudo ./svc.sh install
sudo ./svc.sh start
```

### Option B: Docker-based Runner

Alternatively, run the runner in Docker:

```bash
docker run -d --restart always \
  --name github-runner \
  -e REPO_URL="https://github.com/YOUR_USERNAME/llama-cpp-dgx-spark" \
  -e RUNNER_TOKEN="YOUR_REGISTRATION_TOKEN" \
  -e RUNNER_NAME="dgx-spark-runner" \
  -e LABELS="self-hosted,ARM64,Linux" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /home/timwu/runner-data:/runner \
  myoung34/github-runner:latest
```

## Step 5: Enable GitHub Container Registry

1. Go to your repository on GitHub
2. Click "Packages" (or navigate to `https://github.com/YOUR_USERNAME?tab=packages`)
3. After the first build, your package will appear
4. Click on the package > "Package settings"
5. Scroll to "Danger Zone" > "Change visibility"
6. Select "Public" to allow anyone to pull images

## Step 6: Test the Workflow

### Manual Trigger:

1. Go to "Actions" tab in your repository
2. Click "Build and Publish ARM64 Docker Image"
3. Click "Run workflow"
4. Optionally specify llama.cpp version (leave empty for `master`)
5. Click "Run workflow"

### Automatic Trigger:

Make any change and push to main:

```bash
# Make a small change
echo "# Testing" >> README.md

# Commit and push
git add README.md
git commit -m "Test workflow trigger"
git push
```

## Step 7: Pull and Use the Image

Once the build completes:

```bash
# Pull the image
docker pull ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server

# Run it
docker run --gpus all -p 8080:8080 \
  ghcr.io/YOUR_USERNAME/llama-cpp-dgx-spark:server
```

## Troubleshooting

### Runner not starting builds

1. Check runner status in Settings > Actions > Runners
2. Ensure labels include: `self-hosted`, `ARM64`, `Linux`
3. Restart runner: `sudo ./svc.sh restart`

### Build failing

1. Check Actions tab for error logs
2. Ensure runner has:
   - Docker installed and running
   - NVIDIA Container Toolkit installed
   - Sufficient disk space (>20GB free)
   - Network access to ghcr.io

### Cannot pull published images

1. Ensure package visibility is set to "Public"
2. Check package exists at `https://github.com/YOUR_USERNAME/llama-cpp-dgx-spark/pkgs/container/llama-cpp-dgx-spark`
3. Try logging in: `docker login ghcr.io -u YOUR_USERNAME`

## Updating llama.cpp Version

### Via Workflow Dispatch:

1. Go to Actions > "Build and Publish ARM64 Docker Image" > "Run workflow"
2. Enter version (e.g., `b4368`, `v1.2.3`, or `master`)
3. Click "Run workflow"

### Via Git Tag:

```bash
# Create and push a version tag
git tag -a v1.0.0 -m "Release v1.0.0 with llama.cpp master"
git push origin v1.0.0
```

This will trigger a build and tag the image with `v1.0.0`.

## Security Notes

1. **GitHub Token**: The workflow uses `GITHUB_TOKEN` which is automatically provided by GitHub Actions
2. **Runner Security**: Self-hosted runners have access to your DGX Spark - only use in trusted repositories
3. **Public Images**: Images will be publicly accessible if package visibility is public

## Maintenance

### Cleaning up old images

GitHub Packages has retention policies. To manually delete old images:

1. Go to Package settings
2. Scroll to package versions
3. Delete unwanted versions

### Updating the runner

```bash
cd ~/actions-runner
sudo ./svc.sh stop
./config.sh remove --token YOUR_REMOVAL_TOKEN
# Download new version
# Re-configure and start
```

## Additional Resources

- [GitHub Actions Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [llama.cpp Documentation](https://github.com/ggml-org/llama.cpp)

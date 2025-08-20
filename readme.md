# Clay Docker Environments - Complete Setup Guide

This repository contains **two Docker images** to separate lightweight preprocessing from heavier model inference, with local Clay repo mounting for development.

## Prerequisites

- Docker installed and running
- Git
- For Windows users: WSL2 with Docker Desktop configured

## Directory Structure After Setup

```
base_plus_inference/
├── Dockerfile.clay-base          # Base image with common dependencies
├── Dockerfile.clay-infer         # Inference image built on clay-base
├── base-notebooks/               # Preprocessing notebooks (used with clay-base)
│   └── data_processing.ipynb
├── inference-notebooks/          # Model inference notebooks (used with clay-infer)
│   └── clay_inference_test.ipynb
├── clay-model/                   # Cloned Clay repository (will be created)
│   ├── .git/
│   ├── setup.py
│   ├── requirements.txt
│   └── ... (Clay model source code)
├── data/                         # Your datasets (create this folder)
│   └── (geoTIFFs or other data files)
└── base_plus_inference.md        # This file
```

## Setup Instructions

### Step 1: Clone This Repository
```bash
git clone https://github.com/YashGupta3003/BASE_PLUS_INFERENCE.git base_plus_inference
cd base_plus_inference
```

### Step 2: Clone the Clay Model Repository
```bash
# Clone Clay repo locally (this will create the clay-model/ directory)
git clone https://github.com/Clay-foundation/model.git clay-model
```

### Step 3: Build Docker Images

#### Build Base Image (Lightweight Environment)
```bash
docker build -f Dockerfile.clay-base -t clay-base .
```

#### Build Inference Image (Heavy Environment with Clay Dependencies)
```bash
docker build -f Dockerfile.clay-infer -t clay-infer .
```

**Note**: The inference image pre-installs Clay model dependencies during build, so runtime containers stay lightweight.

## Running the Containers

### Option 1: Base Container (Data Preprocessing)
```bash
docker run -it \
  -p 8888:8888 \
  -v $(pwd):/workspace \
  clay-base
```

**Access Jupyter**: After running the container, look in the terminal for a link like:
```
http://127.0.0.1:8888/tree?token=32e5e84ee48b1b95..........6
```
Copy this link and paste it in your browser to access the Jupyter server and run notebooks.

### Option 2: Inference Container (Model Inference)
```bash
docker run -it \
  -p 8889:8888 \
  -v $(pwd):/workspace \
  -v $(pwd)/clay-model:/workspace/clay-model \
  clay-infer
```

**Access Jupyter**: After running the container, look in the terminal for a link like:
```
http://127.0.0.1:8889/tree?token=32e5e84ee48b1b95...........6
```
Copy this link and paste it in your browser to access the Jupyter server and run notebooks.

## WSL2 Setup for Windows Users

### Prerequisites
1. **Enable WSL2**:
   ```powershell
   # Run in PowerShell as Administrator
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
   ```

2. **Install WSL2**:
   ```powershell
   wsl --install
   ```

3. **Install Docker Desktop for Windows** and enable WSL2 integration

4. **Restart your computer**

### WSL2 Commands
After setting up WSL2, open your WSL2 terminal and run the same commands as Linux:

```bash
# Navigate to your project (adjust path as needed)
cd /mnt/c/Users/YourUsername/Desktop/base_plus_inference

# Build images
docker build -f Dockerfile.clay-base -t clay-base .
docker build -f Dockerfile.clay-infer -t clay-infer .

# Clone Clay repo
git clone https://github.com/Clay-foundation/model.git clay-model

# Run containers
docker run -it -p 8888:8888 -v $(pwd):/workspace clay-base

# In another terminal
docker run -it -p 8888:8888 -v $(pwd):/workspace -v $(pwd)/clay-model:/workspace/clay-model clay-infer

**Note**: After running each container, look in the terminal for a Jupyter link with a token. Copy and paste this link in your browser to access the Jupyter server.
```

## How It Works

### Base Image (`clay-base`)
- Contains common dependencies: numpy, pandas, matplotlib, rasterio, geopandas, shapely, xarray, rioxarray, pystac_client, stackstac, python-box
- Includes Jupyter setup
- Lightweight for data preprocessing tasks

### Inference Image (`clay-infer`)
- Built on top of `clay-base`
- Pre-installs Clay model dependencies during build via `pip install git+https://github.com/Clay-foundation/model.git`
- At runtime, if `/workspace/clay-model` is mounted, installs it in editable mode with `--no-deps`
- Includes PyTorch (CPU version) for inference

### Runtime Behavior
- **Dependencies**: Pre-baked into the image during build
- **Local Code**: Mounted and installed in editable mode at runtime
- **Ephemeral Containers**: State is not persisted, keeping containers lightweight and production-ready

## Development Workflow

1. **Edit Clay Model Code**: Make changes in the `clay-model/` directory on your host machine
2. **Container Sees Changes**: The mounted directory makes changes immediately available inside the container
3. **Restart Container**: When you need a fresh environment, just stop and restart the container
4. **Code Persists**: Your changes stay on the host, containers remain ephemeral

## Troubleshooting

### Port Already in Use
```bash
# Check what's using the port
docker ps

# Stop and remove conflicting containers
docker stop <container-id>
docker rm <container-id>
```

### Permission Issues (WSL2)
```bash
# If you get permission errors, ensure Docker Desktop is running
# and WSL2 integration is enabled in Docker Desktop settings
```

### Container Exits Immediately
```bash
# Check container logs
docker logs <container-id>

# Ensure clay-model directory exists and is properly mounted
ls -la clay-model/
```

## Production Considerations

- **Ephemeral Containers**: Designed for frequent container restarts
- **Shared Storage**: Mount local directories for persistent data and code
- **Scalable**: Base image can be shared across multiple inference containers
- **Lightweight**: Runtime containers only install local code, not dependencies

## File Organization Rules

- **`base-notebooks/`**: Use for preprocessing notebooks (used with `clay-base`)
- **`inference-notebooks/`**: Use for model inference notebooks (used with `clay-infer`)
- **`clay-model/`**: Contains the Clay repository (cloned locally)
- **`data/`**: Place your datasets here (will be mounted into containers)

## Downloading Pretrained Model Weights

After completing all the above setup steps, you'll need to download the pretrained model weights to start inference.

### Step 1: Navigate to Your Project Directory
```bash
cd base_plus_inference
```

### Step 2: Download the Model Checkpoint
When you reach a cell in your inference notebook containing this code:
```python
device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
ckpt = "clay-v1.5.ckpt"
torch.set_default_device(device)
```

**Go to your terminal and run this command to download the pretrained weights:**
```bash
wget -q https://huggingface.co/made-with-clay/Clay/resolve/main/v1.5/clay-v1.5.ckpt
```

This will download the `clay-v1.5.ckpt` file to your current directory, which the model will then load for inference.

### Step 3: Verify Download
```bash
ls -la clay-v1.5.ckpt
```

You should see the checkpoint file in your directory, and your inference code should now be able to load the pretrained model weights successfully.

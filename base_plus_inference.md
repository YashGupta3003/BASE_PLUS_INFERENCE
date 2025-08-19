# Project README – Clay Docker Environments

This repository contains **two Docker images** to separate lightweight preprocessing from heavier model inference:

## Directory Structure Guidelines

```
.
├── Dockerfile.clay-base
├── Dockerfile.clay-infer
├── base-notebooks/
│   ├── data_processing.ipynb
│   └── (put base notebooks here)
├── inference_notebooks/
│   └── clay_inference_test.ipynb
├── data/
│   └── (geoTIFFs or datasets you mount)
└── README.md
```

##  Rules

* Use `base-notebooks/` for **preprocessing notebooks** (used with clay-base).
* Use `inference_notebooks/` for **model inference notebooks** (used with clay-infer).
* `Dockerfile.clay-base`: lightweight environment for data loading/EDA.
* `Dockerfile.clay-infer`: built on clay-base, installs clay model & torch.

##  Build & Run clay-base

Build image:

```bash
docker build -f Dockerfile.clay-base -t clay-base .
```

Run container:

```bash
docker run -it \
  -p 8888:8888 \
  -v $(pwd):/workspace \
  --name clay-basic-container \
  clay-base
```

➡️ Access Jupyter at: `http://localhost:8888`

✔ Recommended: Put your preprocessing notebooks inside `notebooks/` — they will be visible inside container at `/workspace/notebooks`

##  Build & Run clay-infer

Build image:

```bash
docker build -f Dockerfile.clay-infer -t clay-infer .
```

Run container:

```bash
docker run -it \
  -p 8889:8888 \
  -v $(pwd):/workspace \
  --name clay-infer-container \
  clay-infer
```

 Access Jupyter for inference at: `http://localhost:8889`

✔ Put inference notebooks in `inference_notebooks/` — they'll appear in container at `/workspace/inference_notebooks/`
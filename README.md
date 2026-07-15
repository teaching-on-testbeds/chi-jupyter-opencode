# Remote Jupyter and OpenCode

This container runs JupyterLab and OpenCode Web against the same workspace. Docker publishes both authenticated services on the VM's network interfaces by default.

## Launch on Chameleon

Run one of these notebooks in the Chameleon Jupyter environment:

- `launch-cpu.ipynb` reserves an `m1.medium` VM with an optional disposable boot volume.
- `launch-gpu.ipynb` reserves a `g1.h100.pci.1` VM with a CUDA image and a 100 GiB disposable boot volume by default.
- `cleanup.ipynb` deletes the selected VM, floating IP, disposable volume, and lease.

The launch notebooks expose two settings: `LEASE_DAYS` and `VOLUME_SIZE_GB`. They create inbound security-group rules for SSH, Jupyter, and OpenCode; assign a floating IP; generate reusable CHI@TACC S3 credentials; and start the container. The S3 credentials are written to `project/.env`, loaded into Jupyter and OpenCode, and excluded from Git by `project/.gitignore`.

Run each notebook from the directory containing the Docker and Compose files in this package.

## Start the services

Create `.env` from `.env.example`, then replace both token placeholders. You can generate each token with `openssl rand -hex 24`.

```bash
docker compose up --build -d
docker compose ps
```

On a VM with an NVIDIA GPU, NVIDIA Container Toolkit, and a working `nvidia-smi`, add the GPU overlay:

```bash
docker compose -f compose.yaml -f compose.gpu.yaml up --build -d
```

Follow startup logs with:

```bash
docker compose logs -f jupyter
```

On first startup, the container creates `/home/jovyan/work/<OPENCODE_PROJECT_DIR>` as a Git repository and opens it as OpenCode's default project. The default directory is `/home/jovyan/work/project`. JupyterLab keeps `/home/jovyan/work` as its root so it can access every project in the workspace.

## Open the browser interfaces

Open JupyterLab at `http://<FLOATING_IP>:8888/lab?token=<JUPYTER_TOKEN>`.

Open OpenCode at `http://<FLOATING_IP>:4096`. Sign in with the `OPENCODE_SERVER_USERNAME` and `OPENCODE_SERVER_PASSWORD` values from `.env`.

The cloud security group must allow inbound TCP ports 8888 and 4096. Direct HTTP access does not encrypt credentials or session traffic. Set `BIND_ADDRESS=127.0.0.1` and use SSH forwarding when you need encrypted transport.

## Check GPU access

Run these checks only when you started the service with `compose.gpu.yaml`.

```bash
docker compose exec jupyter nvidia-smi
docker compose exec jupyter python -c 'import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))'
```

Your notebooks and repositories persist under `JUPYTER_DATA_DIR`. OpenCode and GitHub CLI settings persist in named Docker volumes.

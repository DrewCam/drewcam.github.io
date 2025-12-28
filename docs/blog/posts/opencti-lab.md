---
title: OpenCTI Lab VM Build
draft: false
date: 2025-12-28
description: Build an Ubuntu Server VM and deploy OpenCTI using the official Docker Compose stack, with practical validation steps.
categories:
  - Cybersecurity
  - Threat Intelligence
  - Lab
tags:
  - OpenCTI
  - Threat Intelligence
  - Docker
  - Docker Compose
  - VMware
  - Ubuntu
  - STIX
---

# OpenCTI Lab VM Build

OpenCTI (Open Cyber Threat Intelligence) is an open source platform used to manage cyber threat intelligence knowledge and observables. It is designed to **structure, store, organise, and visualise technical and non-technical information about cyber threats** (see the [OpenCTI GitHub repository](https://github.com/OpenCTI-Platform/opencti)).

<!-- more -->

OpenCTI structures data using a **knowledge schema based on STIX standards**, supports linking intel to primary sources, and supports relationships between objects with features like first and last seen dates and confidence levels (see [OpenCTI Ecosystem](https://filigran.notion.site/OpenCTI-Ecosystem-868329e9fb734fca89692b2ed6087e76)).

OpenCTI ingestion is typically done via **connectors**. Import connectors retrieve data from external services, convert it to **STIX 2.1 bundles**, then import it into OpenCTI using workers (see the [OpenCTI connectors documentation](https://docs.opencti.io/latest/deployment/connectors/)).
STIX 2.1 is the CTI language/spec used to represent threat and observable information (see the [OASIS STIX 2.1 standard page](https://www.oasis-open.org/standard/6426/) and the [STIX 2.1 specification](https://docs.oasis-open.org/cti/stix/v2.1/csprd01/stix-v2.1-csprd01.html)).

## Goal

Deploy OpenCTI in a VMware Workstation Pro Ubuntu Server VM using the official Docker Compose stack, then validate the platform is reachable and healthy.

## Platform

* Hypervisor: VMware Workstation Pro
* Guest OS: Ubuntu Server 24.04 LTS (Noble), minimised
* Deployment: OpenCTI Docker deployment helpers repo at [OpenCTI-Platform/docker](https://github.com/OpenCTI-Platform/docker)

## 1. OS install steps (Ubuntu Server 24.04)

### 1.1 Create the VM (Workstation Pro)

1. VMware Workstation Pro → **File → New Virtual Machine** → Typical
2. Select the Ubuntu Server ISO (`ubuntu-24.04.x-live-server-amd64.iso`)
3. Set VM name and storage location
4. Set hardware:

   * vCPU: **4**
   * RAM: **8GB**
   * Disk: **120GB** (thin provision)
   * Network: default (DHCP for now)

### 1.2 Install Ubuntu

1. Boot from ISO
2. Choose language, keyboard
3. Select **Ubuntu Server (minimised)**

   * HWE kernel not required for this VM use case (minimised keeps the base footprint down)
4. Networking: accept DHCP defaults
5. Proxy: none
6. Mirror: accept default / closest region
7. Storage: guided, use entire disk (defaults)
8. Create local user:

   * user: `administrator`
   * set password
9. Enable **OpenSSH server**
10. Skip **Import SSH identity** (GitHub or Launchpad import not used)
11. Featured server snaps: none
12. Finish install, reboot, remove ISO when prompted

### 1.3 First login checks

```bash
whoami
ip -br a
hostnamectl
```

## 2. Set hostname

Hostname set to:

* `per-dcc-octi01`

```bash
sudo hostnamectl set-hostname per-dcc-octi01
sudo nano /etc/hosts
```

Ensure `/etc/hosts` has an entry like:

```text
127.0.1.1 per-dcc-octi01
```

Reload shell prompt:

```bash
exec bash
```

## 3. Base packages and time sync

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install open-vm-tools chrony curl git jq ca-certificates uuid-runtime
sudo systemctl enable --now chrony
```

## 4. Install Docker

Installed Docker Engine and the Compose plugin using the Docker convenience script:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker
docker version
```

## 5. Set `vm.max_map_count` for the search engine

OpenCTI’s Docker deployment guidance notes you must set `vm.max_map_count` before running the containers (see the [OpenCTI installation guide](https://docs.opencti.io/latest/deployment/installation/)).

```bash
sudo sysctl -w vm.max_map_count=1048575
echo 'vm.max_map_count=1048575' | sudo tee -a /etc/sysctl.conf
```

## 6. Deploy OpenCTI (Docker Compose)

OpenCTI recommends deploying using Docker and the default compose file from the OpenCTI Docker repo (see the [OpenCTI installation guide](https://docs.opencti.io/latest/deployment/installation/) and the [OpenCTI-Platform/docker repo](https://github.com/OpenCTI-Platform/docker)).

### 6.1 Clone repo and create `.env`

```bash
mkdir -p ~/opencti && cd ~/opencti
git clone https://github.com/OpenCTI-Platform/docker.git
cd docker
cp .env.sample .env
```

### 6.2 Identify VM IP (for UI access)

```bash
ip -4 -o addr show ens33 | awk '{print $4}' | cut -d/ -f1
```

### 6.3 Generate UUIDs for admin token and healthcheck key

```bash
uuidgen   # OPENCTI_ADMIN_TOKEN
uuidgen   # OPENCTI_HEALTHCHECK_ACCESS_KEY
```

### 6.4 Edit `.env`

```bash
nano .env
```

Set or update at minimum:

* `OPENCTI_HOST=<vm-ip>`
* `OPENCTI_PORT=8080`
* `OPENCTI_EXTERNAL_SCHEME=http`
* `OPENCTI_ADMIN_EMAIL=<your admin email>`
* `OPENCTI_ADMIN_PASSWORD=<your password>`
* `OPENCTI_ADMIN_TOKEN=<uuid>`
* `OPENCTI_HEALTHCHECK_ACCESS_KEY=<uuid>`
* Replace all remaining `changeme` secrets (MinIO, RabbitMQ, OpenSearch/Elastic admin password, etc.)

The install docs show the expected variables and the pattern of copying `.env.sample` to `.env` (see the [OpenCTI installation guide](https://docs.opencti.io/latest/deployment/installation/)).

### 6.5 Start the stack

```bash
docker compose pull
docker compose up -d
docker compose ps
```

## 7. Validation steps

### 7.1 Confirm 8080 is listening

```bash
ss -lntp | grep 8080 || true
```

### 7.2 Confirm OpenCTI returns HTTP 200 locally

```bash
curl -I http://127.0.0.1:8080
```

### 7.3 Web UI access

From your host browser:

* `http://<vm-ip>:8080`

Log in with:

* `OPENCTI_ADMIN_EMAIL`
* `OPENCTI_ADMIN_PASSWORD`

## 8. Quick log checks

```bash
docker compose logs --tail=200 opencti
docker compose logs --tail=200 connector-mitre
```

## 9. Operational preference when not in use

For a lab VM you will not touch again today:

* Bring containers down cleanly, then shut the VM down.

```bash
cd ~/opencti/docker
docker compose down
sudo shutdown -h now
```

Next time:

```bash
cd ~/opencti/docker
docker compose up -d
docker compose ps
```

## 10. OpenCTI up/down script (includes logs and status)

Save as: `~/opencti/docker/opencti-compose.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

# opencti-compose.sh
# Manage the OpenCTI Docker Compose stack (up/down/start/stop/status/restart/logs).
#
# Default directory is where this script lives.
# Override by setting OPENCTI_DIR=/path/to/opencti/docker

OPENCTI_DIR="${OPENCTI_DIR:-$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)}"

die() { echo "Error: $*" >&2; exit 1; }
need_cmd() { command -v "$1" >/dev/null 2>&1 || die "Missing command: $1"; }

need_cmd docker

cd "$OPENCTI_DIR" || die "Cannot cd to: $OPENCTI_DIR"
[[ -f "docker-compose.yml" || -f "compose.yml" ]] || die "No docker compose file found in $OPENCTI_DIR"

compose() { docker compose "$@"; }

usage() {
  cat <<'EOF'
Usage: opencti-compose.sh <command> [options]

Commands:
  up [--pull]      Start stack (optionally pull images first)
  down             Stop and remove containers (keeps volumes)
  down-volumes     Stop and remove containers + volumes (destructive)
  start            Start existing containers
  stop             Stop containers
  restart          Restart containers
  status           Show container status (compose ps)
  logs [service]   Follow logs (all services or one service)

Examples:
  ./opencti-compose.sh up --pull
  ./opencti-compose.sh status
  ./opencti-compose.sh logs opencti
EOF
}

cmd="${1:-}"; shift || true

case "$cmd" in
  up)
    if [[ "${1:-}" == "--pull" ]]; then
      compose pull
      shift || true
    fi
    compose up -d
    compose ps
    ;;
  down) compose down ;;
  down-volumes)
    echo "WARNING: This deletes volumes (data)."
    compose down -v
    ;;
  start)
    compose start
    compose ps
    ;;
  stop) compose stop ;;
  restart)
    compose restart
    compose ps
    ;;
  status|ps) compose ps ;;
  logs)
    svc="${1:-}"
    if [[ -n "$svc" ]]; then
      compose logs -f "$svc"
    else
      compose logs -f
    fi
    ;;
  ""|-h|--help|help) usage ;;
  *) die "Unknown command: $cmd (run with --help)" ;;
esac
```

Make executable:

```bash
chmod +x ~/opencti/docker/opencti-compose.sh
```

### Usage examples

From the OpenCTI compose directory:

```bash
cd ~/opencti/docker
```

Start the stack (first time or after a shutdown):

```bash
./opencti-compose.sh up --pull
```

Start the stack (no pull):

```bash
./opencti-compose.sh up
```

Show status:

```bash
./opencti-compose.sh status
# or
./opencti-compose.sh ps
```

Follow all logs:

```bash
./opencti-compose.sh logs
```

Follow logs for a single service (examples):

```bash
./opencti-compose.sh logs opencti
./opencti-compose.sh logs connector-mitre
./opencti-compose.sh logs worker
./opencti-compose.sh logs elasticsearch
```

Stop containers (keeps everything, fastest “pause”):

```bash
./opencti-compose.sh stop
```

Start existing containers again:

```bash
./opencti-compose.sh start
```

Restart containers:

```bash
./opencti-compose.sh restart
```

Stop and remove containers (keeps data volumes, clean shutdown):

```bash
./opencti-compose.sh down
```

**Destructive** reset (removes containers _and_ volumes, deletes data):

```bash
./opencti-compose.sh down-volumes
```

## Relevant links

* OpenCTI project overview: [https://github.com/OpenCTI-Platform/opencti](https://github.com/OpenCTI-Platform/opencti)
* OpenCTI documentation: [https://docs.opencti.io/latest/](https://docs.opencti.io/latest/)
* OpenCTI installation guide: [https://docs.opencti.io/latest/deployment/installation/](https://docs.opencti.io/latest/deployment/installation/)
* OpenCTI connectors guide: [https://docs.opencti.io/latest/deployment/connectors/](https://docs.opencti.io/latest/deployment/connectors/)
* OpenCTI Docker deployment helpers: [https://github.com/OpenCTI-Platform/docker](https://github.com/OpenCTI-Platform/docker)
* OASIS STIX 2.1 standard page: [https://www.oasis-open.org/standard/6426/](https://www.oasis-open.org/standard/6426/)
* STIX 2.1 specification: [https://docs.oasis-open.org/cti/stix/v2.1/csprd01/stix-v2.1-csprd01.html](https://docs.oasis-open.org/cti/stix/v2.1/csprd01/stix-v2.1-csprd01.html)

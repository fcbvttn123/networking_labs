# Table of contents

- [Table of contents](#table-of-contents)
- [EVE-NG Network Emulation Lab on Google Cloud](#eve-ng-network-emulation-lab-on-google-cloud)
  - [Project Overview](#project-overview)
    - [Objectives](#objectives)
  - [Architecture and Technology Stack](#architecture-and-technology-stack)
  - [1. Prepare the Ubuntu Boot Image](#1-prepare-the-ubuntu-boot-image)
    - [1.1 Enable the Compute Engine API](#11-enable-the-compute-engine-api)
    - [1.2 Create a Custom Ubuntu Image](#12-create-a-custom-ubuntu-image)
    - [1.3 Verify the Image](#13-verify-the-image)
  - [2. Provision the EVE-NG Virtual Machine](#2-provision-the-eve-ng-virtual-machine)
    - [2.1 Machine Configuration](#21-machine-configuration)
    - [2.2 Operating System and Storage](#22-operating-system-and-storage)
    - [2.3 Deployment Considerations](#23-deployment-considerations)
  - [3. Install EVE-NG Professional](#3-install-eve-ng-professional)
    - [3.1 Download and Run the Installer](#31-download-and-run-the-installer)
    - [3.2 Reboot and Configure the IP Address](#32-reboot-and-configure-the-ip-address)
  - [4. Install EVE-NG Docker Integration](#4-install-eve-ng-docker-integration)

# EVE-NG Network Emulation Lab on Google Cloud

## Project Overview

This project documents the deployment of an EVE-NG Professional network emulation lab on Google Cloud Platform (GCP).

The lab runs on a custom Ubuntu 24.04 LTS image with nested virtualization enabled, providing an environment for building, configuring, and testing virtual network topologies.

### Objectives

- Deploy EVE-NG Professional on Google Compute Engine.
- Configure a custom Ubuntu image with nested virtualization support.
- Provision cloud infrastructure for network emulation workloads.
- Install EVE-NG and Docker integration.
- Establish a foundation for hands-on networking and troubleshooting labs.

## Architecture and Technology Stack

| Component | Configuration |
|---|---|
| Cloud Provider | Google Cloud Platform |
| Compute Service | Google Compute Engine |
| Operating System | Ubuntu 24.04 LTS |
| Virtualization | Nested virtualization |
| CPU Platform | AMD Milan |
| Machine Type | `n2d-standard-8` |
| vCPUs | 8 |
| Memory | 32 GB |
| Boot Disk | 100 GB balanced persistent disk |
| Provisioning Model | Spot VM |
| Network Emulation | EVE-NG Professional |
| Container Integration | EVE-NG Docker package |

## 1. Prepare the Ubuntu Boot Image

### 1.1 Enable the Compute Engine API

Open Google Cloud Shell and select the Google Cloud project where the lab will be deployed.

Enable the Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

### 1.2 Create a Custom Ubuntu Image

Create an Ubuntu 24.04 LTS image with the license required for nested virtualization:

```bash
gcloud compute images create nested-ubuntu-noble \
    --source-image-family=ubuntu-2404-lts-amd64 \
    --source-image-project=ubuntu-os-cloud \
    --licenses=https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx
```

This creates a custom image named `nested-ubuntu-noble`.

### 1.3 Verify the Image

List custom images in the Google Cloud project:

```bash
gcloud compute images list \
    --project=eve-ng-home-lab \
    --no-standard-images
```

Verify that `nested-ubuntu-noble` appears in the output.

> **Note:** Replace `eve-ng-home-lab` with your actual Google Cloud project ID if necessary.

## 2. Provision the EVE-NG Virtual Machine

Create a new virtual machine in Google Compute Engine using the custom Ubuntu image.

### 2.1 Machine Configuration

Configure the VM with the following specifications:

- **Machine family:** General Purpose
- **Machine series:** N2D
- **CPU platform:** AMD Milan
- **Machine type:** `n2d-standard-8`
- **vCPUs:** 8
- **Memory:** 32 GB
- **Provisioning model:** Spot

This configuration provides the compute resources needed to run network emulation workloads.

### 2.2 Operating System and Storage

Configure the boot disk:

- **Operating system image:** `nested-ubuntu-noble`
- **Disk capacity:** 100 GB
- **Disk type:** Balanced persistent disk

### 2.3 Deployment Considerations

- Verify that the selected machine type and region support the required nested virtualization configuration.
- Spot VMs can be preempted, so the lab should be treated as an interruptible environment.
- Back up important lab configurations and virtual appliance files.
- Restrict network access to trusted sources and expose only the management services required.

## 3. Install EVE-NG Professional

Connect to the VM using SSH after the instance has started.

### 3.1 Download and Run the Installer

Switch to the root account and download the official EVE-NG installation script:

```bash
sudo -i

wget https://www.eve-ng.net/noble/install-eve-pro.sh

chmod +x install-eve-pro.sh

./install-eve-pro.sh
```

After the installer completes, update and upgrade the system packages:

```bash
apt update
apt upgrade
```

### 3.2 Reboot and Configure the IP Address

Reboot the VM when prompted by the installation process.

Allow the VM to restart, then reconnect to the SSH session.

When the EVE-NG IP configuration wizard appears:

1. Press `Ctrl+C` to exit the current wizard screen, following the installation procedure.
2. Switch to the root account:

   ```bash
   sudo -i
   ```

3. Follow the IP configuration wizard.
4. Select DHCP for IP address assignment.
5. Complete the configuration and reboot the VM.

After the reboot, reconnect to the VM using SSH.

Verify the VM's current IP address in Google Cloud Console.

## 4. Install EVE-NG Docker Integration

After reconnecting to the VM, switch to the root account and update the package lists:

```bash
sudo -i

apt update
```

Install the EVE-NG Docker package:

```bash
apt install eve-ng-dockers
```

Allow the package installation to complete.

Mark each item as complete only after verifying it in the deployed environment.
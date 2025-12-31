# Ansible Molecule Disk Testing

An Ansible role for testing disk mounting operations using Molecule and Docker. This project demonstrates automated testing of infrastructure code for disk management tasks.

## Overview

This role tests disk mounting operations by:
- Creating a simulated disk image using a loopback device
- Formatting the disk with ext4 filesystem
- Mounting the disk to a specified mount point
- Verifying the mount is successful and writable

## Use Case

Developed to learn Ansible testing practices with Molecule. Useful for:
- Testing disk provisioning automation
- Validating configuration management for storage operations
- CI/CD integration for infrastructure code

## Prerequisites

**Local Development (macOS):**
- Docker Desktop
- Python 3.8+
- pip

**CI/CD:**
- GitHub Actions (automatically configured)

## Installation
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/ansible-molecule-disk-testing.git
cd ansible-molecule-disk-testing

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install molecule molecule-docker ansible-core
```

## Usage

### Run Full Test Suite
```bash
molecule test
```

### Development Workflow
```bash
molecule create      # Create test container
molecule converge    # Apply the role
molecule verify      # Run verification tests
molecule login       # Debug inside container
molecule destroy     # Clean up
```

## Project Structure
```
.
├── .github/
│   └── workflows/
│       └── molecule.yml       # CI/CD pipeline
├── molecule/
│   └── default/
│       ├── molecule.yml       # Molecule configuration
│       ├── converge.yml       # Test playbook
│       ├── verify.yml         # Verification tests
│       └── Dockerfile.j2      # Test container image
├── tasks/
│   └── main.yml               # Ansible role tasks
├── defaults/
│   └── main.yml               # Default variables
└── README.md
```

## CI/CD

The project includes a GitHub Actions workflow that automatically runs Molecule tests on every push and pull request. View test results in the **Actions** tab of the repository.

## Configuration

Default mount point: `/mnt/data`

Override by setting the `mount_point` variable:
```yaml
mount_point: /custom/mount/path
```

## Technical Details

- **Container**: Ubuntu 22.04
- **Filesystem**: ext4
- **Disk Size**: 100MB (configurable in Dockerfile.j2)
- **Testing Framework**: Molecule with Docker driver

## Platform Notes

- **macOS**: Uses file-backed mounting (`mount -o loop`)
- **Linux**: Uses loopback devices (`/dev/loop0`)
- **CI/CD**: Runs on Ubuntu with full loopback support

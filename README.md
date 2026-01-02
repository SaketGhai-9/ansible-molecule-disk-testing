# Ansible Molecule Disk Testing

An Ansible role for testing disk operations using Molecule and Docker. This project demonstrates automated testing of infrastructure code for disk management tasks including partitioning, formatting, and mounting.

## Overview

This role tests comprehensive disk operations by:
- **Partitioning**: Creating GPT partition tables and primary partitions on disk images
- **Formatting**: Formatting partitions with ext4 filesystem
- **Mounting**: Mounting formatted partitions to specified mount points
- **Verification**: Testing that all operations are successful, idempotent, and writable

## Use Case

Developed to learn Ansible testing practices with Molecule. Useful for:
- Testing disk provisioning automation
- Validating configuration management for storage operations
- CI/CD integration for infrastructure code
- Learning Molecule testing patterns for complex disk operations

## Platform Requirements

⚠️ **Important Platform Notes:**

### GitHub Actions (Linux) - Recommended
The CI/CD pipeline is configured for **Ubuntu Linux** where loopback devices (`/dev/loop0`, `/dev/loop0p1`) work natively. This is the primary testing environment and all operations work seamlessly here.

### Local Development

**macOS:**
- Docker Desktop on Mac runs containers in a Linux VM
- Loopback device partition scanning doesn't work the same way
- The role includes fallback logic using `kpartx` for partition mapping
- Device paths will be `/dev/mapper/loop0p1` instead of `/dev/loop0p1`
- All operations work but with different device mapping

**Windows:**
- Docker Desktop on Windows has similar limitations to Mac
- Recommend using WSL2 for better compatibility
- Or test exclusively through GitHub Actions

**Linux (Native Docker):**
- Works identically to GitHub Actions
- Full loopback device support
- Recommended for local testing if available

## Prerequisites

**For Local Development:**
- Docker Desktop (Mac/Windows) or Docker Engine (Linux)
- Python 3.8+
- pip

**For CI/CD:**
- GitHub Actions (automatically configured)

## Installation
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/ansible-molecule-disk-testing.git
cd ansible-molecule-disk-testing

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install molecule molecule-docker ansible-core
```

## Usage

### Run Full Test Suite
```bash
molecule test
```

This will test all three operations together: partition → format → mount

### Development Workflow
```bash
molecule create      # Create test container
molecule converge    # Apply the role
molecule verify      # Run verification tests
molecule login       # Debug inside container
molecule destroy     # Clean up
```

### Check Idempotence
```bash
molecule test
```
The test suite automatically runs idempotence checks to ensure operations don't create unnecessary changes when run multiple times.

## Project Structure
```
.
├── .github/
│   └── workflows/
│       └── molecule.yml       # CI/CD pipeline (Linux-based)
├── molecule/
│   └── default/
│       ├── molecule.yml       # Molecule configuration
│       ├── converge.yml       # Test playbook
│       ├── verify.yml         # Verification tests
│       └── Dockerfile.j2      # Test container image
├── tasks/
│   ├── main.yml              # Main task orchestration
│   ├── partition.yml         # Partitioning operations
│   ├── format.yml            # Formatting operations
│   └── mount.yml             # Mounting operations
├── defaults/
│   └── main.yml              # Default variables
└── README.md
```

## Disk Operations Details

### 1. Partitioning
- Creates GPT partition table on disk image
- Creates a single primary partition using 100% of disk space
- Sets up loop devices with partition scanning
- Uses `kpartx` for partition mapping on platforms where native partition scanning doesn't work

### 2. Formatting
- Formats the partition with ext4 filesystem
- Checks for existing filesystems before formatting
- Validates filesystem creation with `blkid`
- Idempotent - skips formatting if filesystem already exists

### 3. Mounting
- Creates mount point directory (`/mnt/data` by default)
- Mounts the formatted partition
- Verifies mount is successful and writable
- Idempotent - skips mounting if already mounted

## Configuration

Configure operations via `defaults/main.yml`:
```yaml
disk_operation: "all"          # Options: partition, format, mount, all
disk_device: "/disk.img"       # Disk image path
filesystem_type: "ext4"        # Filesystem type (ext4, xfs, ext3)
mount_point: "/mnt/data"       # Mount point path
force_format: false            # Force reformat even if filesystem exists
```

## CI/CD

The project includes a GitHub Actions workflow that automatically runs Molecule tests on every push and pull request.

**Pipeline Configuration:**
- Runs on: `ubuntu-latest` (native Linux with full loopback support)
- Tests: Partition, Format, Mount (all operations)
- Validates: Syntax, Idempotence, Verification

**View Pipeline Results:**
1. Go to your repository on GitHub
2. Click the **Actions** tab
3. View workflow runs and test results

## Technical Details

**Container Base:** Ubuntu 22.04  
**Filesystem:** ext4  
**Disk Size:** 500MB (configurable in Dockerfile.j2)  
**Testing Framework:** Molecule with Docker driver  
**Partition Table:** GPT  
**Loop Devices:** Managed via `losetup` and `kpartx`

## Platform-Specific Behavior

| Platform | Loop Device | Partition Device | Notes |
|----------|-------------|------------------|-------|
| **GitHub Actions (Linux)** | `/dev/loop0` | `/dev/loop0p1` or `/dev/mapper/loop0p1` | Full support, recommended |
| **Mac (Docker Desktop)** | `/dev/loop0` | `/dev/mapper/loop0p1` | Uses kpartx fallback |
| **Linux (Native Docker)** | `/dev/loop0` | `/dev/loop0p1` | Full support |
| **Windows (Docker Desktop)** | `/dev/loop0` | `/dev/mapper/loop0p1` | Uses kpartx fallback |

## Troubleshooting

**"Partition device not found"**
- On Mac/Windows: This is expected, kpartx fallback will be used
- The role automatically detects and handles this

**"Idempotence test failed"**
- Check that loop devices and mounts are properly detected before recreation
- Review the partition.yml and mount.yml idempotence checks

**"Mount failed: wrong fs type"**
- Ensure the partition was formatted successfully
- Check that the correct partition device is being used

## Testing Locally on Mac

If testing locally on macOS, expect:
1. Partition creation will succeed
2. Loop device will be created
3. kpartx will map the partition to `/dev/mapper/loop0p1`
4. All tests should pass, just with different device paths

The code handles this automatically through conditional logic.


## Future Enhancements

Possible extensions for this project:
- Support for multiple partitions
- LVM (Logical Volume Management) testing
- Different filesystem types (xfs, btrfs)
- Disk resizing operations
- RAID configuration testing
- Encryption (LUKS) testing

## License

MIT

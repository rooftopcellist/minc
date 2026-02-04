# MINC: MicroShift in Container

MINC enables the deployment of [MicroShift](https://github.com/openshift/microshift), a lightweight OpenShift/Kubernetes distribution, within [Podman](https://podman.io/) as container.
This approach facilitates a streamlined and efficient environment for developing, testing, and running cloud-native applications.

## Features

- Containerized deployment of MicroShift using Podman.
- Simplified setup for lightweight Kubernetes clusters.
- Ideal for development, testing, and edge computing scenarios.

## Getting Started

### Prerequisites

- [Podman](https://podman.io/getting-started/installation) installed on your system.
- Basic understanding of Kubernetes and containerization concepts.
- `kubectl` or `oc` cli tool installed

#### Windows

In windows on wsl environment make sure you have cgroupsv2 enabled which is not the case by default
- https://github.com/spurin/wsl-cgroupsv2
- Kernel command line parameter kernelCommandLine=systemd.unified_cgroup_hierarchy=1 results in creation 
of cgroup V1 and V2 hierarchy. It is prohibited now, microsoft/WSL#6662 (more details around cgroups-v1/v2)

### Installation

Get the latest release from GitHub release page as per your platform.

#### Linux

```bash
curl -L -o minc  https://github.com/minc-org/minc/releases/latest/download/minc_linux_amd64
chmod +x minc
```

By default, MINC uses `sudo` to run Podman in rootful mode. See the [Rootless Mode](#rootless-mode-linux) section below for running without sudo.

#### Mac
```bash
curl -L -o minc  https://github.com/minc-org/minc/releases/latest/download/minc_darwin_arm64
chmod +x minc
```

#### Windows
```bash
curl.exe -L -o minc.exe  https://github.com/minc-org/minc/releases/latest/download/minc.exe
```

## Usage

### Create the cluster 
```bash
minc create
```

### Status of the cluster

This command provide output in `json` format
```bash
minc status
{
  "container": "running",
  "apiserver": "running"
}
```
In case of error output would be look like below
```bash
minc status
{
  "container": "stopped",
  "apiserver": "stopped",
  "error": "no microshift containers found, use 'create' command to create it"
}
```

### Delete the cluster
```bash
minc delete
```

### Regenerate kubeconfig file for cluster
```bash
minc generate-kubeconfig
```

### Get help and options
```bash
minc help
```

### Config options
```bash
minc config -h
```

### Available Config Settings
| Parameter            | Description                                                                                                                                           |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `microshift-config`  | Custom MicroShift config file to change MicroShift defaults. [More info](https://github.com/openshift/microshift/blob/main/docs/user/howto_config.md) |
| `microshift-version` | MicroShift version, check available tags at `quay.io/minc-org/minc`                                                                                   |
| `log-level`          | Log level (default: `info`)                                                                                                                           |
| `provider`           | Container runtime provider, e.g., `docker`, `podman` (default: `podman`)                                                                              |
| `https-port`         | Different port to use for exposing https service (default:`9443`)                                                                                     |
| `http-port`          | Different port to use for exposing https service (default:`9080`)                                                                                     |
| `rootless`           | Run in rootless mode without sudo on Linux (default: `false`). See [Rootless Mode](#rootless-mode-linux).                                            |


Once the container is running, you can interact with the MicroShift cluster using `kubectl` or `oc` tools.

## Rootless Mode (Linux)

MINC supports running with rootless Podman on Linux, which eliminates the need for `sudo`. This is useful on systems like Fedora where rootless Podman is the default.

### Enabling Rootless Mode

```bash
# Via command-line flag
minc --rootless create

# Or set it permanently via config
minc config set rootless true
minc create
```

### System Requirements for Rootless Mode

Running MicroShift in rootless Podman requires sufficient subordinate UID/GID mappings. The MicroShift container image contains files owned by high-numbered UIDs (100000+), which must be mapped to your user namespace.

#### Fedora / RHEL / CentOS

1. **Check your current mappings:**
   ```bash
   cat /etc/subuid
   cat /etc/subgid
   ```

   You need at least 65536 subordinate IDs starting from a base of 100000 or higher.

2. **Add sufficient mappings (if needed):**
   ```bash
   sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 $USER
   ```

3. **Apply the changes:**
   ```bash
   podman system migrate
   ```

4. **Verify the mappings:**
   ```bash
   podman unshare cat /proc/self/uid_map
   ```

#### Troubleshooting

If you see an error like:
```
insufficient UIDs or GIDs available in user namespace (requested 100058:100058)
```

This means your subordinate UID/GID range is too small. Follow the steps above to add more mappings.

#### Additional Resources

- [Podman Rootless Setup](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [Understanding subuid/subgid](https://www.redhat.com/sysadmin/rootless-podman-user-namespace-modes)

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your changes.

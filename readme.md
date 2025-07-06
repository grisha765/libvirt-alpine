# libvirt-alpine
A minimal Alpine Linux container running `libvirtd`, `virtlogd`, and `virtlockd`, suitable for managing virtual machines with `virsh` over TCP.

## Requirements

- `podman` or `docker`
- `/dev/kvm` and `/dev/net/tun` available on the host
- Host must support nested virtualization (if running in a VM)

## Usage

- Build the Image:
    ```bash
    sudo podman build -t libvirt:latest -f dockerfile .
    ```

- Run the Container:
    ```bash
    sudo podman run --privileged -d \
      --name libvirt \
      --restart=always \
      --network host \
      -v libvirt-run:/var/run/libvirt \
      -v libvirt-lib:/var/lib/libvirt \
      -v libvirt-qemu:/etc/libvirt/qemu \
      -v libvirt-network:/etc/libvirt/network \
      -v libvirt-storage:/etc/libvirt/storage \
      -v /path/to/host-dir-images:/data/images \
      --device /dev/kvm \
      --device /dev/net/tun \
      localhost/libvirt:latest
    ```

### Explanation of Flags

- `--privileged`: required for libvirt and KVM device access
- `--network host`: exposes libvirt on the host's network
- `--device /dev/kvm` and `/dev/net/tun`: required for VM creation and networking
- `--restart=always`: ensures the container restarts on reboot
- Volume mounts ensure libvirt state is persistent

### Connect with virsh

- From a remote or local machine:
    ```bash
    virsh -c 'qemu+tcp://<host-ip>/system' list --all
    ```
    - Replace `<host-ip>` with the actual IP address of the host running the container.

## Notes

- Authentication is **disabled** in this configuration (for simplicity). Do **not** expose this to the public internet without proper firewalling or VPN.
- VM images should be stored in `/data/images` inside the container (mapped from host).
- You can modify the `libvirtd.conf`, `qemu.conf`, or `supervisord.conf` files before building to suit your setup.

## Features

- Alpine-based minimal image
- libvirt over TCP (no TLS, no authentication)
- QEMU/KVM virtualization support
- Pre-configured with SPICE and VNC remote display support


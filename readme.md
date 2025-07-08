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

- Run the rootful Container:
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

- Run the rootless Container:
    ```bash
    podman run -d \
        --name libvirt \
        --restart=always \
        -v $XDG_RUNTIME_DIR/libvirt:/var/run/libvirt:z \
        -v libvirt-lib:/var/lib/libvirt \
        -v libvirt-qemu:/etc/libvirt/qemu \
        -v libvirt-network:/etc/libvirt/network \
        -v libvirt-storage:/etc/libvirt/storage \
        -v $PWD/images:/data/images:z \
        --device /dev/kvm \
        localhost/libvirt:latest
    ```

### Connect with virsh

- From a remote or tcp rootful machine:
    ```bash
    virsh -c 'qemu+tcp://<host-ip>/system' list --all
    ```
    - Replace `<host-ip>` with the actual IP address of the host running the container.

- From a remote or socket rootless machine:
    ```bash
    virsh -c 'qemu+unix:///session?socket=$XDG_RUNTIME_DIR/libvirt/libvirt-sock' list --all
    ```

## Notes

- Authentication is **disabled** in this configuration (for simplicity). Do **not** expose this to the public internet without proper firewalling or VPN.
- VM images should be stored in `/data/images` inside the container (mapped from host).
- You can modify the `libvirtd.conf`, `qemu.conf`, or `supervisord.conf` files before building to suit your setup.
- [virt-manager](https://flathub.org/apps/org.virt_manager.virt-manager) does not support environment variables, so instead of `$XDG_RUNTIME_DIR`, you must use the full path to the socket.

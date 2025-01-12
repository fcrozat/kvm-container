# What's inside

This container provides a kvm toolstack inside a container.

* `Dockerfile` with the definition of the kvm container
currently based on the openSUSE Tumbleweed BCI image.
Installs qemu, libvirt, virt-install and some additional tools
* `kvm-container.conf` contains environment variables used during deployment
* `kvm-container-functions` functions to check configuration
* `kvm-container-host-service` is a script to deploy the kvm container and the libvirt daemons through their systemd services
* `virsh` is the wrapper on the host to use virsh command
* `virt-install-demo.sh` is a wrapper to quickly install a test VM
* `virt-install` is the wrapper on the host to virt-install
* `default_network.xml` contains a deafult network configuration for the container and its workloads

# Current deployments

For each of the commands below, replace `<registry_path>` with one of the following:

* `registry.opensuse.org/suse/alp/workloads/tumbleweed_containerfiles/suse/alp/workloads/kvm-modular-libvirt` - The latest stable release deployed to the SUSE ALP distribution (Default)
* `registry.opensuse.org/virtualization/containerfile/suse/alp/workloads/kvm-modular-libvirt` - The latest development branch of the kvm-container
* `<custom-registry-path>` - Path to your OBS home project registry, local registy, or external registry

# How to get a working kvm-container

## Prepare your host system
```
# podman container runlabel install <registry_path>:latest
```
> Note: Ensure the value of `IMAGE` in `/etc/kvm-container.conf` is the same as the registry path used for the installation

> Note: systemd units are currently installed to the host administrator's systemd directory (`/etc/systemd/system/`) since the root filesystem is mounted read-only on an ALP system

> Note: All commands to be run as root

## Deploy the container

```
# podman container runlabel service-enable <registry_path>:latest
```
This will first start the container, then will use the installed systemd units to deploy the libvirt daemons inside the container
 
## Install the test VM

```
# virt-install-demo.sh
```

# Local VM management
`/usr/local/bin/virsh` is passed through to the container so it can be used locally just as if libvirt were running on the host
```
# virsh list --all
```

# Remote VM management
Ensure ssh access is configured between the client machine (running virsh or virt-manger locally) and the container host (where the kvm-container was deployed), then:
```
virsh -c "qemu+ssh://root@YOURHOST/system"
```
Optionally with an ssh key:
```
virsh -c "qemu+ssh://root@YOURHOST/system?keyfile=<local_path_to_private_key>"
```

# Remote VM access 

## From the container's host machine (assuming the test VM and default VM network)
```
# vncviewer 192.168.10.1:5950
```

## From an external system
* Ensure ssh access is configured between the client machine and the container host
* Ensure the VM was created with a vnc server (i.e. `--graphics vnc,listen=0.0.0.0,port=5950` for the test VM)
* Create a port-forwarded ssh tunnel: `ssh -NL 5900:127.0.0.1:5900 <ip_of_container_host>`
* Establish vnc connection from client: `vncviewer 127.0.0.1::5900`

# Stop the container
```
# systemctl stop kvm-container-meta.service
```
> Note using `podman stop` is not advised. Since the container lifecycle is managed by systemd, this will only cause the container to re-exec but none of the container's libvirt services will be restarted

# Update the container
First stop the container with `systemctl stop kvm-container-meta.service`, then: 
```
# sudo podman runlabel update <registry_path>
```
This will update to the latest container image including updated virtualization components

# Disable containerized libvirt services
```
# sudo podman runlabel service-disable <registry_path>
```
This will stop all libvirt service in the container, stop the container, and disable the service from running on the next reboot. Nothing is uninstalled from the host. [Redeploy](README.md#deploy-the-container) as desired. 

# Uninstall the kvm-container
```
# sudo podman runlabel uninstall <registry_path>
```

# Warning

This code is only provided for experimentation.

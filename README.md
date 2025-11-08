# Learning-Kubernetes
Initial Setup



## **Issue**

When Running the following command on the control-plane (master) node:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

i get error:

```bash
I1107 19:50:20.055497   20408 version.go:256] remote version is much newer: v1.34.1; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.14
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

This error means IP forwarding is disabled on your host system, which is required for applications like Kubernetes to route traffic between different network interfaces.

## **How to Fix this**

To fix it, we must enable IP forwarding by changing the /proc/sys/net/ipv4/ip_forward file to 1. This can be done temporarily with `sudo sysctl -w net.ipv4.ip_forward=1` or permanently by adding `net.ipv4.ip_forward=1` to a file in `/etc/sysctl.d/` and then running `sudo sysctl --system`. 




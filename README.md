# homelab

Kubernetes home lab running on a MacBook Pro (M2, 16GB) via UTM virtual machines.

## Cluster

Three Debian 13 (trixie) arm64 VMs, bridged networking, k3s:

| Node        | Role          | IP             | Specs        |
|-------------|---------------|----------------|--------------|
| k3s-server  | control-plane | 192.168.86.172 | 2GB / 2 cores |
| k3s-agent-1 | worker        | (fill in)      | 2GB / 2 cores |
| k3s-agent-2 | worker        | (fill in)      | 2GB / 2 cores |

Ingress is handled by k3s's bundled Traefik on port 80 of every node.
Hostnames like `uptime.home` are resolved via `/etc/hosts` entries on
client machines, pointed at the server IP.

## Apps

| App         | Namespace  | URL                | Manifest                         |
|-------------|------------|--------------------|----------------------------------|
| Uptime Kuma | monitoring | http://uptime.home | `apps/uptime-kuma/uptime-kuma.yaml` |

## Workflow

Every change goes through this repo:

1. Edit the manifest
2. Commit and push
3. `sudo kubectl apply -f <manifest>`

Never edit resources directly on the cluster.

## Rebuilding from scratch

1. Create three Debian arm64 VMs in UTM (2GB RAM, 2 cores, 20GB disk,
   bridged network). Randomize MAC addresses on clones. Set unique
   hostnames and regenerate `/etc/machine-id` on clones.
2. Install k3s on the server:
   `curl -sfL https://get.k3s.io | sh -`
3. Join agents using the token from
   `/var/lib/rancher/k3s/server/node-token`:
   `curl -sfL https://get.k3s.io | K3S_URL=https://<SERVER_IP>:6443 K3S_TOKEN=<TOKEN> sh -`
4. Apply everything in `apps/`:
   `sudo kubectl apply -f apps/ -R`

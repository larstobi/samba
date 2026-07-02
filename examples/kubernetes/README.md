# Kubernetes examples

These examples show two common ways to run the Samba container in Kubernetes.
Copy one of the manifests, change the passwords, paths, storage settings, and
image tag, then apply it.

## Host network with hostPath

Use `host-network-hostpath.yaml` for a home-lab or bare-metal cluster where the
Samba server should appear in normal network discovery views.

This example:

- uses `hostNetwork: true`
- enables Avahi, wsdd, and nmbd discovery
- stores shared files on a node path with `hostPath`
- exposes SMB directly on the selected node's ports

Only one pod can bind these ports on a node. If the cluster has several nodes,
pin the workload to the node that owns the storage path.

```console
kubectl apply -f examples/kubernetes/host-network-hostpath.yaml
```

## LoadBalancer with PVC

Use `load-balancer-pvc.yaml` when the cluster provides persistent volumes and a
LoadBalancer implementation such as MetalLB or a cloud load balancer.

This example:

- stores shared files on a `PersistentVolumeClaim`
- exposes ports 139 and 445 through a `Service`
- disables multicast discovery because it normally does not pass through a
  Kubernetes Service

Connect directly to the Service's external IP, for example `smb://IP/Data`.

```console
kubectl apply -f examples/kubernetes/load-balancer-pvc.yaml
```

## Notes

The manifests keep Samba users in Kubernetes `Secret` objects. The sample
passwords are placeholders and must be changed before use.

User lists in `SHARE`, `SHARE2`, and later variants are comma-separated:

```text
SHARE: "Data;/share/data;yes;no;no;alice,bob;none;none;Shared data"
```

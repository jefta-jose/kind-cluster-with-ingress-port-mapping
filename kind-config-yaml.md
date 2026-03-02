
---

## Top Level

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
```

* `kind: Cluster` → This defines a **Kind cluster** configuration file.
* `apiVersion: kind.x-k8s.io/v1alpha4` → Uses the v1alpha4 schema for Kind; newer versions of Kind may have slightly different fields.

---

## Nodes

```yaml
nodes:
  - role: control-plane
    ...
```

* We define **one node**, and it’s a **control-plane node** (the master node in Kubernetes terms).
* Everything (scheduler, API server, etc.) runs on this node.
* This node will also host workloads if you don’t add worker nodes.

---

### kubeadmConfigPatches

```yaml
kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
```

* `kubeadmConfigPatches` allows you to **patch the cluster’s kubeadm configuration**.
* `kind: InitConfiguration` → This is part of kubeadm’s bootstrap configuration.
* `nodeRegistration.kubeletExtraArgs.node-labels` → This **adds a label to the node**:

```text
ingress-ready=true
```

* **Why?** NGINX Ingress can be configured with a `nodeSelector` to only schedule pods on nodes that are ready for ingress.
* This is the key step that lets the Ingress controller know **“I can be deployed here”**.

---

### extraPortMappings

```yaml
extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```

* `extraPortMappings` exposes ports **from the Kind node container to your host machine**.
* Each Kind cluster node is actually a **Docker container**, so by default ports inside the container are isolated.
* `containerPort` → Port inside the Kind node container
* `hostPort` → Port on your actual machine
* `protocol: TCP` → Standard TCP protocol

**What this achieves:**

* Port `80` (HTTP) and `443` (HTTPS) on your host machine now **map directly to the Kind node**.
* This allows you to type `http://go-api.local` in your browser, and the traffic reaches the Ingress controller inside Kind.

Without these port mappings, the cluster would be unreachable on standard HTTP/HTTPS ports.

---

## Summary

1. **Creates a single-node Kind cluster** (`control-plane`).
2. **Labels the node** as `ingress-ready=true` so Ingress controllers know where to deploy.
3. **Maps ports 80 and 443** from the node container to your host, allowing local HTTP/HTTPS access.

Essentially, this setup ensures that you can run an **NGINX Ingress controller in Kind** and have it handle traffic on standard web ports on your machine.

---
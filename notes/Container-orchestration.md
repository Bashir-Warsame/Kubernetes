## 1. Container Runtimes & CRI (Container Runtime Interface)

### What is a Container Runtime?

The **Container Runtime** is the software that actually runs containers. Kubernetes supports multiple runtimes through a standardized interface.

| Runtime | Description | Status |
|---------|-------------|--------|
| **containerd** | Industry-standard runtime, donated by Docker to CNCF | **Default in most clusters** |
| **CRI-O** | Lightweight runtime specifically for Kubernetes | Popular in OpenShift |
| **Docker** | Via `cri-dockerd` shim (Docker Engine itself isn't a CRI runtime) | Legacy support |

**Key Point**: Kubernetes uses the **CRI (Container Runtime Interface)**—a gRPC API specification—to communicate with container runtimes. This allows Kubernetes to be runtime-agnostic.

### CRI Architecture

```
Kubelet (on Worker Node)
    ↓ (speaks CRI - gRPC API)
Container Runtime (containerd/CRI-O)
    ↓
Container (running application)
```

**Why this matters**: CRI enables you to swap container runtimes without changing Kubernetes itself. This is part of Kubernetes' extensibility philosophy.

### OCI (Open Container Initiative)

The **OCI** defines standards for:
- **Runtime Spec**: How to run containers (`runc` is the reference implementation)
- **Image Spec**: Container image format (layers, manifests)

**containerd** uses `runc` under the hood to actually create and run containers.

---

## 2. CNI (Container Network Interface)

### What is CNI?

**CNI** is a specification and libraries for configuring network interfaces in Linux containers. It's how Kubernetes pods get their IP addresses and network connectivity.

### How CNI Works

1. **Kubelet** calls CNI plugin when pod is created
2. **CNI Plugin** assigns IP address from configured range
3. **CNI Plugin** sets up network interface (veth pair, bridge, etc.)
4. **Network Policy** enforcement (if supported by plugin)

### Popular CNI Plugins

| Plugin | Characteristics | Best For |
|--------|----------------|----------|
| **Flannel** | Simple, overlay network (VXLAN), no NetworkPolicy | Small clusters, simplicity |
| **Calico** | BGP routing, NetworkPolicy support, scalable | Production, security-focused |
| **Cilium** | eBPF-based, high performance, observability | Advanced networking, service mesh |
| **Weave Net** | Mesh networking, encrypted overlays | Multi-cloud, complex topologies |
| **AWS VPC CNI / Azure CNI** | Native cloud integration | Cloud-specific deployments |

**KCNA Focus**: Know that CNI plugins provide pod networking and some support NetworkPolicies. You don't need to configure them, just understand their role.

---

## 3. CSI (Container Storage Interface)

### What is CSI?

**CSI** standardizes how storage systems expose block and file storage to container orchestrators. Like CRI and CNI, it makes Kubernetes storage-agnostic.

### CSI Architecture

```
Application Pod
    ↓
PersistentVolumeClaim (PVC)
    ↓
PersistentVolume (PV) provisioned by CSI Driver
    ↓
CSI Driver (AWS EBS, Azure Disk, NFS, etc.)
    ↓
Actual Storage Backend
```

### Key CSI Concepts

| Term | Description |
|------|-------------|
| **CSI Driver** | Software implementing CSI spec for specific storage (e.g., AWS EBS CSI driver) |
| **Volume Snapshot** | Point-in-time copy of a volume |
| **Volume Cloning** | Create new volume from existing one |
| **Volume Expansion** | Resize volumes dynamically |

**Benefits of CSI**:
- Storage vendors can write drivers without modifying Kubernetes core
- Supports advanced features (snapshots, cloning, resizing)
- Consistent API across different storage backends

---

## 4. Kubernetes Security Fundamentals

### RBAC (Role-Based Access Control)

RBAC controls who can do what in a Kubernetes cluster.

#### Core RBAC Objects

| Object | Scope | Purpose |
|--------|-------|---------|
| **Role** | Namespace | Defines permissions within one namespace |
| **ClusterRole** | Cluster-wide | Defines permissions across all namespaces or cluster-scoped resources |
| **RoleBinding** | Namespace | Assigns Role/ClusterRole to users/groups/serviceaccounts in one namespace |
| **ClusterRoleBinding** | Cluster-wide | Assigns ClusterRole to users/groups/serviceaccounts cluster-wide |

#### RBAC Example

```yaml
# Role: Can view pods in "default" namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]          # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# RoleBinding: Assigns "pod-reader" to user "jane"
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Key Points**:
- **Principle of Least Privilege**: Give minimum permissions needed
- **ServiceAccounts**: Used by pods to access Kubernetes API (not just humans)
- **ClusterRoleBinding + Role**: Gives cluster-wide permissions to a Role

### Pod Security Standards (PSS)

Replaced Pod Security Policies (PSP) in Kubernetes v1.25+.

| Profile | Description |
|---------|-------------|
| **Privileged** | Unrestricted, dangerous capabilities allowed |
| **Baseline** | Minimally restrictive, prevents known privilege escalations |
| **Restricted** | Heavily restricted, follows security best practices |

Enforced via **Pod Security Admission** (built-in admission controller).

### Network Policies

We covered these in Fundamentals, but from an orchestration perspective:
- **Default allow-all**: No NetworkPolicy = all traffic allowed
- **Default deny**: Create empty ingress/egress rules to block by default
- **Namespace isolation**: Prevent cross-namespace traffic

---

## 5. Service Mesh

### What is a Service Mesh?

A **Service Mesh** is a dedicated infrastructure layer for handling service-to-service communication. It adds capabilities without changing application code.

### Service Mesh Features

| Feature | Description |
|---------|-------------|
| **mTLS** | Automatic mutual TLS encryption between services |
| **Traffic Management** | Canary deployments, A/B testing, circuit breaking |
| **Observability** | Distributed tracing, metrics, service graphs |
| **Security** | Identity-based access control, encryption |
| **Resilience** | Retries, timeouts, circuit breakers |

### Popular Service Mesh Implementations

| Mesh | Characteristics | CNCF Status |
|------|----------------|-------------|
| **Istio** | Feature-rich, Envoy-based, complex | Graduated |
| **Linkerd** | Lightweight, Rust-based, simpler | Graduated |
| **Consul Connect** | HashiCorp ecosystem integration | - |
| **Cilium Service Mesh** | eBPF-based, high performance | Incubating |

**Sidecar Pattern**: Traditional service meshes deploy a proxy (Envoy) alongside each application pod.

```
[App Container] ←→ [Sidecar Proxy (Envoy)] ←→ Network
```

**Sidecar-less (eBPF)**: Emerging pattern where networking is handled at kernel level (Cilium) for better performance.

---

## 6. DNS & Service Discovery

### CoreDNS

**CoreDNS** is the default cluster DNS server in Kubernetes.

**Functions**:
- Assigns DNS records to Services and Pods
- Enables service discovery via DNS names instead of IPs

### DNS Naming Conventions

| Resource | DNS Pattern | Example |
|----------|-------------|---------|
| Service (same namespace) | `service-name` | `my-service` |
| Service (different namespace) | `service-name.namespace.svc.cluster.local` | `my-service.prod.svc.cluster.local` |
| Pod | `pod-ip.namespace.pod.cluster.local` | `10-0-0-1.default.pod.cluster.local` |

**Headless Services**: When `clusterIP: None`, DNS returns pod IPs directly (useful for StatefulSets).

---

## 7. Autoscaling

### Types of Autoscaling

| Scaler | What it scales | Based on | Use Case |
|--------|---------------|----------|----------|
| **HPA** (Horizontal Pod Autoscaler) | Number of pod replicas | CPU, memory, or custom metrics | Variable load on application |
| **VPA** (Vertical Pod Autoscaler) | Pod resource requests/limits | Actual resource usage | Right-sizing pod resources |
| **Cluster Autoscaler** | Number of nodes in cluster | Pending pods that can't be scheduled | Infrastructure scaling |

### HPA Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50    # Scale when CPU > 50%
```

**Important**: HPA requires **Metrics Server** to be installed in the cluster.

---

## 8. Scheduling & Resource Management

### How the Scheduler Works

1. **Filtering**: Remove nodes that don't meet requirements (resource constraints, selectors, affinity rules)
2. **Scoring**: Rank remaining nodes by suitability (resource availability, data locality, etc.)
3. **Binding**: Assign pod to highest-scoring node

### Key Scheduling Concepts

| Concept | Description |
|---------|-------------|
| **Resource Requests** | Guaranteed minimum resources for a pod |
| **Resource Limits** | Maximum resources a pod can consume |
| **Node Selector** | Simple constraint: pod only runs on nodes with specific labels |
| **Affinity/Anti-affinity** | Complex constraints (preferred vs. required) |
| **Taints & Tolerations** | Nodes repel pods unless they "tolerate" the taint |
| **Pod Topology Spread** | Distribute pods across failure domains |

### Resource Example

```yaml
resources:
  requests:
    memory: "256Mi"      # Guaranteed minimum
    cpu: "250m"          # 250 millicores = 0.25 CPU
  limits:
    memory: "512Mi"      # Max memory (OOMKill if exceeded)
    cpu: "500m"          # Max CPU (throttled if exceeded)
```

**Quality of Service (QoS) Classes**:
- **Guaranteed**: Requests = Limits (all resources)
- **Burstable**: Requests < Limits (or only some resources set)
- **BestEffort**: No requests/limits set

---

## 9. Key Interfaces Summary

| Interface | Purpose | Examples |
|-----------|---------|----------|
| **CRI** | Container Runtime Interface | containerd, CRI-O |
| **CNI** | Container Network Interface | Calico, Flannel, Cilium |
| **CSI** | Container Storage Interface | AWS EBS, Azure Disk, NFS |
| **CPI** | Cloud Provider Interface | AWS, Azure, GCP integrations |
| **OCI** | Open Container Initiative | Image format, runtime specs |

---

## Quick Review Checklist

Before moving on, ensure you understand:

- [ ] What is CRI and why does Kubernetes use it?
- [ ] Difference between containerd, CRI-O, and Docker's role today
- [ ] What does CNI do and name 2-3 popular plugins?
- [ ] How does CSI enable storage vendor independence?
- [ ] RBAC: Difference between Role and ClusterRole?
- [ ] What is a Service Mesh and what problems does it solve?
- [ ] Difference between HPA, VPA, and Cluster Autoscaler?
- [ ] How does CoreDNS enable service discovery?
- [ ] What are the 3 Pod Security Standards?
- [ ] What is the sidecar pattern in service meshes?

---

## Practice Questions

**Q1**: Which interface allows Kubernetes to work with different container runtimes like containerd and CRI-O without code changes?

<details>
<summary>Answer</summary>
CRI (Container Runtime Interface). It standardizes communication between kubelet and container runtimes.
</details>

**Q2**: You need to automatically scale your application based on CPU usage from 3 to 20 replicas. Which component should you use?

<details>
<summary>Answer</summary>
Horizontal Pod Autoscaler (HPA). It scales pod replicas based on observed metrics like CPU utilization.
</details>

**Q3**: What is the primary benefit of using a Service Mesh like Istio or Linkerd?

<details>
<summary>Answer</summary>
It provides mTLS encryption, traffic management, and observability for service-to-service communication without modifying application code.
</details>

**Q4**: Which RBAC object would you use to grant permissions across all namespaces in a cluster?

<details>
<summary>Answer</summary>
ClusterRole. Roles are namespace-scoped, while ClusterRoles are cluster-scoped.
</details>

**Q5**: A storage vendor wants to integrate their enterprise SAN with Kubernetes. Which standard should they implement?

<details>
<summary>Answer</summary>
CSI (Container Storage Interface). It allows storage vendors to write drivers that work with Kubernetes without modifying core code.
</details>

---

**Next**: Ready for **Cloud Native Architecture (16%)**? This covers microservices, serverless, CNCF project maturity, and cloud native roles. Or want more practice on Container Orchestration first?
**Cloud Native Architecture (16% of KCNA)**. 
This section covers architectural patterns, the CNCF ecosystem, and organizational aspects of cloud native adoption.

---

## 1. Microservices Architecture

### Monolithic vs. Microservices

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| **Structure** | Single unified codebase | Multiple independent services |
| **Deployment** | One deployable unit | Each service deploys independently |
| **Scaling** | Scale entire application | Scale individual services |
| **Technology** | Single stack | Polyglot (different languages/frameworks) |
| **Complexity** | Simpler initially | Higher operational complexity |
| **Failure** | Single point of failure | Isolated failures |

### Microservices Characteristics

| Principle | Description |
|-----------|-------------|
| **Single Responsibility** | Each service does one thing well |
| **Independently Deployable** | Deploy without affecting other services |
| **Decentralized Data** | Each service owns its data (database per service) |
| **API Communication** | Services talk via APIs (HTTP/gRPC/messaging) |
| **Resilience** | Failures don't cascade (circuit breakers, bulkheads) |

### Service Communication Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Synchronous (REST/gRPC)** | Real-time request/response | User authentication |
| **Asynchronous (Message Queue)** | Decoupled, eventual consistency | Order processing |
| **Event-Driven** | React to state changes | Inventory updates |

**Challenges of Microservices**:
- **Distributed complexity**: Network latency, partial failures
- **Data consistency**: Distributed transactions
- **Operational overhead**: Monitoring, logging across services
- **Testing**: Integration testing complexity

---

## 2. Serverless & FaaS (Function as a Service)

### What is Serverless?

**Serverless** doesn't mean "no servers"—it means you don't manage servers. The cloud provider handles infrastructure provisioning, scaling, and maintenance.

### FaaS Characteristics

| Feature | Description |
|---------|-------------|
| **Event-driven** | Functions triggered by events (HTTP, queue, schedule) |
| **Stateless** | Each invocation is independent |
| **Ephemeral** | Short-lived (typically seconds to minutes) |
| **Auto-scaling** | Scales to zero and up automatically |
| **Pay-per-use** | Billed by execution time and memory |

### Kubernetes Serverless Options

| Project | Description | CNCF Status |
|---------|-------------|-------------|
| **Knative** | Kubernetes-based serverless platform | Incubating |
| **OpenFaaS** | Simple functions on Kubernetes | - |
| **Kubeless** | Native serverless framework | - |

### Knative Components

| Component | Purpose |
|-----------|---------|
| **Serving** | Scale-to-zero, traffic splitting, revision management |
| **Eventing** | Event-driven architecture, event sources and sinks |

**Use Cases for Serverless**:
- API backends
- Data processing pipelines
- Scheduled tasks (cron jobs)
- IoT data ingestion
- Chatbots and webhooks

---

## 3. CNCF (Cloud Native Computing Foundation) Landscape

### What is CNCF?

The **CNCF** is part of the Linux Foundation that hosts open-source cloud native projects. It provides governance, marketing, and community support.

### Project Maturity Levels

| Level | Criteria | Examples |
|-------|----------|----------|
| **Sandbox** | Early-stage, experimental | New projects, innovative ideas |
| **Incubating** | Used in production, growing community | Knative, etcd (was), Cilium |
| **Graduated** | Stable, widely adopted, production-ready | Kubernetes, Prometheus, Envoy, Istio |

**Graduation Requirements**:
- Documented adopters in production
- Governance and committer processes
- Security audits
- Defined release process
- Healthy community metrics

### Key CNCF Projects by Category

| Category | Projects |
|----------|----------|
| **Orchestration** | Kubernetes, Crossplane |
| **Observability** | Prometheus, Grafana, Jaeger, Fluentd |
| **Service Mesh** | Istio, Linkerd, Envoy |
| **Storage** | etcd, Rook, Vitess |
| **Streaming/Messaging** | NATS, Apache Kafka (not CNCF but popular), gRPC |
| **Security** | Falco, Notary, SPIFFE/SPIRE |
| **CI/CD** | Argo, Flux, Tekton |

### Cloud Native Landscape Map

The [CNCF Landscape](https://landscape.cncf.io/) visualizes the ecosystem:

- **Provisioning**: Automation, security, registry
- **Runtime**: Cloud native storage, container runtime, networking
- **Orchestration & Management**: Scheduling, coordination, service discovery
- **App Definition & Development**: Databases, streaming, messaging, application definition
- **Observability & Analysis**: Monitoring, logging, tracing, chaos engineering

---

## 4. Cloud Native Roles & Responsibilities

### Key Roles

| Role | Focus | Key Skills |
|------|-------|------------|
| **DevOps Engineer** | CI/CD pipelines, infrastructure automation | Jenkins, GitLab CI, Terraform, Ansible |
| **SRE (Site Reliability Engineer)** | System reliability, SLAs, incident response | Monitoring, chaos engineering, automation |
| **Platform Engineer** | Internal developer platforms, self-service infrastructure | Kubernetes, APIs, developer experience |
| **Cloud Architect** | High-level design, cloud strategy | Multi-cloud, cost optimization, security |
| **Security Engineer** | Cloud native security, compliance | Policy as code, vulnerability scanning, zero trust |

### DevOps vs. SRE vs. Platform Engineering

| Aspect | DevOps | SRE | Platform Engineering |
|--------|--------|-----|---------------------|
| **Primary Goal** | Faster delivery through automation | Reliability and availability | Developer productivity |
| **Approach** | Cultural shift, breaking silos | Engineering solutions to operations | Building internal platforms |
| **Metrics** | Deployment frequency, lead time | SLIs, SLOs, error budgets | Developer satisfaction, time-to-production |
| **Tooling** | CI/CD, IaC | Monitoring, incident management | Self-service portals, abstractions |

---

## 5. Kubernetes Governance & Community

### Kubernetes Release Cycle

| Aspect | Details |
|--------|---------|
| **Frequency** | 3 releases per year (roughly every 4 months) |
| **Support** | 14 months of support for each release |
| **Versioning** | Semantic versioning (v1.28, v1.29, etc.) |
| **Branches** | `master` (development), release branches |

### Special Interest Groups (SIGs)

Kubernetes is organized into **SIGs** focused on specific areas:

| SIG | Responsibility |
|-----|---------------|
| **SIG-Architecture** | Design and architecture decisions |
| **SIG-Node** | Node components, kubelet, container runtime |
| **SIG-Network** | Networking, DNS, ingress, service mesh |
| **SIG-Storage** | Storage plugins, volumes, CSI |
| **SIG-Scheduling** | Pod scheduling, resource management |
| **SIG-Scalability** | Performance at scale |
| **SIG-Security** | Security features, audits, policies |

### Kubernetes Enhancement Proposals (KEPs)

- **KEPs** are design documents for significant changes to Kubernetes
- Required for new features, architectural changes, or process modifications
- Ensures community review and consensus before implementation

---

## 6. Cloud Native Best Practices

### 12-Factor App Methodology

| Factor | Principle |
|--------|-----------|
| 1. Codebase | One codebase tracked in revision control, many deploys |
| 2. Dependencies | Explicitly declare and isolate dependencies |
| 3. Config | Store config in environment variables |
| 4. Backing Services | Treat backing services as attached resources |
| 5. Build, Release, Run | Strictly separate build and run stages |
| 6. Processes | Execute the app as one or more stateless processes |
| 7. Port Binding | Export services via port binding |
| 8. Concurrency | Scale out via the process model |
| 9. Disposability | Fast startup and graceful shutdown |
| 10. Dev/Prod Parity | Keep development, staging, production as similar as possible |
| 11. Logs | Treat logs as event streams |
| 12. Admin Processes | Run admin/management tasks as one-off processes |

### Cloud Native Design Principles

| Principle | Implementation |
|-----------|---------------|
| **Immutable Infrastructure** | Replace rather than modify; version-controlled artifacts |
| **Declarative APIs** | Describe desired state, system reconciles |
| **Observability** | Metrics, logs, traces built-in from start |
| **Security by Default** | Zero trust, least privilege, defense in depth |
| **Resilience** | Design for failure, circuit breakers, retries |
| **Automation** | GitOps, CI/CD, infrastructure as code |

---

## 7. Multi-Cloud & Hybrid Cloud

### Deployment Patterns

| Pattern | Description |
|---------|-------------|
| **Single Cloud** | All workloads on one provider (AWS, Azure, GCP) |
| **Multi-Cloud** | Workloads across multiple providers |
| **Hybrid Cloud** | On-premises + public cloud |
| **Distributed Cloud** | Cloud services distributed to different physical locations |

### Kubernetes in Multi-Cloud

| Approach | Tooling |
|----------|---------|
| **Single Cluster Spanning Clouds** | Complex, high latency (rare) |
| **Federated Clusters** | KubeFed (deprecated), newer tools emerging |
| **Cluster per Cloud** | Independent clusters with shared tooling (ArgoCD, etc.) |
| **Cluster API** | Declarative cluster lifecycle management across providers |

### Challenges

- **Data gravity**: Moving data between clouds is expensive/slow
- **Vendor lock-in**: Managed services vary between providers
- **Complexity**: Different networking, IAM, and operational models
- **Cost management**: Tracking spend across providers

---

## Quick Review Checklist

- [ ] What are the key differences between monolithic and microservices?
- [ ] What are the 3 CNCF project maturity levels and their criteria?
- [ ] Name 3 Graduated CNCF projects
- [ ] What is FaaS and how does it relate to serverless?
- [ ] What is Knative and what are its two main components?
- [ ] Difference between DevOps, SRE, and Platform Engineer?
- [ ] What is a KEP and why is it important?
- [ ] How often does Kubernetes release new versions?
- [ ] What is the 12-Factor App methodology?
- [ ] What does "immutable infrastructure" mean?

---

## Practice Questions

**Q1**: A company wants to adopt cloud native practices but needs to keep some workloads on-premises for compliance. What deployment pattern describes this?

<details>
<summary>Answer</summary>
Hybrid Cloud. It combines on-premises infrastructure with public cloud resources.
</details>

**Q2**: Which CNCF maturity level indicates a project is stable, widely adopted, and production-ready?

<details>
<summary>Answer</summary>
Graduated. Projects must meet strict criteria including production adopters, security audits, and established governance.
</details>

**Q3**: What is the primary responsibility of a Site Reliability Engineer (SRE)?

<details>
<summary>Answer</summary>
System reliability and availability, measured through SLIs (Service Level Indicators) and SLOs (Service Level Objectives), with error budgets for balancing reliability vs. feature velocity.
</details>

**Q4**: Which Kubernetes component enables serverless workloads with scale-to-zero capabilities?

<details>
<summary>Answer</summary>
Knative (specifically Knative Serving). It extends Kubernetes to provide serverless characteristics like automatic scaling to zero and revision management.
</details>

**Q5**: What is the purpose of a Kubernetes Enhancement Proposal (KEP)?

<details>
<summary>Answer</summary>
KEPs are design documents that propose significant changes to Kubernetes, ensuring community review, consensus, and documentation before implementation.
</details>

---

**Next**: Ready for **Cloud Native Observability (8%)**? This covers monitoring, logging, tracing, and the three pillars of observability. Or want to review any architecture concepts first?
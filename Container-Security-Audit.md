# Security Audit Questions for Containerized Application Deployment in Azure Cloud

## Introduction

This audit checklist is designed to assess the security posture of a containerized application deployment project on Azure Cloud. The questions are aligned with best practices from Microsoft Cloud Security Benchmark (MCSB), Microsoft Defender for Cloud Secure Score, and Kubernetes security guidelines.

Each section covers a specific domain of container security and orchestration. Auditors, security architects, DevSecOps professionals, and compliance teams can use this checklist to evaluate Azure-based container environments effectively.

---

## 1. Container Image Security

### ✅ Image Source and Validation

- Are only **approved base images** used in container builds?
- Are container images pulled from **trusted registries** (e.g., ACR, DockerHub with verified publishers)?
- Is **Azure Container Registry (ACR)** used with private endpoints and network rules?
- Are **image tags pinned** to specific versions (e.g., `nginx:1.25.2`) to avoid drifting?

### ✅ Image Scanning and Vulnerability Management

- Are all images **scanned for vulnerabilities** before being pushed to ACR?
- Is **Microsoft Defender for Containers** enabled for continuous vulnerability monitoring?
- Are there **automated alerts and remediation workflows** for high/critical CVEs?
- Are **base image dependencies reviewed and patched** regularly?

---

## 2. Identity and Access Management

### ✅ Azure RBAC

- Are Azure **Role-Based Access Controls (RBAC)** configured with the principle of least privilege?
- Are **AcrPull** and **AcrPush** roles assigned appropriately?
- Are **Managed Identities** used for applications accessing Azure resources?
- Is access to **AKS and ACR** reviewed periodically?

### ✅ Kubernetes RBAC

- Are **Kubernetes Roles and RoleBindings** defined per namespace?
- Are `ClusterAdmin` or `admin` roles restricted to authorized users?
- Are **default service accounts** disabled or limited?
- Are access attempts **logged and audited**?

---

## 3. Network Security

### ✅ Network Isolation

- Are workloads deployed in **dedicated subnets** within an Azure Virtual Network?
- Are **NSGs** used to restrict traffic at the subnet level?
- Is **egress traffic controlled** using UDRs or firewall rules?

### ✅ Kubernetes Network Policies

- Are **NetworkPolicies** implemented to allow only required pod-to-pod traffic?
- Is there a **default deny** rule for ingress/egress in each namespace?
- Is **Calico** or an equivalent CNI plugin used for network segmentation?

### ✅ Ingress/Egress Security

- Is traffic terminated with **TLS at the ingress layer**?
- Is a **Web Application Firewall (WAF)** enabled for public-facing endpoints?
- Are **API servers restricted by IP range** or private cluster enabled?

---

## 4. Secrets and Configuration Management

### ✅ Secret Storage

- Are **Kubernetes Secrets encrypted at rest** using customer-managed keys (CMK)?
- Are application secrets managed in **Azure Key Vault** instead of environment variables?
- Are **managed identities** used to fetch secrets securely?

### ✅ Secret Access Control

- Are access logs to **Key Vault** and Kubernetes Secrets **monitored and audited**?
- Are **secret rotation policies** in place for database credentials, API keys, and certificates?
- Is access to secrets based on **RBAC and need-to-know** basis?

---

## 5. Runtime Security and Host Hardening

### ✅ Container Runtime Settings

- Do containers run with **non-root users** by default?
- Are unnecessary **Linux capabilities dropped** via `securityContext`?
- Are **privileged containers**, `hostNetwork`, and `hostPID` restricted?

### ✅ Host OS Security

- Are AKS nodes based on **Azure Linux or a hardened OS image**?
- Is **auto-upgrade and patching** enabled for node pools?
- Are **unused ports and services** disabled on nodes?

### ✅ Runtime Threat Detection

- Is **Microsoft Defender for Containers** enabled for behavior monitoring?
- Are runtime policies defined for detecting shell access, port scanning, and lateral movement?
- Are open-source tools like **Falco** or **Sysdig Secure** in place for runtime visibility?

---

## 6. Logging, Monitoring, and Auditing

### ✅ Logging and Aggregation

- Are logs from containers and the control plane sent to **Azure Monitor or Log Analytics**?
- Is **Container Insights** enabled for performance metrics?
- Are logs **retained and archived** according to compliance policies?

### ✅ Alerting and Anomaly Detection

- Are there **alerts for CPU/memory spikes, OOMKills, and pod crashes**?
- Are **Kubernetes audit logs** enabled and exported?
- Are suspicious patterns (e.g., repeated login failures, unknown IPs) flagged?

### ✅ Centralized Audit and SIEM

- Are logs forwarded to a **SIEM** (e.g., Microsoft Sentinel)?
- Is there a **security operations playbook** for responding to incidents?
- Are **all administrative actions auditable**?

---

## 7. Policy Enforcement and Governance

### ✅ Azure Policy

- Are **Azure Policies** in place to enforce:
  - Trusted container registries?
  - Defender for Containers enablement?
  - AKS add-on configurations?

- Are policy **compliance results monitored regularly**?

### ✅ Admission Control

- Are **PodSecurity Standards (PSS)** configured in the AKS cluster?
- Is **OPA Gatekeeper or Kyverno** used to enforce custom policies?
- Are there controls that:
  - Prevent containers from running as root?
  - Enforce labels and annotations?
  - Block use of `hostPath` and wildcard image tags?

---

## 8. Supply Chain and CI/CD Security

### ✅ CI/CD Integration

- Are **image scanners integrated into CI/CD** pipelines?
- Is **static code analysis (SAST)** part of the build workflow?
- Are **pipeline credentials rotated** and stored securely?

### ✅ Image Signing and SBOM

- Are images **digitally signed** before deployment using tools like **Cosign**?
- Is a **Software Bill of Materials (SBOM)** generated and stored?
- Are SBOMs used to track CVEs in third-party dependencies?

---

## 9. Compliance and Secure Score

### ✅ Compliance Frameworks

- Does the deployment align with standards like:
  - CIS Kubernetes Benchmark?
  - Microsoft Cloud Security Benchmark (MCSB)?
  - ISO 27001, SOC 2, HIPAA, PCI DSS?

- Are regular **compliance scans** performed using **kube-bench** or Defender for Cloud?

### ✅ Defender for Cloud Secure Score

- Is the **Container Secure Score** monitored in Defender for Cloud?
- Are remediation steps from recommendations **tracked and implemented**?
- Is the environment integrated with **regulatory compliance dashboard** in Azure?

---

## 10. Backup, Recovery, and High Availability

### ✅ Backup and Restore

- Are cluster configurations and volume snapshots **backed up using Velero or Kasten**?
- Are **scheduled and on-demand backups** tested periodically?
- Are backups encrypted and stored in **geo-redundant Azure Blob Storage**?

### ✅ Disaster Recovery

- Is there a documented **DR strategy with RTO/RPO** defined?
- Is cross-region **AKS cluster replication** or **restore capability** tested?
- Is GitOps used to **redeploy infrastructure and workloads quickly**?

### ✅ High Availability

- Are AKS control planes deployed in **multiple availability zones**?
- Are **node pools auto-scaled** and equipped with **PodDisruptionBudgets**?
- Are health probes and **auto-healing features** enabled?

---

## Conclusion

This audit checklist provides a comprehensive view of security benchmarks for a containerized application deployment in Azure. Organizations should regularly perform self-assessments using these questions to ensure alignment with industry standards and cloud-native security best practices.

Using tools like **Microsoft Defender for Containers**, **Azure Policy**, **OPA Gatekeeper**, and **Key Vault**, along with structured logging and monitoring, creates a resilient, secure deployment pipeline that can withstand evolving threats.

---

## Ask me For Detailed Professional Audit Template for Your Project. Happy Audit Journey



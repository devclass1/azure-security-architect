# Specify Security Requirements for Containers in Azure

## Introduction

As enterprises move towards cloud-native architectures, containerization has become a foundational technology for building, deploying, and managing scalable applications. Microsoft Azure provides robust tools and services for container management such as Azure Kubernetes Service (AKS), Azure Container Instances (ACI), and Azure Container Registry (ACR). However, ensuring security in containerized environments requires more than just deployment; it demands a comprehensive approach to specifying and enforcing security requirements.

This article explores the essential security requirements for containers in Azure, addressing aspects such as identity and access management, image integrity, network policies, runtime protection, secrets management, logging, compliance, and more.

---

## 1. Container Image Security

### 1.1 Use Approved and Hardened Base Images

Using unverified or publicly available container images can introduce vulnerabilities into your deployment pipeline. It is recommended to:

- Use Microsoft-curated base images or vendor-verified images from Azure Marketplace.
- Establish an internal golden image repository with hardened, patched images.
- Regularly scan and patch base images as part of a vulnerability management lifecycle.

### 1.2 Azure Container Registry (ACR) Best Practices

When using Azure Container Registry:

- Enable Content Trust to enforce image signing and verification.
- Enable Geo-replication for global deployments while maintaining compliance.
- Integrate Microsoft Defender for Cloud to scan for known vulnerabilities in pushed images.
- Use network rules and private endpoints to restrict public access.

### 1.3 Image Scanning and Vulnerability Management

- Integrate image scanning tools such as Trivy, Aqua, or Qualys during CI/CD.
- Use Defender for Containers to detect vulnerabilities in running workloads.
- Ensure image scanning is enforced as part of CI policies before allowing deployments.

---

## 2. Access Control and Identity Management

### 2.1 Azure Role-Based Access Control (RBAC)

Implement RBAC at the container and registry levels to ensure least-privilege access:

| Role           | Description                         |
|----------------|-------------------------------------|
| AcrPull        | Pull-only access to ACR             |
| AcrPush        | Push and pull access to ACR         |
| AKS Cluster Admin | Full control over AKS resources |
| AKS Viewer     | Read-only access to AKS             |

- Assign roles at the resource group or namespace level.
- Use Managed Identities to securely authenticate services and avoid hardcoding credentials.

### 2.2 Kubernetes Role-Based Access Control (K8s RBAC)

Within AKS, use Kubernetes-native RBAC for fine-grained access:

- Define Roles and RoleBindings for namespaces.
- Restrict kubectl exec, port-forward, and access to secrets.
- Disable anonymous access and default service accounts where unnecessary.

---

## 3. Secure Network Policies

### 3.1 Network Isolation

- Use Azure Virtual Network (VNet) integration to isolate container workloads.
- Define NSGs (Network Security Groups) to restrict ingress/egress traffic.
- Deploy production and non-production workloads in separate subnets.

### 3.2 Kubernetes Network Policies

- Enforce NetworkPolicy resources to control pod-to-pod communication.
- Deny all traffic by default and allow only what’s required.
- Use Calico or Azure-native policies to implement microsegmentation.

### 3.3 Ingress and Egress Control

- Use Azure Application Gateway or NGINX ingress controller with WAF enabled.
- Implement egress policies to limit access to specific domains or IP ranges.
- Configure DNS policies to restrict unauthorized domain lookups.

---

## 4. Secure Secrets and Configurations

### 4.1 Use Azure Key Vault

- Store all sensitive configuration values like credentials and certificates in Azure Key Vault.
- Use managed identity for secure access to Key Vault from containers.

### 4.2 Kubernetes Secrets Management

- Avoid storing plain-text secrets in Kubernetes Secret objects.
- Enable encryption at rest using customer-managed keys (CMK) or Azure-managed keys.
- Rotate secrets regularly and audit access via Azure Monitor.

---

## 5. Host and Runtime Security

### 5.1 Secure AKS Node Pools

- Use Azure Linux or Hardened Ubuntu as base OS for AKS node pools.
- Enable automatic OS patching and node auto-upgrade.
- Use node taints and pod security standards to control workload placement.

### 5.2 Privilege and Capability Restrictions

- Prevent containers from running as root unless absolutely necessary.
- Use `securityContext` in pod specifications:

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  capabilities:
    drop: ["ALL"]
```

### 5.3 Runtime Threat Detection

- Enable Microsoft Defender for Containers to detect:
  - Shell access into containers
  - Crypto-mining
  - Suspicious traffic patterns
- Consider open-source tools like Falco for container behavior monitoring.

---

## 6. Logging, Auditing, and Monitoring

### 6.1 Centralized Logging

- Use Azure Monitor, Log Analytics, and Container Insights for log aggregation.
- Collect logs from:
  - Application stdout/stderr
  - AKS control plane via Diagnostic settings
  - Azure Container Registry access logs

### 6.2 Audit Trails

- Enable Azure Activity Logs for all resource operations.
- Capture Kubernetes audit logs to track user activity and system events.
- Forward logs to Microsoft Sentinel or other SIEM platforms.

### 6.3 Alerting and Notifications

- Define alert rules based on:
  - High CPU/memory usage
  - OOMKills or container restarts
  - Unauthorized access attempts
- Configure action groups for email, webhook, or Logic App notifications.

---

## 7. Policy Enforcement and Governance

### 7.1 Azure Policy for Containers

Use Azure Policy to define and enforce container-related constraints:

- Enforce use of trusted container registries.
- Ensure AKS clusters have Defender for Containers enabled.
- Block deployment of privileged or host-networked containers.

### 7.2 Gatekeeper and OPA

Deploy OPA Gatekeeper in your AKS cluster to enforce admission policies:

- Block containers that:
  - Don’t specify CPU/memory limits
  - Run as root
  - Use hostPath volumes

Example constraint:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-app-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels: ["app"]
```

---

## 8. Supply Chain Security

### 8.1 DevSecOps Integration

- Shift security left by integrating:
  - Static code analysis tools
  - Dockerfile linters
  - Image scanners (e.g., Trivy, Aqua)
  - Signing and verification using Notary v2 (coming to Azure)

### 8.2 Software Bill of Materials (SBOM)

- Use tools like Syft or SPDX to generate SBOMs during CI builds.
- Attach SBOM metadata to images in ACR.
- Use SBOMs to trace vulnerabilities during CVE announcements.

---

## 9. Compliance and Regulatory Alignment

### 9.1 Microsoft Cloud Security Benchmark (MCSB)

Align container workloads with MCSB guidelines for:

- Identity and access control
- Logging and monitoring
- Threat protection
- Data security and governance

### 9.2 Secure Score for Containers

- Enable Defender for Containers to contribute to Secure Score.
- Regularly review recommendations in Microsoft Defender for Cloud.

Examples of recommendations:
- Enable network policies in AKS
- Limit public access to container registries
- Apply security patches to running images

### 9.3 Regulatory Standards

Ensure Azure container environments meet compliance for:

- ISO 27001
- SOC 2 Type II
- GDPR
- HIPAA (for healthcare-related workloads)

---

## 10. Backup, Recovery, and Business Continuity

### 10.1 Backup of Workloads

- Use Velero to back up AKS cluster objects and persistent volumes.
- Store backups in encrypted Azure Blob Storage with lifecycle management.

### 10.2 Disaster Recovery

- Use multi-region AKS clusters or restore from snapshots in the event of failure.
- Use GitOps with FluxCD or ArgoCD for redeploying infrastructure configurations.

### 10.3 High Availability

- Design AKS clusters with multiple node pools and availability zones.
- Use health probes and auto-repair features for resiliency.

---

## Conclusion

Securing containers in Azure requires a layered defense approach. From build-time image scanning and identity control to runtime monitoring and threat detection, each step in the container lifecycle presents opportunities for hardening your environment.

Organizations should align with the Microsoft Cloud Security Benchmark and use native Azure tools like Defender for Cloud, Azure Policy, and Key Vault to minimize risks. Leveraging Kubernetes-native features such as RBAC, network policies, and OPA Gatekeeper ensures that containerized applications are secure by design and resilient to evolving threats.

Regular audits, automation, and proactive security governance are key to maintaining a strong container security posture in Azure’s cloud ecosystem.

---

## References

- [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/)
- [Azure Kubernetes Service Security](https://learn.microsoft.com/en-us/azure/aks/security-baseline)
- [Microsoft Defender for Containers](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-introduction)
- [Microsoft Cloud Security Benchmark](https://learn.microsoft.com/en-us/security/benchmark/azure/)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/)
- [Azure Policy Definitions](https://azure.microsoft.com/en-us/resources/templates/policy-definitions/)

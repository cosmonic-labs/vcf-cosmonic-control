# ArgoCD GitOps Repository Structure

This repository demonstrates a sample GitHub folder structure for deploying Cosmonic Control and WebAssembly workloads using ArgoCD ApplicationSets and HTTPTrigger custom resources.

## References

- [VMware Blog: VCF 9.0 for Edge - Automating App Deployment at Scale with GitOps using Argo CD](https://blogs.vmware.com/cloud-foundation/2025/07/29/vcf-9-0-for-edge-automating-app-deployment-at-scale-with-gitops-using-argo-cd/) by Dhruv Tyagi
- [VCF Edge ArgoCD Sample Repository](https://github.com/dhruv-tyagi-broadcom/vcf-edge-argo-cd-sample-repo)

## Directory Structure

### Cosmonic Configuration (`cosmonic/`)

**`cosmonic-control.yaml`** - ApplicationSet for deploying the Cosmonic Control Helm chart. Installs Cosmonic Console (UI optional), Nexus (API and NATS server), and Envoy. Only one instance is required per cluster.

**`cosmonic-hostgroup-a.yaml`** - Default Hostgroup configuration for hosting and running WebAssembly workloads. At least one Hostgroup is required per cluster. Multiple Hostgroups can be configured with independent resource allocations.

**`cosmonic-hostgroup-b.yaml`** - Example of an additional Hostgroup. WebAssembly workloads are scheduled to the first Hostgroup matching the selector criteria and possessing required capabilities.

**`wasm-apps.yaml`** - ArgoCD Application that recursively deploys all YAML manifests in the `apps/` folder. Suitable for rapid application development cycles where individual ArgoCD Applications are not necessary.

### Application Deployments (`apps/`)

**`apps/petstore-mcp/httptrigger.yaml`** - HTTPTrigger resource for the [Petstore-MCP example](https://github.com/cosmonic-labs/control-demos/tree/main/petstore-mcp). Based on the [Model Context Protocol (MCP) TypeScript template](https://github.com/cosmonic-labs/mcp-server-template-ts).

**`apps/wasm-application-demo/httptrigger.yaml`** - HTTPTrigger resource for the [Cosmonic Welcome Tour example](https://github.com/cosmonic-labs/control-demos/tree/main/welcome-tour).

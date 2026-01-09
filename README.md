# Cosmonic Control on VMware Cloud Foundation (VCF)

This repository provides instructions for installing Cosmonic Control on a VCF Kubernetes cluster, enabling the cluster to run both traditional pod deployments and WebAssembly (Wasm) workloads.

For an overview of the Cosmonic Control architecture, refer to the [Cosmonic Architecture Documentation](https://docs.cosmonic.com/architecture)

## Overview

This installation process for Cosmonic Control on VCF includes the following steps:

1. Deploy the Cosmonic Control and Cosmonic Control Hostgroup Helm charts
2. Configure Contour ingress to route subdomains to the Cosmonic Envoy service for xDS-based Wasm workload routing
3. Set up Contour ingress to the Cosmonic Perses UI for OpenTelemetry (OTEL) querying and monitoring

## Prerequisites

Before installing Cosmonic Control, ensure that Contour is configured as an ingress controller and Cert-Manager is installed (recommended for TLS support).

### VCF Cluster Setup

1. **Add VKS standard packages:**
   ```bash
   vcf package repository add vks-repo --url projects.packages.broadcom.com/vsphere/supervisor/packages/2025.10.22/vks-standard-packages:3.5.0-20251022 -n packages
   ```

2. **Install Cert-Manager:**
   ```bash
   vcf package install cert-manager -p cert-manager.kubernetes.vmware.com --version 1.18.2+vmware.2-vks.2 -n packages
   ```

3. **Install Contour:**

   Follow the [VMware VCF documentation](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vsphere-supervisor-services-and-standalone-components/latest/managing-vsphere-kuberenetes-service-clusters-and-workloads/installing-standard-packages-on-tkg-service-clusters/installing-standard-packages-on-tkg-cluster-using-tkr-for-vsphere-8-x/install-contour-with-envoy.html) for detailed Contour installation instructions.

   Retrieve and customize the default values file:
   ```bash
   vcf package available get contour.kubernetes.vmware.com/1.33.0+vmware.1-vks.1 --default-values-file-output contour-data-values.yaml -n packages
   ```

   A sample configuration is available in [vcf-contour/contour-data-values.yaml](vcf-contour/contour-data-values.yaml).

   Install Contour with the customized values:
   ```bash
   vcf package install contour -p contour.kubernetes.vmware.com --version 1.33.0+vmware.1-vks.1 --values-file contour-data-values.yaml -n packages
   ```

## Installing Cosmonic Control

Refer to the [Cosmonic Control documentation](https://docs.cosmonic.com/install-cosmonic-control#installation) for detailed installation guidelines.

### Option 1: Helm Installation

#### Basic Installation (Without Console UI)

Install Cosmonic Control with a NodePort service for the Envoy proxy:

```bash
helm install cosmonic-control oci://ghcr.io/cosmonic/cosmonic-control \
  --version 0.3.0 \
  --namespace cosmonic-system \
  --create-namespace \
  --set envoy.service.type=NodePort \
  --set envoy.service.httpNodePort=30950 \
  --set cosmonicLicenseKey="<insert-license-key>"
```

#### Installation With Console UI

To enable the Cosmonic Control Console UI, configure the `console_ui.enabled` and `hostName` parameters. A sample values file is available in [cosmonic-control/values.yaml](cosmonic-control/values.yaml).

```bash
helm install cosmonic-control oci://ghcr.io/cosmonic/cosmonic-control \
  --version 0.3.0 \
  --namespace cosmonic-system \
  --create-namespace \
  --set envoy.service.type=NodePort \
  --set envoy.service.httpNodePort=30950 \
  --set cosmonicLicenseKey="<insert-license-key>" \
  --set hostName="console.localhost.cosmonic.sh" \
  --set console_ui.enabled=true
```

#### Install Cosmonic Control Hostgroup

```bash
helm install hostgroup oci://ghcr.io/cosmonic/cosmonic-control-hostgroup \
  --version 0.3.0 \
  --namespace cosmonic-system
```

### Configure Contour Ingress

The [vcf-contour](vcf-contour/) folder contains ingress configurations for integrating Cosmonic Control with VCF's Contour ingress controller.

**[vcf-contour/vcf-nihao-example-deployment.yaml](vcf-contour/vcf-nihao-example-deployment.yaml):** This example demonstrates basic Contour ingress functionality. Refer to the [VMware Contour ingress documentation](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vsphere-supervisor-services-and-standalone-components/latest/managing-vsphere-kuberenetes-service-clusters-and-workloads/deploying-workloads-on-tkg-service-clusters/ingress-using-contour.html) for implementation details.

**[vcf-contour/cosmonic-ingress.yaml](vcf-contour/cosmonic-ingress.yaml):** Configures ingress for the Cosmonic Console and Perses UI in the `cosmonic-system` namespace. The default hostnames are `console.vcf.cosmonic.world` and `perses.vcf.cosmonic.world`. For testing, you may use `localhost.cosmonic.sh` which resolves to 127.0.0.1.

**[vcf-contour/vcf-wasm-ingress.yaml](vcf-contour/vcf-wasm-ingress.yaml):** Routes all `*.vcf-wasm.cosmonic.world` traffic to the Cosmonic Envoy ingress service. For production environments, consider specifying individual hosts instead of wildcards to reduce invalid hostname requests.

**[vcf-contour/wasm-ingress-examples.yaml](vcf-contour/wasm-ingress-examples.yaml):** Contains examples for specific domain routing as well as path-based routing to a Wasm workload. This can be used for when an application is containerized, but has components that are Sandboxed/Wasm workloads as well.

### Option 2: ArgoCD Installation

For a GitOps-based deployment approach using ArgoCD, see the [sample-argo-gh-folder-structure](sample-argo-gh-folder-structure/) directory for an example GitHub repository structure that demonstrates how to deploy both Cosmonic Control and sample Wasm workloads.

## Deploying WebAssembly Workloads

WebAssembly workloads are deployed by creating an HTTPTrigger resource. Examples are available in the [sample-argo-gh-folder-structure/apps](sample-argo-gh-folder-structure/apps/) folder.

For additional examples and deployment patterns, refer to the [Cosmonic Labs control-demos repository](https://github.com/cosmonic-labs/control-demos).

### Example: Using Helm to Deploy

The welcome-tour demo demonstrates deploying Wasm workloads using Helm. Refer to the [welcome-tour repository](https://github.com/cosmonic-labs/control-demos/tree/main/welcome-tour) for implementation details.

```bash
helm install welcome-tour --version 0.1.2 oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  -f https://raw.githubusercontent.com/cosmonic-labs/control-demos/refs/heads/main/welcome-tour/values.http-trigger.yaml
```

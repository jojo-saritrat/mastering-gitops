# 🚀 Helm Installation Guide

This document explains how to install Helm, the Kubernetes package manager, on different operating systems.

## Prerequisites

- Kubernetes cluster configured and running
- kubectl installed and configured

## Installation Methods

### 1. Install on macOS (Homebrew)

Run the following command in your terminal:

```
brew install helm
```

### 2. Install on Windows (Chocolatey)
Open PowerShell as Administrator and run:
```
choco install kubernetes-helm
```

# 📦  How to use Helm

## 🛠️ Prerequisites

- Helm 3 installed
- kubectl configured and connected to your Kubernetes cluster

## ✅ Steps

1. Open the **charts/values.yaml** file to edit the configuration values for your Helm chart. For example, you can modify the image settings in this file to change which container image version will be deployed. The **image** section looks like this by default

```yaml
image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""
```

2. Modify the **tag** field to specify the image version you want to deploy. For example, to use version **1.23** of nginx:
```yaml
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.23"
```

3. Install the Helm chart with the release name demo:

```
helm install demo charts
```
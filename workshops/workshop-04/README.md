# 🚀 Kustomize Installation Guide

Kustomize lets you customize raw, template-free YAML files for multiple purposes, managing overlays, patches, and configuration variations easily.

### Install Kustomize CLI

1. On Linux / macOS

To install Kustomize CLI using Homebrew (on macOS or Linux with Homebrew), just run this command:

```
brew install kustomize
```

1. On Windows

- Go to the [Kustomize releases page](https://github.com/kubernetes-sigs/kustomize/releases)
- Download the latest Windows **.exe** file, e.g., **kustomize_v5.1.1_windows_amd64.exe.**
- Rename it to **kustomize.exe** and place it somewhere in your system PATH.
- Open a command prompt and run:


```
kustomize version
```

### ✅ How to apply

1. Using kubectl **kustomize**

```
kubectl apply -k ./path-to-your-kustomize-folder
```

2. Using **kustomize** CLI

```
kustomize build <path-to-kustomization-dir> | kubectl apply -f -
```
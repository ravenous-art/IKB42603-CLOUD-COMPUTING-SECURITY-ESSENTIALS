# Lab 0: Environment Setup Report

## Course Information

**Course:** IKB42603 Cloud Computing Security Essentials  
**Lab:** Lab 0 - Environment Setup  
**Name:** YOUR FULL NAME HERE  
**Date:** 29 July 2026  

## Objective

The objective of this setup is to prepare the local lab environment required before Lab 1. Based on the setup cheatsheet, the environment must support Docker, AWS CLI v2, kind, kubectl, OpenSSL, oathtool, LocalStack, and a local Kubernetes cluster.

All services are intended to run locally. LocalStack is used as the local AWS simulator, and kind is used to run Kubernetes inside Docker.

## Environment Summary

Evidence is provided for the Kali environment.

| Component | Kali Verified Version / Status | Kali Proof |
|-----------|--------------------------------|------------|
| Docker | Docker version XX.X.X | `Evidence/kali/1.docker.png` |
| AWS CLI | aws-cli/2.XX.X | `Evidence/kali/2.awscli.png` |
| kind | kind v0.XX.0 | `Evidence/kali/3.kind.png` |
| kubectl | Client version v1.XX.X | `Evidence/kali/3.kubectl.png` |
| OpenSSL | OpenSSL 3.X.X | `Evidence/kali/4.Helper.png` |
| oathtool | OATH Toolkit 2.6.14 | `Evidence/kali/4.Helper.png` |
| LocalStack | Running and healthy on port 4566 | `Evidence/kali/5.localstack.png` |
| Kubernetes | kind cluster `ccse` running with node `ccse-control-plane` ready | `Evidence/kali/5.1.kubernetes.png` |
| AWS CLI LocalStack endpoint | Dummy credentials configured and STS caller identity tested through LocalStack | `Evidence/kali/6-config.png` |

## Step 1: Install and Verify Docker

Docker is required to run containers, LocalStack, and the kind Kubernetes cluster.

On Linux, Docker was installed using the official convenience script:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

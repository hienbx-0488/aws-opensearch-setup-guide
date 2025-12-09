# AWS OpenSearch Setup Guide

Comprehensive guide for setting up Amazon OpenSearch domains for development, staging, and production environments.

## Contents

### 1. [Dev/Stg Setup Guide](./opensearch-setup-with-images-dev-stg.md)
Complete step-by-step guide for creating an OpenSearch domain in development and staging environments.

**Configuration:**
- Instance: t3.medium.search (1 node) - 2 vCPU, 4 GB RAM
- Storage: EBS gp3, 100 GB
- Availability: 1 AZ, no standby
- Cost: ~$100-120/month

### 2. [Production Setup Guide](./opensearch-setup-with-images-prod.md)
Quick reference guide for production OpenSearch setup with high availability.

**Configuration:**
- Data nodes: r8g.medium.search (2 nodes) - 1 vCPU, 8 GB RAM each
- Master nodes: m7g.large.search (3 nodes) - 2 vCPU, 8 GB RAM each
- Storage: EBS gp3, 100 GB
- Availability: 2 AZ, without standby
- Cost: ~$585.53/month

## Quick Start

Both guides cover:
- Prerequisites and IAM permissions
- Security group configuration
- Domain creation on AWS Console
- Endpoint retrieval
- Connection testing
- Index creation and testing

## Prerequisites

- AWS Account with appropriate IAM permissions
- VPC and subnets for domain placement
- EC2 bastion/application servers for testing

## Images

All configuration screenshots are located in the `images/` folder.

## Getting Help

Refer to the detailed dev/stg guide for complete step-by-step instructions with screenshots. The production guide is a condensed reference assuming familiarity with the setup process.

---

Last updated: December 2024

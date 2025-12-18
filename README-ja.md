# AWS OpenSearch Setup Guide

開発、ステージング、本番環境向けのAmazon OpenSearchドメインセットアップの包括的なガイドです。

## Contents

### 1. [Dev/Stg Setup Guide](./opensearch-setup-with-images-dev-stg-ja.md)
開発・ステージング環境でOpenSearchドメインを作成するための完全なステップバイステップガイドです。

**Configuration:**
- Instance: t3.medium.search (1ノード) - 2 vCPU, 4 GB RAM
- Storage: EBS gp3, 100 GB
- Availability: 1 AZ、standbyなし
- Cost: 約$100-120/月

### 2. [Production Setup Guide](./opensearch-setup-with-images-prod-ja.md)
高可用性を備えた本番OpenSearchセットアップのクイックリファレンスガイドです。

**Configuration:**
- Data nodes: r8g.medium.search (2ノード) - 各1 vCPU, 8 GB RAM
- Master nodes: m7g.large.search (3ノード) - 各2 vCPU, 8 GB RAM
- Storage: EBS gp3, 100 GB
- Availability: 2 AZ、standbyなし
- Cost: 約$585.53/月

## Quick Start

両ガイドでカバーする内容:
- Prerequisites と IAM permissions
- Security group configuration
- AWS Consoleでのドメイン作成
- Endpoint取得
- 接続テスト
- Index作成とテスト

## Prerequisites

- 適切なIAM permissionsを持つAWSアカウント
- ドメイン配置用のVPCとsubnets
- テスト用のEC2 bastion/applicationサーバー

## Images

すべてのconfiguration screenshotsは `images/` フォルダに配置されています。

## Getting Help

完全なステップバイステップの手順とscreenshotsについては、詳細なdev/stgガイドを参照してください。Productionガイドは、setupプロセスに精通していることを前提とした簡潔なリファレンスです。

---

Last updated: December 2024

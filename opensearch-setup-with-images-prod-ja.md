# Amazon OpenSearch Production Setup

## Configuration

### Production の最小構成
| コンポーネント | 構成 |
|-----------|----------|
| **Engine Version** | OpenSearch 1.3 |
| **Instance** | r8g.medium.search (2ノード) |
| **Storage** | EBS gp3, 100 GB |
| **Availability** | 2 AZ、standbyなし |
| **Master** | m7g.large.search 3ノード |
| **Warm/Cold** | Off |
| **Security** | VPC private、SG 443のみ |
| **FGAC** | On、master user + password |
| **Encryption** | at-rest (KMS)、in-transit (TLS) |
| **Auto-Tune** | Off |
| **Snapshots** | Daily、03:00 UTC |

### コスト見積もり（ap-northeast-1）
- **2 × r8g.medium.search data nodes**: $172.28/月 (2 × $0.118/時間 × 730時間)
- **3 × m7g.large.search master nodes**: $383.25/月 (3 × $0.175/時間 × 730時間)
- **200 GB gp3 storage**: 約$30/月
- **合計**: 約$585.53/月

---

## Setup手順

![Production Configuration](./images/prod.png)

dev/stgガイドと同じ手順に従い、以下の重要な違いに注意:
- **Domain name**: `car360-opensearch-prod`
- **Security group**: `car360-opensearch-sg-prod`
- **Templates**: **Production** を選択
- **Deployment options**: **Domain without standby** を選択
- **Availability Zones**: **2** を選択
- **Data nodes**: r8g.medium.search × 2ノード
- **Master nodes**: m7g.large.search × 3ノード（dedicated masterを有効化）
- **Subnets**: 2つのprivate subnetsを選択（各AZに1つ）

# Amazon OpenSearch Production Setup

## Configuration

### Minimum Configuration for Production
| Component | Configuration |
|-----------|----------|
| **Engine Version** | OpenSearch 1.3 |
| **Instance** | r8g.medium.search (2 node) |
| **Storage** | EBS gp3, 100 GB |
| **Availability** | 2 AZ, without standby |
| **Master** | m7g.large.search 3 node |
| **Warm/Cold** | Off |
| **Security** | VPC private, SG 443 only |
| **FGAC** | On, master user + password |
| **Encryption** | at-rest (KMS), in-transit (TLS) |
| **Auto-Tune** | Off |
| **Snapshots** | Daily, 03:00 UTC |

### Cost Estimation (ap-northeast-1)
- **2 × r8g.medium.search data nodes**: $172.28/month (2 × $0.118/hour × 730 hours)
- **3 × m7g.large.search master nodes**: $383.25/month (3 × $0.175/hour × 730 hours)
- **200 GB gp3 storage**: ~$30/month
- **Total**: ~$585.53/month

---

## Setup Instructions

![Production Configuration](./images/prod.png)

Follow the same steps as dev/stg guide with these key differences:
- **Domain name**: `car360-opensearch-prod`
- **Security group**: `car360-opensearch-sg-prod`
- **Templates**: select **Production**
- **Deployment options**: select **Domain without standby**
- **Availability Zones**: select **2**
- **Data nodes**: r8g.medium.search × 2 nodes
- **Master nodes**: m7g.large.search × 3 nodes (enable dedicated master)
- **Subnets**: select 2 private subnets (one per AZ)

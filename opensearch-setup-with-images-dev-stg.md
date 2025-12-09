# Detailed Guide to Creating Amazon OpenSearch (dev/stg)

## 1) Prerequisites and Preparation
- **IAM Permissions**: can use the managed policy `AmazonOpenSearchServiceFullAccess` (dev) or a set of permissions to create domain, VPC, security group, and IAM.
- **VPC & Network**: should place domain in private VPC, access via bastion/NAT; avoid Public access.
- **Domain name template**: `car360-opensearch-dev`.
- **Region**: select region matching your BE system (e.g., ap-northeast-1 Tokyo).

### Minimum Configuration for dev/stg
| Component | Configuration |
|-----------|----------|
| **Engine Version** | OpenSearch 1.3 |
| **Instance** | t3.medium.search (1 node) |
| **Storage** | EBS gp3, 100 GB |
| **Availability** | 1 AZ, no standby |
| **Master** | Off (integrated with data nodes) |
| **Warm/Cold** | Off |
| **Security** | VPC private, SG 443 only |
| **FGAC** | On, master user + password |
| **Encryption** | at-rest (KMS), in-transit (TLS) |
| **Auto-Tune** | Off |
| **Snapshots** | Daily, 03:00 UTC |

### Cost Estimation (rough estimate, ap-northeast-1)
- **1 × t3.medium.search node**: ~$80-90/month (on-demand).
- **100 GB gp3 storage**: ~$15/month.
- **Data transfer (out)**: $0.14/GB if transferring outside VPC.
- **Total rough**: ~$100-120/month (minimum dev).

---

## 2) Create Security Group for OpenSearch
![Create Security Group](./images/sg.png)
- Create security group before entering the domain creation form:
    - Go to **EC2** -> **Security Groups** -> **Create security group**.
    - **Name**: `car360-opensearch-sg-dev`.
    - **Description**: `Security group for OpenSearch car360-opensearch-dev`.
    - **VPC**: select VPC where bastion/app servers are running.
    - **Inbound rules**:
      - **Type**: HTTPS
      - **Protocol**: TCP
      - **Port range**: 443
      - **Source**: select security group of EC2 backend/bastion or admin IP range (e.g., `10.0.0.0/16` or `sg-bastion-xxxxxxx`)
    - **Outbound rules**: keep default (All traffic, 0.0.0.0/0).
    - Click **Create security group**.

---

## 3) Create Domain on AWS Console (detailed form steps)

### Basic Steps

**Log in to AWS Console & Select Region**
- Log in to AWS Console -> select appropriate region (e.g., ap-northeast-1 Tokyo).

**Find OpenSearch Service & Create Domain**
- Find **OpenSearch Service** -> click **Create domain**.

![alt text](./images/image.png)

**Enter Domain Name**
- **Domain name**: enter `car360-opensearch-dev`.

**Choose Creation Method**
- **Creation method**: select **Standard create**.

---

### Templates & Deployment

**Choose Templates**
![alt text](./images/image-1.png)
- **Templates**: select **Dev/test** (for dev/stg, t3.medium, 1 AZ, no standby).

**Choose Deployment Options**
- **Deployment options**: select **Domain without standby** (no high availability standby).

**Choose Availability Zones**

- **Availability Zones**: select **1** (single-AZ for dev/stg, reduces cost).

---

### Engine & Nodes Configuration

**Choose Engine Version**
![alt text](./images/image-2.png)
- **Engine version**: select **OpenSearch 1.3**.
- **Enable compatibility mode**: true

**Configure Data Node (Instance family, Node type, Instance type)**
![alt text](./images/image-3.png)
- **Node configuration**:
    - **Instance family**: select **General purpose** (t3 general purpose; `r6i` memory optimized only for prod heavy).
    - **Instance type**: select **t3.medium.search** (balance CPU/RAM; `t3.large.search` if handling large volumes).
    - **Data nodes**: **1** node for dev/stg.

---

### Storage Configuration

**Configure Storage (EBS type, volume size, IOPS, Throughput)**
![alt text](./images/image-4.png)
- **Storage**:
    - **Storage type**: select **EBS** (check "Use EBS").
    - **EBS volume type**: select **gp3** (general purpose, cost optimized).
    - **EBS volume size**: enter **20** GB (minimum; increase for large data).
    - **IOPS**: keep default (3000, or custom if needed).
    - **Throughput**: keep default (125 MB/s, or custom).

---

### Advanced Nodes Configuration

**Configure Warm Storage**
![alt text](./images/image-5.png)
- **Warm storage** (hot nodes, frequently accessed):
    - **Use warm data nodes**: **Off** (no warm tier needed for dev).

**Configure Master Nodes**
![alt text](./images/image-6.png)
- **Enable dedicated master nodes**:
    - **Off** for dev/stg (no separate master needed).

**Configure Dedicated Coordinator Nodes**
![alt text](./images/image-7.png)
- **Enable dedicated coordinator nodes**:
    - **Off** for dev/stg (no separate coordinator needed).

**Configure Snapshot Configuration**
![alt text](./images/image-8.png)
- Keep default

**Configure Custom Endpoint**
![alt text](./images/image-9.png)
- **Custom endpoint**: **Off**

---

### Network & Access

**Configure Network Configuration**
![alt text](./images/image-17.png)
- **Network configuration**:
    - **Network**: select **VPC access** (not public).
    - **IP address type**: IPv4 only
    - **VPC**: select VPC where bastion/app servers are running.
    - **Subnets**: select 1 private subnet (matching 1 AZ).
    - **Security groups**: select security group just created `car360-opensearch-sg-dev`.


---

### Access Control & Security

**Configure Fine-grained Access Control**
![alt text](./images/image-11.png)
- **Fine-grained access control** (FGAC):
    - **Enable fine-grained access control**: **On**.
    - **Create master user**: **On** (use internal database).
    - **Master user name**: enter `admin` (or other name).
    - **Master user password**: enter strong password (at least 8 characters, supports special characters).

**Configure Authentication**
![alt text](./images/image-12.png)
- Keep default.

**Access Policies**
![alt text](./images/image-14.png)
  - **Only use fine-grained access control**: **On**.
---

**Encryption**
![alt text](./images/image-15.png)
- Keep default.

**Off-peak Window and Automatic Software Update**
![alt text](./images/image-16.png)
- Keep default.


### Review & Create

**Review Configuration**
![Review](./images/full.png)
- **Review**: review all configuration.

**Click Create Domain**
- **Create domain**: click **Create** -> wait for domain creation (usually 20-30 minutes).

**Monitor Domain Status**
- **Monitor**: go to **Domains** -> select `car360-opensearch-dev` -> view **Domain processing status**:
    - **Active**: domain ready for use.
    - **Creating/Modifying**: in creation process.

---

## 4) Get Endpoint Information After Creation
1. Go to domain `car360-opensearch-dev`.
2. Get **Domain endpoint** (HTTPS URL, e.g., `https://vpc-car360-opensearch-dev-sosornnipm7gbycwv2yat4jdhq.ap-northeast-1.es.amazonaws.com`).
3. Get **Kibana endpoint** (if needed for dashboard).

---

## 5) Test Connection After Creation
SSH to EC2 Backend and execute the following commands to test connection:

```bash
export ENDPOINT="https://vpc-car360-opensearch-dev-sosornnipm7gbycwv2yat4jdhq.ap-northeast-1.es.amazonaws.com"
export ADMIN_USER="admin"
export ADMIN_PASSWORD="password****"

# Check cluster health
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" "$ENDPOINT/_cluster/health"

# Expected result:
# {"cluster_name":"car360-opensearch-dev","status":"green","timed_out":false,...}
```

---

## 6) Create Sample Index and Test
```bash
# 1. Create "cars" index with 1 shard, 0 replicas (dev)
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" -X PUT \
  "$ENDPOINT/cars" \
  -H 'Content-Type: application/json' \
  -d '{
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }'

# 2. Add document
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" -X POST \
  "$ENDPOINT/cars/_doc" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Toyota Camry",
    "price": 28000,
    "category": "Sedan"
  }'

# 3. Add second document
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" -X POST \
  "$ENDPOINT/cars/_doc" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Honda CR-V",
    "price": 32000,
    "category": "SUV"
  }'

# 4. Search
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" \
  "$ENDPOINT/cars/_search?q=Toyota"

# Expected result:
# {"took":5,"timed_out":false,"_shards":...,"hits":{"total":{"value":1},"hits":[...]}}

# 5. View all documents
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" \
  "$ENDPOINT/cars/_search"

# Expected result:
# {"took":3,"timed_out":false,"_shards":...,"hits":{"total":{"value":2},"hits":[...]}}
```

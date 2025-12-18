# Amazon OpenSearch (dev/stg) 作成詳細ガイド

## 1) 前提条件と準備
- **IAM権限**: マネージドポリシー `AmazonOpenSearchServiceFullAccess`（dev用）を使用するか、ドメイン、VPC、Security Group、IAMを作成するための権限セットを使用できます。
- **VPC & ネットワーク**: ドメインはプライベートVPCに配置し、bastion/NAT経由でアクセス。Public accessは避けてください。
- **ドメイン名テンプレート**: `car360-opensearch-dev`
- **Region**: BEシステムに一致するリージョンを選択（例：ap-northeast-1 Tokyo）

### dev/stg の最小構成
| コンポーネント | 構成 |
|-----------|----------|
| **Engine Version** | OpenSearch 1.3 |
| **Instance** | t3.medium.search (1ノード) |
| **Storage** | EBS gp3, 100 GB |
| **Availability** | 1 AZ、standbyなし |
| **Master** | Off（データノードに統合） |
| **Warm/Cold** | Off |
| **Security** | VPC private、SG 443のみ |
| **FGAC** | On、master user + password |
| **Encryption** | at-rest (KMS)、in-transit (TLS) |
| **Auto-Tune** | Off |
| **Snapshots** | Daily、03:00 UTC |

### コスト見積もり（概算、ap-northeast-1）
- **1 × t3.medium.search ノード**: 約$80-90/月（on-demand）
- **100 GB gp3 storage**: 約$15/月
- **Data transfer（out）**: VPC外への転送の場合 $0.14/GB
- **合計概算**: 約$100-120/月（最小dev）

---

## 2) OpenSearch用Security Groupの作成
![Create Security Group](./images/sg.png)
- ドメイン作成フォームに入る前にSecurity Groupを作成:
    - **EC2** -> **Security Groups** -> **Create security group** へ移動
    - **Name**: `car360-opensearch-sg-dev`
    - **Description**: `Security group for OpenSearch car360-opensearch-dev`
    - **VPC**: bastion/appサーバーが稼働しているVPCを選択
    - **Inbound rules**:
      - **Type**: HTTPS
      - **Protocol**: TCP
      - **Port range**: 443
      - **Source**: EC2 backend/bastionのSecurity Groupまたは管理者IP range（例：`10.0.0.0/16` または `sg-bastion-xxxxxxx`）を選択
    - **Outbound rules**: デフォルトのまま（All traffic, 0.0.0.0/0）
    - **Create security group** をクリック

---

## 3) AWSコンソールでドメインを作成（詳細フォーム手順）

### 基本手順

**AWS Consoleにログイン & Region選択**
- AWS Consoleにログイン -> 適切なRegionを選択（例：ap-northeast-1 Tokyo）

**OpenSearch Serviceを検索 & ドメイン作成**
- **OpenSearch Service** を検索 -> **Create domain** をクリック

![alt text](./images/image.png)

**Domain nameを入力**
- **Domain name**: `car360-opensearch-dev` を入力

**Creation methodを選択**
- **Creation method**: **Standard create** を選択

---

### Templates & Deployment

**Templatesを選択**
![alt text](./images/image-1.png)
- **Templates**: **Dev/test** を選択（dev/stg用、t3.medium、1 AZ、standbyなし）

**Deployment optionsを選択**
- **Deployment options**: **Domain without standby** を選択（高可用性standbyなし）

**Availability Zonesを選択**

- **Availability Zones**: **1** を選択（dev/stg用single-AZ、コスト削減）

---

### Engine & Nodes Configuration

**Engine versionを選択**
![alt text](./images/image-2.png)
- **Engine version**: **OpenSearch 1.3** を選択
- **Enable compatibility mode**: true

**Data Nodeを構成（Instance family、Node type、Instance type）**
![alt text](./images/image-3.png)
- **Node configuration**:
    - **Instance family**: **General purpose** を選択（t3 汎用、`r6i` memory optimizedは本番の重い処理のみ）
    - **Instance type**: **t3.medium.search** を選択（CPU/RAMバランス、大量処理の場合は `t3.large.search`）
    - **Data nodes**: dev/stg用に **1** ノード

---

### Storage Configuration

**Storageを構成（EBS type、volume size、IOPS、Throughput）**
![alt text](./images/image-4.png)
- **Storage**:
    - **Storage type**: **EBS** を選択（「Use EBS」にチェック）
    - **EBS volume type**: **gp3** を選択（general purpose、コスト最適化）
    - **EBS volume size**: **20** GB を入力（最小、大量データの場合は増加）
    - **IOPS**: デフォルトのまま（3000、必要に応じてカスタム）
    - **Throughput**: デフォルトのまま（125 MB/s、カスタム可）

---

### Advanced Nodes Configuration

**Warm storageを構成**
![alt text](./images/image-5.png)
- **Warm storage**（hot nodes、頻繁にアクセス）:
    - **Use warm data nodes**: **Off**（devではWarm tier不要）

**Master nodesを構成**
![alt text](./images/image-6.png)
- **Enable dedicated master nodes**:
    - dev/stg用は **Off**（個別master不要）

**Dedicated coordinator nodesを構成**
![alt text](./images/image-7.png)
- **Enable dedicated coordinator nodes**:
    - dev/stg用は **Off**（個別coordinator不要）

**Snapshot configurationを構成**
![alt text](./images/image-8.png)
- デフォルトのまま

**Custom endpointを構成**
![alt text](./images/image-9.png)
- **Custom endpoint**: **Off**

---

### Network & Access

**Network configurationを構成**
![alt text](./images/image-17.png)
- **Network configuration**:
    - **Network**: **VPC access** を選択（publicではなく）
    - **IP address type**: IPv4 only
    - **VPC**: bastion/appサーバーが稼働しているVPCを選択
    - **Subnets**: 1つのprivate subnetを選択（1 AZに対応）
    - **Security groups**: 作成した `car360-opensearch-sg-dev` を選択


---

### Access Control & Security

**Fine-grained access controlを構成**
![alt text](./images/image-11.png)
- **Fine-grained access control** (FGAC):
    - **Enable fine-grained access control**: **On**
    - **Create master user**: **On**（internal databaseを使用）
    - **Master user name**: `admin` を入力（または他の名前）
    - **Master user password**: 強力なパスワードを入力（8文字以上、特殊文字対応）

**Authenticationを構成**
![alt text](./images/image-12.png)
- デフォルトのまま

**Access policies**
![alt text](./images/image-14.png)
  - **Only use fine-grained access control**: **On**
---

**Encryption**
![alt text](./images/image-15.png)
- デフォルトのまま

**Off-peak window and Automatic software update**
![alt text](./images/image-16.png)
- デフォルトのまま


### Review & Create

**Configurationを確認**
![Review](./images/full.png)
- **Review**: すべてのconfigurationを確認

**Create domainをクリック**
- **Create domain**: **Create** をクリック -> ドメイン作成を待つ（通常20-30分）

**Domain statusを監視**
- **Monitor**: **Domains** へ移動 -> `car360-opensearch-dev` を選択 -> **Domain processing status** を確認:
    - **Active**: ドメイン使用可能
    - **Creating/Modifying**: 作成処理中

---

## 4) 作成後のEndpoint情報取得
1. ドメイン `car360-opensearch-dev` へ移動
2. **Domain endpoint** を取得（HTTPS URL、例：`https://vpc-car360-opensearch-dev-sosornnipm7gbycwv2yat4jdhq.ap-northeast-1.es.amazonaws.com`）
3. **Kibana endpoint** を取得（dashboard用）

---

## 5) 作成後の接続テスト
EC2 BackendにSSHして以下のコマンドを実行し接続をテスト:

```bash
export ENDPOINT="https://vpc-car360-opensearch-dev-sosornnipm7gbycwv2yat4jdhq.ap-northeast-1.es.amazonaws.com"
export ADMIN_USER="admin"
export ADMIN_PASSWORD="password****"

# Cluster healthを確認
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" "$ENDPOINT/_cluster/health"

# 期待される結果:
# {"cluster_name":"car360-opensearch-dev","status":"green","timed_out":false,...}
```

---

## 6) Sample index作成とテスト
```bash
# 1. 1 shard、0 replicasで "cars" indexを作成 (dev)
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" -X PUT \
  "$ENDPOINT/cars" \
  -H 'Content-Type: application/json' \
  -d '{
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }'

# 2. Documentを追加
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" -X POST \
  "$ENDPOINT/cars/_doc" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Toyota Camry",
    "price": 28000,
    "category": "Sedan"
  }'

# 3. 2つ目のdocumentを追加
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

# 期待される結果:
# {"took":5,"timed_out":false,"_shards":...,"hits":{"total":{"value":1},"hits":[...]}}

# 5. すべてのdocumentsを表示
curl -u "$ADMIN_USER:$ADMIN_PASSWORD" \
  "$ENDPOINT/cars/_search"

# 期待される結果:
# {"took":3,"timed_out":false,"_shards":...,"hits":{"total":{"value":2},"hits":[...]}}
```

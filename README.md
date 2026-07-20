# JobMatch Kubernetes マニフェスト

このディレクトリには、JobMatchマイクロサービスをKubernetesにデプロイするためのマニフェスト（YAML）が含まれています。

## ディレクトリ構成

```
jobmatch-k8s/
├── namespace.yaml                      # Namespace定義
├── configmap.yaml                      # 環境変数設定
├── secret.yaml                         # シークレット（パスワード等）
├── database/
│   └── postgres-deployment.yaml        # PostgreSQL Deployment & Service
├── services/
│   ├── frontend-deployment.yaml        # React Frontend
│   ├── condition-service-deployment.yaml # FastAPI
│   ├── company-service-deployment.yaml   # Django
│   └── project-service-deployment.yaml   # Flask
├── ingress.yaml                        # Ingress設定
└── README.md                           # このファイル
```

## 前提条件

- Docker Desktop with Kubernetes enabled
- kubectl インストール済み
- 各サービスのDocker イメージがローカルに存在すること

## デプロイ手順

### 1. Docker イメージのビルド

```powershell
# jobmatch-frontend
cd C:\git\jobmatch-frontend
docker build -t jobmatch-frontend:latest -f docker/Dockerfile .

# condition-service (FastAPI)
cd C:\git\condition-service
docker build -t condition-service:latest .

# company-service (Django)
cd C:\git\company-service
docker build -t company-service:latest .

# project-service (Flask)
cd C:\git\project-service
docker build -t project-service:latest .
```

### 2. Kubernetes マニフェストのデプロイ

すべてのマニフェストを適用：

```powershell
# Namespaceを作成
kubectl apply -f namespace.yaml

# ConfigMapとSecretを作成
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# データベースをデプロイ
kubectl apply -f database/postgres-deployment.yaml

# サービスをデプロイ
kubectl apply -f services/frontend-deployment.yaml
kubectl apply -f services/condition-service-deployment.yaml
kubectl apply -f services/company-service-deployment.yaml
kubectl apply -f services/project-service-deployment.yaml

# Ingressを設定
kubectl apply -f ingress.yaml
```

### 3. デプロイ状況の確認

```powershell
# Podの状態確認
kubectl get pods -n jobmatch

# Serviceの確認
kubectl get svc -n jobmatch

# ログ確認
kubectl logs -n jobmatch -l app=condition-service
kubectl logs -n jobmatch -l app=company-service
kubectl logs -n jobmatch -l app=project-service
kubectl logs -n jobmatch -l app=frontend
kubectl logs -n jobmatch -l app=postgres
```

## アクセス方法

### Frontend
```
http://localhost:3000
```

### API エンドポイント

- Django (企業管理): http://localhost:8000
- FastAPI (条件管理): http://localhost:8001
- Flask (案件管理): http://localhost:8002

## トラブルシューティング

### Podが起動しない場合

```powershell
# Pod詳細確認
kubectl describe pod <pod-name> -n jobmatch

# ログ確認
kubectl logs <pod-name> -n jobmatch
```

### イメージが見つからない場合

Docker イメージがローカルにあるか確認：

```powershell
docker images | grep -E "jobmatch|condition|company|project"
```

### データベース接続エラー

PostgreSQL Podが起動しているか確認：

```powershell
kubectl get pods -n jobmatch -l app=postgres
kubectl logs -n jobmatch -l app=postgres
```

## 削除方法

全リソースを削除：

```powershell
kubectl delete namespace jobmatch
```

## 本番環境への展開

本番環境にデプロイする際は、以下を修正してください：

1. **Secret の管理**
   - Base64 でエンコードされたシークレットを実際の値に変更
   - 本番用のシークレットマネージャーを使用（AWS Secrets Manager等）

2. **イメージレジストリ**
   - `imagePullPolicy: Never` を削除
   - 実際のレジストリ（Docker Hub、ECR等）を指定

3. **リソース制限**
   - `resources.requests` と `resources.limits` を本番環境に合わせる

4. **レプリカ数**
   - 適切な `replicas` 数を設定

5. **Persistent Volume**
   - 本番環境のストレージクラスを指定

6. **Ingress**
   - SSL/TLS 設定を追加
   - ドメイン名を指定

## その他

詳細は各マニフェストファイルのコメントを参照してください。

# k8s Todo API 学習プロジェクト

## 構成

```
.
├── api/
│   ├── main.py          # FastAPI アプリ (Todo CRUD)
│   ├── requirements.txt
│   └── Dockerfile
└── k8s/
    ├── namespace.yaml
    ├── db/
    │   ├── secret.yaml      # DB 認証情報
    │   ├── pvc.yaml         # 永続ボリューム
    │   ├── deployment.yaml  # PostgreSQL (replicas: 1)
    │   └── service.yaml
    └── api/
        ├── deployment.yaml  # FastAPI (replicas: 2)
        └── service.yaml     # NodePort: 30080
```

## 起動手順

### 1. minikube 起動

```bash
minikube start
```

### 2. Docker イメージを minikube 内でビルド

```bash
# minikube の Docker デーモンに切り替え
eval $(minikube docker-env)

# イメージビルド
docker build -t todo-api:latest ./api
```

### 3. k8s リソースをデプロイ

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/db/
kubectl apply -f k8s/api/
```

### 4. 起動確認

```bash
kubectl get all -n todo-app
```

### 5. API にアクセス

```bash
# minikube の NodePort URL を取得
minikube service todo-api -n todo-app --url
```

ブラウザで `http://<上記URL>/docs` を開くと Swagger UI が確認できます。

## API エンドポイント

| Method | Path | 説明 |
|--------|------|------|
| GET | /health | ヘルスチェック |
| GET | /todos | Todo 一覧 |
| POST | /todos | Todo 作成 |
| GET | /todos/{id} | Todo 詳細 |
| PATCH | /todos/{id}/done | 完了にする |
| DELETE | /todos/{id} | 削除 |

## 動作確認例

```bash
BASE=$(minikube service todo-api -n todo-app --url)

# 作成
curl -X POST "$BASE/todos" -H "Content-Type: application/json" -d '{"title": "k8s を学ぶ"}'

# 一覧
curl "$BASE/todos"

# 完了
curl -X PATCH "$BASE/todos/1/done"
```

## 後片付け

```bash
kubectl delete namespace todo-app
minikube stop
```

# Helm Chart với Biến Môi Trường

Chart này hỗ trợ việc tùy chỉnh các giá trị thông qua biến môi trường, cho phép Jenkins pipeline thay đổi các giá trị khi triển khai.

## Cách Sử Dụng

### 1. Sử dụng script tự động

```bash
# Chạy script để tạo values.yaml từ template
./generate-values.sh
```

### 2. Thiết lập biến môi trường

Trước khi chạy script, bạn có thể thiết lập các biến môi trường:

```bash
# Ví dụ thiết lập các giá trị tùy chỉnh
export VOTE_IMAGE=myregistry/vote-app
export VOTE_TAG=v1.2.3
export VOTE_REPLICA_COUNT=3
export RESULT_IMAGE=myregistry/result-app
export RESULT_TAG=v1.2.3
export RESULT_REPLICA_COUNT=2
export WORKER_IMAGE=myregistry/worker-app
export WORKER_TAG=v1.2.3
export WORKER_REPLICA_COUNT=2
export DB_PASSWORD=mysecurepassword
export DB_PERSISTENCE_SIZE=5Gi
```

### 3. Sử dụng trong Jenkins Pipeline

Trong Jenkinsfile, bạn có thể thiết lập như sau:

```groovy
pipeline {
    environment {
        VOTE_IMAGE = "myregistry/vote-app"
        VOTE_TAG = "${BUILD_NUMBER}"
        VOTE_REPLICA_COUNT = "3"
        RESULT_IMAGE = "myregistry/result-app"
        RESULT_TAG = "${BUILD_NUMBER}"
        RESULT_REPLICA_COUNT = "2"
        WORKER_IMAGE = "myregistry/worker-app"
        WORKER_TAG = "${BUILD_NUMBER}"
        WORKER_REPLICA_COUNT = "2"
        DB_PASSWORD = credentials('db-password')
        DB_PERSISTENCE_SIZE = "5Gi"
    }
    
    stages {
        stage('Generate Values') {
            steps {
                sh '''
                    cd CD/helm
                    ./generate-values.sh
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    cd CD/helm
                    helm upgrade --install voting-app . -f values.yaml
                '''
            }
        }
    }
}
```

## Các Biến Môi Trường Hỗ Trợ

### Vote Service
- `VOTE_IMAGE`: Docker image cho vote service (default: dockersamples/examplevotingapp_vote)
- `VOTE_TAG`: Docker tag cho vote service (default: latest)
- `VOTE_REPLICA_COUNT`: Số lượng replica cho vote service (default: 1)

### Result Service
- `RESULT_IMAGE`: Docker image cho result service (default: dockersamples/examplevotingapp_result)
- `RESULT_TAG`: Docker tag cho result service (default: latest)
- `RESULT_REPLICA_COUNT`: Số lượng replica cho result service (default: 1)

### Worker Service
- `WORKER_IMAGE`: Docker image cho worker service (default: dockersamples/examplevotingapp_worker)
- `WORKER_TAG`: Docker tag cho worker service (default: latest)
- `WORKER_REPLICA_COUNT`: Số lượng replica cho worker service (default: 1)

### Redis
- `REDIS_IMAGE`: Docker image cho Redis (default: redis)
- `REDIS_TAG`: Docker tag cho Redis (default: alpine)

### Database
- `DB_IMAGE`: Docker image cho PostgreSQL (default: postgres)
- `DB_TAG`: Docker tag cho PostgreSQL (default: 15-alpine)
- `DB_USER`: Username cho database (default: postgres)
- `DB_PASSWORD`: Password cho database (default: postgres)
- `DB_PERSISTENCE_ENABLED`: Bật/tắt persistence (default: true)
- `DB_PERSISTENCE_SIZE`: Kích thước persistent volume (default: 1Gi)

## Lưu Ý

1. Script `generate-values.sh` sử dụng `envsubst` để thay thế biến môi trường
2. Nếu biến môi trường không được thiết lập, giá trị mặc định sẽ được sử dụng
3. File `values.template.yaml` chứa template với các biến môi trường
4. File `values.yaml` sẽ được tạo tự động từ template

## Yêu cầu

- `envsubst` command (thường có sẵn trong các hệ thống Linux)
- Bash shell 
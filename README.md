# MinIO Object Storage for ML Platform

S3-compatible object storage for your machine learning datasets, models, and artifacts.

## 🚀 Quick Start

1. **Start MinIO**:
   ```bash
   cd /Users/adminmac
   ./manage-sites.sh start-infra  # Ensure Traefik is running
   ./manage-sites.sh start minio-storage
   ```

2. **Access MinIO**:
   - **Web Console**: http://sites/minio
   - **API Endpoint**: http://sites/minio-api
   - **Username**: `minioadmin`
   - **Password**: `minioadmin123`

## 📁 Pre-configured Buckets

The following buckets are automatically created:

| Bucket | Purpose | Access |
|--------|---------|--------|
| `datasets` | Training/validation data | Public |
| `models` | Trained model files | Public |
| `artifacts` | Experiment artifacts | Private |
| `checkpoints` | Model checkpoints | Private |
| `experiments` | Experiment results | Private |

## 🔧 Integration with Jupyter

### **1. Add to Jupyter .env file**
```bash
# Add to /Users/adminmac/projects/jupyter-project/.env
MINIO_ENDPOINT=sites:80/minio-api
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin123
MINIO_SECURE=false
```

### **2. Python Usage in Notebooks**
```python
import os
from minio import Minio
import boto3

# Method 1: Using MinIO Python client
client = Minio(
    endpoint=os.getenv('MINIO_ENDPOINT'),
    access_key=os.getenv('MINIO_ACCESS_KEY'),
    secret_key=os.getenv('MINIO_SECRET_KEY'),
    secure=False
)

# Method 2: Using boto3 (S3-compatible)
s3_client = boto3.client(
    's3',
    endpoint_url=f'http://{os.getenv("MINIO_ENDPOINT")}',
    aws_access_key_id=os.getenv('MINIO_ACCESS_KEY'),
    aws_secret_access_key=os.getenv('MINIO_SECRET_KEY')
)
```

## 💾 ML Workflow Examples

### **Upload Dataset**
```python
# Upload training data
client.fput_object(
    bucket_name='datasets',
    object_name='train_data.csv',
    file_path='../data/train_data.csv'
)
```

### **Save Model**
```python
import joblib

# Save trained model
joblib.dump(model, 'model.pkl')
client.fput_object(
    bucket_name='models', 
    object_name='my_model_v1.pkl',
    file_path='model.pkl'
)
```

### **Load Model**
```python
# Download and load model
client.fget_object(
    bucket_name='models',
    object_name='my_model_v1.pkl', 
    file_path='downloaded_model.pkl'
)
model = joblib.load('downloaded_model.pkl')
```

### **Experiment Tracking**
```python
import json
from datetime import datetime

# Save experiment results
results = {
    'timestamp': datetime.now().isoformat(),
    'model': 'RandomForest',
    'accuracy': 0.95,
    'parameters': {'n_estimators': 100}
}

with open('experiment.json', 'w') as f:
    json.dump(results, f)

client.fput_object(
    bucket_name='experiments',
    object_name=f'experiment_{datetime.now().strftime("%Y%m%d_%H%M%S")}.json',
    file_path='experiment.json'
)
```

## 🔍 MinIO Console Features

Access the web console at http://sites/minio:

- **📁 Bucket Management**: Create, delete, configure buckets
- **📎 File Browser**: Upload, download, organize files
- **🔒 Access Control**: Manage bucket policies and permissions
- **📊 Monitoring**: View storage usage and metrics
- **🆔 User Management**: Create additional users and access keys

## 🚀 Advanced Usage

### **Bucket Policies**
```python
# Set bucket policy for public read access
policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {"AWS": "*"},
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::datasets/*"
    }]
}

client.set_bucket_policy('datasets', json.dumps(policy))
```

### **Presigned URLs**
```python
from datetime import timedelta

# Generate download URL (valid for 1 hour)
url = client.presigned_get_object(
    bucket_name='models',
    object_name='my_model_v1.pkl',
    expires=timedelta(hours=1)
)
print(f"Download URL: {url}")
```

### **Versioning**
```python
# Enable versioning
client.set_bucket_versioning('models', 'Enabled')

# Upload different versions
client.fput_object('models', 'my_model.pkl', 'model_v1.pkl')
client.fput_object('models', 'my_model.pkl', 'model_v2.pkl')
```

## 🚽 CLI Operations

### **Using MinIO Client (mc)**
```bash
# Configure alias
mc alias set local http://sites/minio-api minioadmin minioadmin123

# List buckets
mc ls local/

# Upload file
mc cp dataset.csv local/datasets/

# Download file  
mc cp local/models/my_model.pkl ./

# Sync directory
mc mirror ./models/ local/models/
```

## 🔐 Security Configuration

### **For Production Use:**

1. **Change Default Credentials**:
   ```yaml
   environment:
     - MINIO_ROOT_USER=your_secure_username
     - MINIO_ROOT_PASSWORD=your_secure_password_min_8_chars
   ```

2. **Create Application-Specific Users**:
   - Use MinIO console to create dedicated users
   - Grant minimal required permissions
   - Use separate credentials for different applications

3. **Enable HTTPS**:
   - Configure TLS certificates
   - Set `MINIO_SECURE=true` in client configuration

## 📈 Monitoring & Backup

### **Storage Usage**
```python
# Check bucket usage
for bucket in client.list_buckets():
    print(f"Bucket: {bucket.name}")
    
    objects = client.list_objects(bucket.name, recursive=True)
    total_size = sum(obj.size for obj in objects)
    print(f"Size: {total_size / (1024*1024):.2f} MB")
```

### **Backup Strategy**
```bash
# Backup entire MinIO data
mc mirror local/ backup-location/

# Incremental backup
mc mirror --newer-than 24h local/ backup-location/
```

## 💡 Integration with ML Libraries

### **MLflow Integration**
```python
import mlflow

# Configure MLflow to use MinIO
os.environ['MLFLOW_S3_ENDPOINT_URL'] = 'http://sites/minio-api'
os.environ['AWS_ACCESS_KEY_ID'] = 'minioadmin'
os.environ['AWS_SECRET_ACCESS_KEY'] = 'minioadmin123'

mlflow.set_tracking_uri('http://localhost:5000')
```

### **DVC Integration**
```yaml
# .dvc/config
remote.minio.url = s3://datasets
remote.minio.endpointurl = http://sites/minio-api
remote.minio.access_key_id = minioadmin
remote.minio.secret_access_key = minioadmin123
```

## 🔧 Management Commands

```bash
# Start MinIO
./manage-sites.sh start minio-storage

# Stop MinIO
./manage-sites.sh stop minio-storage

# View logs
./manage-sites.sh logs minio-storage

# Check status
./manage-sites.sh status
```

## 🎯 Use Cases

- **📊 Dataset Storage**: Centralized data repository
- **🤖 Model Registry**: Versioned model storage
- **📈 Experiment Artifacts**: Results, plots, logs
- **💾 Backup Storage**: Automated backups
- **🔄 Data Pipeline**: ETL intermediate storage
- **📱 Model Serving**: Serve models via HTTP

MinIO provides enterprise-grade object storage for your ML platform! 🚀


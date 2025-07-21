# Fraud Detection ML Pipeline

A comprehensive real-time fraud detection system built with Apache Airflow, Kafka, Spark, and MLflow. This project implements a complete ML pipeline for detecting fraudulent transactions using XGBoost and provides real-time inference capabilities.

## Architecture Overview

This system consists of several containerized microservices that work together to provide end-to-end fraud detection:

### Core Components

- **Apache Airflow**: Orchestrates the ML training pipeline with daily model retraining
- **Kafka**: Streams transaction data for real-time processing
- **Spark Streaming**: Processes transaction streams and applies ML models for inference
- **MLflow**: Tracks experiments, model artifacts, and provides model registry
- **MinIO**: S3-compatible object storage for model artifacts and data
- **PostgreSQL**: Backend database for Airflow and MLflow metadata
- **Redis**: Message broker for Airflow's Celery executor

### Pipeline Components

1. **Producer Service**: Generates synthetic transaction data and publishes to Kafka
2. **Training Pipeline**: Daily ML model training using XGBoost with hyperparameter optimization
3. **Inference Service**: Real-time fraud detection on streaming transactions
4. **Model Registry**: Centralized model versioning and artifact management

## Technology Stack

- **Orchestration**: Apache Airflow
- **Stream Processing**: Apache Spark, Kafka
- **Machine Learning**: XGBoost, scikit-learn, MLflow
- **Data Storage**: PostgreSQL, MinIO (S3-compatible)
- **Containerization**: Docker, Docker Compose
- **Programming**: Python 3.x

## Project Structure

```
├── airflow/                    # Airflow configuration and Dockerfile
│   ├── Dockerfile
│   └── requirements.txt
├── dags/                       # Airflow DAGs
│   ├── fraud_detection_training.py        # Training pipeline implementation
│   └── fraud_detection_training_dag.py    # Airflow DAG definition
├── inference/                  # Real-time inference service
│   ├── Dockerfile
│   ├── main.py                 # Spark streaming inference
│   └── requirements.txt
├── producer/                   # Transaction data generator
│   ├── Dockerfile
│   ├── main.py                 # Kafka producer for synthetic data
│   └── requirements.txt
├── mlflow/                     # MLflow server configuration
│   ├── Dockerfile
│   └── requirements.txt
├── models/                     # Trained model artifacts
│   └── fraud_detection_model.pkl
├── logs/                       # Airflow execution logs
├── plugins/                    # Airflow plugins (if any)
├── config.yaml                 # Central configuration file
├── docker-compose.yml          # Multi-service orchestration
└── init-multiple-dbs.sh        # Database initialization script
```

## Prerequisites

- Docker and Docker Compose
- At least 4GB of RAM
- 2+ CPU cores
- 10GB+ disk space

## Configuration

### Environment Variables

Create a `.env` file in the root directory with the following variables:

```bash
# Airflow Configuration
AIRFLOW_UID=50000
_AIRFLOW_WWW_USER_USERNAME=airflow
_AIRFLOW_WWW_USER_PASSWORD=airflow

# MinIO/S3 Configuration
MINIO_USERNAME=minio
MINIO_PASSWORD=minio123
AWS_ACCESS_KEY_ID=minio
AWS_SECRET_ACCESS_KEY=minio123

# Kafka Configuration (if using external Kafka)
KAFKA_BOOTSTRAP_SERVERS=your-kafka-server:9092
KAFKA_USERNAME=your-username
KAFKA_PASSWORD=your-password
```

### Configuration File

The `config.yaml` file contains ML and service configurations:

- MLflow tracking and model registry settings
- Kafka connection parameters
- XGBoost model hyperparameters
- Spark configuration for inference

## Getting Started

### 1. Clone and Setup

```bash
git clone <repository-url>
cd fraud-detection-pipeline
```

### 2. Create Environment File

```bash
cp .env.example .env
# Edit .env with your specific configuration
```

### 3. Start All Services

```bash
# Build and start all services
docker-compose up -d

# Check service status
docker-compose ps
```

### 4. Initialize Airflow

```bash
# Wait for services to be healthy (may take 2-3 minutes)
docker-compose logs airflow-init

# Access Airflow Web UI
open http://localhost:8080
# Login: airflow / airflow
```

### 5. Access Service UIs

- **Airflow**: http://localhost:8080 (airflow/airflow)
- **MLflow**: http://localhost:5500
- **MinIO**: http://localhost:9001 (minio/minio123)
- **Flower** (Celery monitoring): http://localhost:5555

## Running the Pipeline

### Manual Training

1. Navigate to Airflow UI (http://localhost:8080)
2. Find the `fraud_detection_training` DAG
3. Toggle it ON and trigger manually
4. Monitor execution in the Graph View

### Automatic Training

The pipeline runs daily at 3:00 AM UTC automatically. You can modify the schedule in `dags/fraud_detection_training_dag.py`:

```python
schedule_interval='0 3 * * *'  # Daily at 3 AM
```

### Real-time Inference

The inference service automatically starts consuming from Kafka and processes transactions in real-time. Monitor logs:

```bash
docker-compose logs -f inference
```

## Model Training Details

### Features Engineered

- Transaction amount and frequency patterns
- Time-based features (hour, day of week)
- User behavior patterns
- Location-based features
- Merchant category analysis

### Model Configuration

- **Algorithm**: XGBoost Classifier
- **Optimization**: Precision-focused (fraud detection priority)
- **Cross-validation**: 5-fold stratified
- **Class Imbalance**: SMOTE oversampling
- **Hyperparameter Tuning**: RandomizedSearchCV

### Performance Metrics

- Precision, Recall, F1-score
- PR-AUC (Area under Precision-Recall curve)
- Confusion Matrix
- Feature Importance Analysis

## Monitoring and Troubleshooting

### Viewing Logs

```bash
# All services
docker-compose logs

# Specific service
docker-compose logs airflow-scheduler
docker-compose logs inference
docker-compose logs producer

# Follow logs in real-time
docker-compose logs -f airflow-webserver
```

### Common Issues

1. **Memory Issues**: Ensure Docker has at least 4GB RAM allocated
2. **Port Conflicts**: Check if ports 8080, 5500, 9000, 9001 are available
3. **Permission Issues**: Set correct AIRFLOW_UID in .env file
4. **Database Connection**: Wait for PostgreSQL to be fully initialized

### Health Checks

```bash
# Check service health
docker-compose ps

# Check specific service logs
docker-compose logs <service-name>

# Restart specific service
docker-compose restart <service-name>
```

## Development

### Adding New Features

1. **Training Pipeline**: Modify `dags/fraud_detection_training.py`
2. **Inference Logic**: Update `inference/main.py`
3. **Data Generation**: Enhance `producer/main.py`

### Testing

```bash
# Run training pipeline locally
cd dags
python fraud_detection_training.py

# Test inference service
cd inference
python main.py
```

### Custom Configuration

Modify `config.yaml` for:
- Different model parameters
- Kafka topics and connection settings
- MLflow experiment names
- Spark configuration

## Data Flow

1. **Producer** generates synthetic transaction data → **Kafka**
2. **Training Pipeline** (Airflow) reads historical data → trains **XGBoost model** → stores in **MLflow**
3. **Inference Service** (Spark) reads from **Kafka** → applies model → writes predictions to **Kafka**
4. **MLflow** tracks experiments and model versions
5. **MinIO** stores model artifacts and training data

## Security Considerations

- Use strong passwords in production
- Configure proper Kafka authentication
- Set up network security groups
- Enable MLflow authentication for production
- Use secrets management for sensitive configuration

## Production Deployment

For production deployment:

1. Use managed services (AWS MSK for Kafka, RDS for PostgreSQL)
2. Implement proper monitoring with Prometheus/Grafana
3. Set up log aggregation (ELK stack)
4. Configure auto-scaling for inference services
5. Implement CI/CD pipelines for model deployment
6. Add data quality monitoring and alerting

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Support

For issues and questions:
- Check the troubleshooting section
- Review service logs
- Open an issue in the repository
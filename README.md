# Fraud Detection ML Pipeline

A complete fraud detection ML pipeline with MLflow, Airflow orchestration, and real-time inference capabilities.

## Current Status - Phase 2

Phase 2 implementation is complete! The project now includes full ML pipeline infrastructure with:

## Components

### Core Services
- **Airflow** - Workflow orchestration and scheduling
- **MLflow** - ML model tracking and registry
- **Inference Service** - Real-time fraud prediction API
- **Producer Service** - Data ingestion and processing

### Infrastructure
- `docker-compose.yml` - Multi-service Docker configuration
- `config.yaml` - Pipeline configuration
- `init-multiple-dbs.sh` - Database initialization script
- `wait-for-it.sh` - Service dependency management

### ML Pipeline
- `dags/fraud_detection_training_dag.py` - Airflow DAG for model training
- `dags/fraud_detection_training.py` - Training pipeline logic
- `models/` - Trained model artifacts

### Services
- `airflow/` - Airflow service configuration
- `mlflow/` - MLflow tracking server setup  
- `inference/` - Real-time prediction service
- `producer/` - Data producer service

## Getting Started

1. Clone the repository
2. Run `docker-compose up` to start all services
3. Access Airflow UI for pipeline management
4. Use MLflow for model tracking and versioning
5. Call inference endpoints for real-time predictions

## Architecture

The pipeline provides end-to-end fraud detection capabilities from data ingestion through model training to real-time inference, all orchestrated through Airflow with MLflow for experiment tracking.
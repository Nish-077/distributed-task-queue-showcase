# YADTQ (Yet Another Distributed Task Queue)

YADTQ is a distributed task queue system built with Python, using Kafka as the message broker and Redis as the result backend. It provides robust task distribution, worker management, and fault tolerance capabilities.

## Features

- Distributed task processing with multiple workers
- Automatic worker failure detection and recovery
- Task status tracking and result storage
- Round-robin task distribution
- Worker heartbeat monitoring
- Graceful shutdown handling
- Comprehensive logging

## Prerequisites

### Redis Installation

```bash
# Ubuntu/Debian
sudo apt-get install redis-server

# macOS
brew install redis

# Start Redis server
redis-server
```

### Kafka Installation

```bash
# Download Kafka
wget https://downloads.apache.org/kafka/3.5.1/kafka_2.13-3.5.1.tgz
tar -xzf kafka_2.13-3.5.1.tgz
cd kafka_2.13-3.5.1

# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka
bin/kafka-server-start.sh config/server.properties
```

## Installation

1. Clone the repository.

2. Install dependencies:
```bash
pip install --break-system-packages git+https://github.com/dpkp/kafka-python.git
pip install redis numpy colorlog
```

## Service Configuration

### Configure Kafka
- Navigate to Kafka server properties
```bash
sudo nano kafka/bin/server.properties
```
- Add following lines
```bash
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://localhost:9092
```

### Configure Redis
- Open Redis configuration file
```bash
sudo nano /etc/redis/redis.conf
```
- Add the following lines
```bash
bind localhost
```

### Restart Services
```bash
sudo systemctl restart kafka
sudo systemctl restart redis
```


## Architecture Details

### Worker System
The worker system is built with fault tolerance and automatic recovery:

- **Multi-threaded Processing**: Each worker runs 3 main threads:
  - Consumer Thread: Fetches tasks from Kafka
  - Processor Thread: Executes tasks
  - Heartbeat Thread: Sends health signals
- **Automatic Recovery**: Workers automatically recover from failures with configurable retry attempts
- **Partition Assignment**: Each worker is assigned to a specific Kafka partition for balanced task distribution

### Heartbeat Monitoring
The system includes a robust heartbeat monitoring mechanism:

- **Health Checking**: Workers emit periodic heartbeats to signal their health status
- **Failure Detection**: Client monitors heartbeats and detects worker failures
- **Automatic Recovery**: Failed workers are automatically restarted

### Task Queue Implementation

#### Kafka Setup
- Uses topic partitioning for parallel processing
- Configurable number of partitions
- Automatic topic creation and management

#### Redis Backend
- Stores task statuses and results
- Provides fast status lookups
- Maintains task metadata

## System Configurations

### Default Configuration
Default configuration in `config.py`:
```python
# Kafka Configuration
KAFKA_HOST = 'localhost'
KAFKA_PORT = 9092
KAFKA_SERVERS = [f'{KAFKA_HOST}:{KAFKA_PORT}']
KAFKA_TASK_TOPIC = 'yadtq_tasks'
KAFKA_HEARTBEAT_TOPIC = "worker-heartbeats"

# Redis Configuration
REDIS_HOST = 'localhost'
REDIS_PORT = 6379
REDIS_DB = 0
```

### Advanced Configuration
For production deployments:
```python
# Worker Configuration
MAX_WORKER_RETRIES = 3
HEARTBEAT_INTERVAL = 5
MISSED_HEARTBEATS_THRESHOLD = 3

# Task Queue Configuration
TASK_TIMEOUT = 10
AUTO_COMMIT_INTERVAL_MS = 1000
```

## Usage

### Basic Task Submission
```python
from client import YADTQClient

# Initialize client with 4 worker partitions
client = YADTQClient(num_partitions=4)

# Submit a task
task_id = client.submit_task('addition', args=[1, 2])

# Check task status
status = client.get_task_status(task_id)

# Get result
result = client.get_result(task_id)

# Clean up
client.close()
```

### Batch Task Processing
```python
# Submit multiple tasks
tasks = [
    {
        'task': 'addition',
        'args': [10, 20, 30],
        'kwargs': {'round_to': 2}
    },
    {
        'task': 'multiplication',
        'args': [2.5, 3.5],
        'kwargs': {'round_to': 1}
    }
]

task_ids = client.submit_tasks(tasks)
```

### Supported Task Types
- Basic arithmetic: addition, subtraction, multiplication, division
- Advanced math: power, root
- Matrix operations: matrix_addition, matrix_multiplication

## Error Handling

The system provides comprehensive error handling:

1. **Task Failures**: Failed tasks are marked and errors are stored
2. **Worker Failures**: Automatic worker recovery with configurable retries
3. **Connection Issues**: Automatic reconnection attempts for both Kafka and Redis
4. **Invalid Tasks**: Validation and appropriate error responses

## Monitoring and Management

### Task Status Monitoring
```python
# Get all tasks
all_tasks = client.get_all_tasks()

# Get tasks by status
failed_tasks = client.get_tasks_by_status(TaskStatus.FAILED)
queued_tasks = client.get_tasks_by_status(TaskStatus.QUEUED)
```

### Worker Management
```python
# Check all workers health
worker_status = client.get_all_workers_health()

# Clear all tasks
client.clear_all_tasks()
```

## Error Codes & Troubleshooting

Common error codes and their solutions:

- `KAFKA_CONNECTION_ERROR`: Check if Kafka server is running and accessible
- `REDIS_CONNECTION_ERROR`: Verify Redis connection settings and server status
- `WORKER_INITIALIZATION_FAILED`: Check logs for specific worker startup errors
- `TASK_SUBMISSION_TIMEOUT`: Increase timeout value or check network connectivity
- `WORKER_RECOVERY_FAILED`: Check max retries and worker logs


## Installation Script

To install all necessary dependencies and set up the environment, you can use the provided `setup.sh` script. Follow these steps:

1. **Download the `setup.sh` script**:
    ```sh
    wget https://github.com/Cloud-Computing-Big-Data/RR-Team-45-yadtq-yet-another-distributed-task-queue-.git/setup.sh
    ```

2. **Make the script executable**:
    ```sh
    chmod +x setup.sh
    ```

3. **Run the script**:
    ```sh
    ./setup.sh
    ```

The `setup.sh` script will handle the following tasks:
- Install required system packages
- Set up a virtual environment
- Install Python dependencies
- Configure Kafka and Redis

### Logging

The system uses colored logging for better visibility:
```python
# Log levels are color-coded:
# DEBUG: cyan
# INFO: green
# WARNING: yellow
# ERROR: red
# CRITICAL: red with white background
```

# YADTQ (Yet Another Distributed Task Queue)

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Apache%20Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white" alt="Kafka"/>
  <img src="https://img.shields.io/badge/redis-%23DD0031.svg?&style=for-the-badge&logo=redis&logoColor=white" alt="Redis"/>
</p>

YADTQ is a robust distributed task queue system built from the ground up in Python. It uses Apache Kafka as a high-throughput message broker and Redis as a low-latency result backend, inspired by industry-standard systems like Celery. This project demonstrates a deep understanding of distributed systems, fault tolerance, and asynchronous processing.

**Note:** The source code for this project is hosted in a private college organization repository. This showcase provides a detailed overview of the architecture, features, and my contributions.

***

## Key Features
- ✅ **High Fault Tolerance:** Automatic worker failure detection and recovery with configurable retry attempts ensures system resilience.
- ✅ **Horizontal Scalability:** Natively supports multiple workers, with tasks distributed across Kafka partitions for parallel processing.
- ✅ **Real-time Health Monitoring:** A robust heartbeat mechanism allows the client to monitor worker health in real-time and automatically restart failed nodes.
- ✅ **Asynchronous Task Management:** Submit tasks and track their status (e.g., `QUEUED`, `IN_PROGRESS`, `FAILED`, `SUCCESS`) via a fast Redis backend.
- ✅ **Graceful Shutdown & Error Handling:** Workers handle interruptions gracefully, and the system includes comprehensive error handling for task failures and connection issues.

## Architecture Overview

The system is designed with decoupled components to ensure scalability and maintainability.

<p align="center">
  <img src="https_your_image_link_here" alt="YADTQ Architecture Diagram" width="700"/>
</p>

1.  **Client:** The entry point for the user. It submits tasks to the Kafka message broker and queries Redis for results.
2.  **Kafka (Message Broker):** Ingests tasks and distributes them across different partitions. This allows multiple workers to consume and process tasks in parallel.
3.  **Workers:** The core processing units. Each worker is a multi-threaded Python application that:
    * **Consumes** tasks from its assigned Kafka partition.
    * **Processes** the task (e.g., performs a calculation).
    * **Updates** Redis with the task status and result.
    * **Emits** periodic heartbeats to a dedicated Kafka topic to signal its health.
4.  **Redis (Result Backend):** Stores the final result and status of every task, providing fast and efficient lookups for the client.

## My Role & Contributions
As a lead contributor on this team project, my primary responsibilities were architecting the core fault-tolerance and worker management systems. Specifically, I:
- **Designed and implemented the multi-threaded worker system**, separating task consumption, processing, and heartbeat signals for concurrent operation.
- **Engineered the heartbeat monitoring and failure detection logic** on the client side, enabling the system to identify and handle worker failures automatically.
- **Owned the Kafka integration**, including topic and partition management for balanced task distribution across the worker pool.
- **Developed the result backend interface with Redis**, ensuring atomic updates for task statuses and efficient result retrieval.

## Demo: Submitting a Task

Here is a simple demonstration of the client's functionality.

![Demo GIF showing task submission and result retrieval](https_your_gif_link_here)

The client API is designed to be intuitive and simple:
```python
from client import YADTQClient

# Initialize client, which automatically provisions 4 worker partitions
client = YADTQClient(num_partitions=4)

# Submit a task for execution by a worker
task_id = client.submit_task('addition', args=[100, 25])
print(f"Task {task_id} submitted.")

# Check task status and retrieve the result (blocking call)
print(f"Waiting for result...")
result = client.get_result(task_id)

print(f"Result: {result}") # Output: Result: 125

# Clean up connections
client.close()
```

## Setup and Installation

To keep this README focused on the project's architecture and impact, detailed setup, configuration, and usage instructions have been moved to a separate document.

> ### ➡️ [View Full Installation & Usage Guide](./INSTALL.md)

***

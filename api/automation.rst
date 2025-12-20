Scheduler & Automation API
========================================================

## Overview
----------

The Scheduler & Automation API provides comprehensive scheduling capabilities for the Univa Gateway platform. It enables automated job execution based on time, intervals, events, or cron expressions, allowing for autonomous gateway behavior without manual intervention.

## Base URL
-----------

.. code-block:: http

   https://univa-gateway/api/v1/scheduler

## Authentication
-----------------

All endpoints require JWT authentication via the ``Authorization`` header:

.. code-block:: http

   Authorization: Bearer <jwt_token>

---

## Jobs Management API
----------------------

### Get All Jobs
~~~~~~~~~~~~~~~

.. http:get:: /jobs

   Retrieve list of all scheduled jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional): Filter by status (``active``, ``inactive``, ``error``)
   * **type** (optional): Filter by job type (``interval``, ``daily``, ``weekly``, ``monthly``, ``cron``, ``sunrise``)
   * **tags** (optional): Comma-separated list of tags
   * **limit** (optional): Number of jobs to return (default: 50)
   * **offset** (optional): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "total_jobs": 4,
        "active_jobs": 3,
        "jobs": [
          {
            "id": "job_001",
            "name": "Modbus Poll",
            "description": "Poll Modbus devices for sensor data",
            "type": "interval",
            "enabled": true,
            "status": "active",
            "schedule": {
              "interval_seconds": 10,
              "next_execution": "2025-03-12T15:10:30Z"
            },
            "action": {
              "type": "publish",
              "topic": "device/data/modbus",
              "payload_template": "{\"timestamp\": \"${NOW}\", \"device_id\": \"${DEVICE_ID}\"}"
            },
            "tags": ["modbus", "polling", "telemetry"],
            "created_at": "2025-03-01T08:00:00Z",
            "last_execution": "2025-03-12T15:10:20Z",
            "execution_count": 1205,
            "average_duration_ms": 125,
            "run_url": "/api/v1/scheduler/jobs/job_001/run"
          }
        ],
        "pagination": {
          "limit": 50,
          "offset": 0,
          "has_more": false
        }
      }

### Create New Job
~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs

   Create a new scheduled job.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Health Check",
        "description": "Run system health diagnostics",
        "type": "daily",
        "enabled": true,
        "schedule": {
          "time": "01:00",
          "timezone": "UTC",
          "days_of_week": ["mon", "tue", "wed", "thu", "fri"]
        },
        "action": {
          "type": "system",
          "operation": "health_check",
          "parameters": {
            "check_services": true,
            "check_storage": true,
            "check_network": true
          }
        },
        "advanced": {
          "retry_count": 3,
          "retry_interval_seconds": 2,
          "stop_on_error": false,
          "priority": 5,
          "timeout_seconds": 30
        },
        "tags": ["health", "diagnostics", "maintenance"],
        "notifications": {
          "on_success": false,
          "on_failure": true,
          "on_timeout": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "job_id": "job_003",
        "message": "Job created successfully",
        "job": {
          "id": "job_003",
          "name": "Health Check",
          "enabled": true,
          "status": "active",
          "schedule": {
            "next_execution": "2025-03-13T01:00:00Z"
          },
          "action": {
            "type": "system",
            "operation": "health_check"
          },
          "webhook_url": "/api/v1/scheduler/jobs/job_003/run",
          "status_url": "/api/v1/scheduler/jobs/job_003/status"
        }
      }

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 400 Bad Request
      Content-Type: application/json
      
      {
        "error": "Invalid schedule",
        "message": "Schedule conflicts with existing job: Modbus Poll",
        "code": "SCHEDULE_CONFLICT",
        "conflicts": [
          {
            "job_id": "job_001",
            "job_name": "Modbus Poll",
            "conflict_period": "01:00-01:05"
          }
        ]
      }

### Get Job Details
~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}

   Retrieve detailed information about a specific job.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "job": {
          "id": "job_001",
          "name": "Modbus Poll",
          "description": "Poll Modbus devices for sensor data",
          "type": "interval",
          "enabled": true,
          "status": "active",
          "configuration": {
            "schedule": {
              "interval_seconds": 10,
              "jitter_percentage": 10,
              "start_immediately": true
            },
            "action": {
              "type": "publish",
              "topic": "device/data/modbus",
              "payload": {
                "template": "{\"timestamp\": \"${NOW}\", \"device_id\": \"${DEVICE_ID}\"}",
                "variables": {
                  "NOW": "2025-03-12T15:10:30Z",
                  "DEVICE_ID": "modbus_master_01"
                }
              },
              "qos": 1,
              "retain": false
            },
            "advanced": {
              "retry_count": 3,
              "retry_interval_seconds": 2,
              "stop_on_error": false,
              "priority": 5,
              "timeout_seconds": 30,
              "max_concurrent": 1
            }
          },
          "statistics": {
            "total_executions": 1205,
            "successful_executions": 1198,
            "failed_executions": 7,
            "average_duration_ms": 125,
            "last_execution": "2025-03-12T15:10:20Z",
            "last_duration_ms": 120,
            "last_status": "success",
            "last_error": null,
            "next_execution": "2025-03-12T15:10:30Z",
            "estimated_storage_impact_mb": 0.5
          },
          "metadata": {
            "created_by": "admin",
            "created_at": "2025-03-01T08:00:00Z",
            "updated_by": "admin",
            "updated_at": "2025-03-10T14:30:00Z",
            "tags": ["modbus", "polling", "telemetry"]
          }
        }
      }

### Update Job
~~~~~~~~~~~~~

.. http:put:: /jobs/{id}

   Update an existing job configuration.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /jobs/job_001 HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "schedule": {
          "interval_seconds": 30
        },
        "action": {
          "payload": {
            "template": "{\"timestamp\": \"${NOW}\", \"device_id\": \"${DEVICE_ID}\", \"interval\": \"30s\"}"
          }
        },
        "tags": ["modbus", "polling", "telemetry", "optimized"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Job updated successfully",
        "job_id": "job_001",
        "changes": {
          "schedule.interval_seconds": "10 → 30",
          "tags": "Added: optimized"
        },
        "next_execution": "2025-03-12T15:11:00Z"
      }

### Delete Job
~~~~~~~~~~~~~

.. http:delete:: /jobs/{id}

   Permanently delete a scheduled job.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Job deleted successfully",
        "deleted_job": "job_001",
        "statistics": {
          "total_executions": 1205,
          "total_runtime_hours": 41.8
        }
      }

### Enable/Disable Job
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/{id}/toggle

   Enable or disable a job without deleting it.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/job_001/toggle HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "reason": "System maintenance completed"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Job enabled successfully",
        "job_id": "job_001",
        "enabled": true,
        "next_execution": "2025-03-12T15:10:30Z"
      }

---

## Job Execution API
--------------------

### Run Job Immediately
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/{id}/run

   Manually trigger job execution.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/job_001/run HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "force": true,
        "parameters": {
          "override_topic": "device/data/manual",
          "debug": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "execution_id": "exec_20250312151030123",
        "job_id": "job_001",
        "status": "started",
        "trigger": "manual",
        "start_time": "2025-03-12T15:10:30Z",
        "estimated_duration_ms": 120,
        "progress_url": "/api/v1/scheduler/executions/exec_20250312151030123"
      }

### Get Execution Status
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /executions/{id}

   Check status of a job execution.

   **Path Parameters**:

   * **id** (string): Execution ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "execution": {
          "id": "exec_20250312151030123",
          "job_id": "job_001",
          "job_name": "Modbus Poll",
          "status": "completed",
          "trigger": "manual",
          "start_time": "2025-03-12T15:10:30Z",
          "end_time": "2025-03-12T15:10:32Z",
          "duration_ms": 120,
          "result": {
            "status": "success",
            "message": "Modbus data collected successfully",
            "data_points": 24,
            "bytes_published": 512
          },
          "logs": [
            {
              "timestamp": "2025-03-12T15:10:30.123Z",
              "level": "info",
              "message": "Starting Modbus poll"
            }
          ]
        }
      }

### Get Execution History
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}/history

   Retrieve history of job executions.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **start_date** (optional): ISO date string
   * **end_date** (optional): ISO date string
   * **status** (optional): Filter by status (``success``, ``failed``, ``timeout``)
   * **limit** (optional): Number of records (default: 50)
   * **offset** (optional): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "job_id": "job_001",
        "job_name": "Modbus Poll",
        "total_executions": 1205,
        "success_rate": 99.4,
        "history": [
          {
            "execution_id": "exec_20250312151030123",
            "timestamp": "2025-03-12T15:10:30Z",
            "trigger": "scheduled",
            "duration_ms": 120,
            "status": "success",
            "message": "Modbus data collected successfully",
            "metrics": {
              "data_points": 24,
              "bytes_published": 512,
              "cpu_usage_percent": 12.5,
              "memory_usage_mb": 15.2
            }
          }
        ],
        "pagination": {
          "limit": 50,
          "offset": 0,
          "has_more": true
        }
      }

---

## Schedule Types API
---------------------

### Create Interval Job
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/interval

   Create a job that runs at fixed intervals.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/interval HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Temperature Monitor",
        "interval_seconds": 60,
        "jitter_percentage": 10,
        "action": {
          "type": "publish",
          "topic": "sensors/temperature",
          "payload": "{\"temp\": ${TEMP_READING}}"
        }
      }

### Create Daily Job
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/daily

   Create a job that runs daily at specific time.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/daily HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Daily Backup",
        "time": "02:00",
        "timezone": "UTC",
        "skip_weekends": true,
        "action": {
          "type": "system",
          "operation": "backup"
        }
      }

### Create Weekly Job
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/weekly

   Create a job that runs weekly on specific days.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/weekly HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Weekly Report",
        "days_of_week": ["mon", "wed", "fri"],
        "time": "09:00",
        "action": {
          "type": "file",
          "operation": "generate_report"
        }
      }

### Create Cron Job
~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/cron

   Create a job with cron expression.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/cron HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Complex Schedule",
        "cron_expression": "0 9-17 * * 1-5",
        "timezone": "America/New_York",
        "action": {
          "type": "publish",
          "topic": "status/heartbeat"
        }
      }

### Create Sunrise/Sunset Job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/sun

   Create a job that runs at sunrise/sunset.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/sun HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Evening Lights",
        "mode": "sunset",
        "offset_minutes": -30,
        "latitude": 40.7128,
        "longitude": -74.0060,
        "action": {
          "type": "output",
          "relay": "lights",
          "value": "on"
        }
      }

---

## Action Types Configuration
------------------------------

### Action Types

.. list-table:: Action Types
   :widths: 20 40 40
   :header-rows: 1

   * - Type
     - Description
     - Example Configuration
   * - **publish**
     - Publish data to MQTT topic
     - .. code-block:: json
          {
            "type": "publish",
            "topic": "device/data",
            "payload": "{\"value\": ${DATA}}",
            "qos": 1
          }
   * - **log**
     - Write log messages
     - .. code-block:: json
          {
            "type": "log",
            "message": "Job executed",
            "level": "info"
          }
   * - **output**
     - Control GPIO or relays
     - .. code-block:: json
          {
            "type": "output",
            "relay": "relay_1",
            "value": "on"
          }
   * - **system**
     - Execute system operations
     - .. code-block:: json
          {
            "type": "system",
            "operation": "restart"
          }
   * - **file**
     - Perform file operations
     - .. code-block:: json
          {
            "type": "file",
            "operation": "rotate",
            "pattern": "*.log"
          }
   * - **rule**
     - Trigger rule engine
     - .. code-block:: json
          {
            "type": "rule",
            "rule_node": "alert"
          }

---

## Conflict Detection API
-------------------------

### Check for Conflicts
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /conflicts/check

   Detect scheduling conflicts between jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /conflicts/check HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "job_id": "job_new",
        "schedule": {
          "type": "daily",
          "time": "02:00"
        },
        "estimated_duration_seconds": 300
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "has_conflicts": true,
        "conflicts": [
          {
            "job_id": "job_001",
            "job_name": "Daily Backup",
            "conflict_type": "time_overlap",
            "conflict_period": "02:00-02:05",
            "severity": "medium"
          }
        ],
        "recommendations": [
          "Schedule at 02:10 to avoid conflict"
        ]
      }

### Get System Resource Usage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /resources/usage

   Check system resource utilization by jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "resource_usage": {
          "max_concurrent_jobs": 5,
          "current_concurrent_jobs": 3,
          "cpu_usage_percent": {
            "average": 28.5,
            "peak": 65.2
          },
          "memory_usage_mb": {
            "total": 256,
            "used": 124
          }
        }
      }

---

## Bulk Operations API
----------------------

### Run All Jobs
~~~~~~~~~~~~~~~~

.. http:post:: /jobs/run/all

   Manually trigger all enabled jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/run/all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "exclude_tags": ["maintenance"],
        "force": false,
        "sequential": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "execution_group_id": "group_20250312151500",
        "total_jobs": 3,
        "started_jobs": 3,
        "status": "started"
      }

### Disable All Jobs
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/disable/all

   Disable all scheduled jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/disable/all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "reason": "System maintenance",
        "duration_minutes": 60
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "All jobs disabled",
        "disabled_count": 12
      }

### Export Jobs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/export

   Export all job configurations.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/export HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "format": "json",
        "include_history": false,
        "include_statistics": true
      }

   **Response**: File download with Content-Type: ``application/json``

### Import Jobs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/import

   Import jobs from configuration file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /jobs/import HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="configFile"; filename="jobs.json"
      Content-Type: application/json
      
      {
        "strategy": "merge",
        "dry_run": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "import_id": "import_20250312152000",
        "processed": 8,
        "imported": 6,
        "skipped": 2,
        "errors": 0
      }

---

## Monitoring & Analytics API
-----------------------------

### Get Scheduler Statistics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /statistics

   Get overall scheduler statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **time_range** (optional): ``24h``, ``7d``, ``30d``, ``90d``

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "statistics": {
          "time_range": "24h",
          "total_jobs": 12,
          "active_jobs": 8,
          "total_executions": 1250,
          "successful_executions": 1235,
          "failed_executions": 15,
          "success_rate": 98.8
        }
      }

### Get Job Performance Metrics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}/metrics

   Get performance metrics for a specific job.

   **Path Parameters**:

   * **id** (string): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **granularity** (optional): ``hourly``, ``daily``, ``weekly``
   * **start_date** (optional): ISO date string
   * **end_date** (optional): ISO date string

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "job_id": "job_001",
        "job_name": "Modbus Poll",
        "metrics": {
          "execution_times": {
            "average_ms": 125,
            "p95_ms": 185
          },
          "success_rate": 99.4
        }
      }

---

## WebSocket Events
-------------------

For real-time job execution monitoring:

**Endpoint**: ``wss://univa-gateway/api/v1/scheduler/ws``

**Connection Headers**:

.. code-block:: http

   Authorization: Bearer <token>

**Events**:

.. code-block:: json

   {
     "event": "job_started",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "execution_id": "exec_20250312151030123"
     }
   }

.. code-block:: json

   {
     "event": "job_completed",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "status": "success"
     }
   }

.. code-block:: json

   {
     "event": "job_failed",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "error": "Connection timeout"
     }
   }

---

## Error Handling
-----------------

All error responses follow this format:

.. sourcecode:: http

   HTTP/1.1 400 Bad Request
   Content-Type: application/json
   
   {
     "success": false,
     "error": "Error Code",
     "message": "Human-readable error message",
     "code": "ERROR_CODE",
     "timestamp": "2025-03-12T15:10:30Z"
   }

### Error Codes

.. list-table:: Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **SCHEDULE_CONFLICT**
     - Job schedule conflicts with existing job
   * - **INVALID_CRON_EXPRESSION**
     - Invalid cron expression format
   * - **JOB_NOT_FOUND**
     - Specified job does not exist
   * - **MAX_JOBS_EXCEEDED**
     - Maximum number of jobs exceeded
   * - **RESOURCE_LIMIT_EXCEEDED**
     - Job would exceed system resource limits
   * - **ACTION_NOT_SUPPORTED**
     - Requested action type not supported
   * - **EXECUTION_TIMEOUT**
     - Job execution timed out
   * - **DEPENDENCY_FAILED**
     - Job dependency failed

---

## Rate Limiting
----------------

.. list-table:: Rate Limits
   :widths: 30 40 30
   :header-rows: 1

   * - Endpoint Type
     - Limit
     - Window
   * - **GET endpoints**
     - 100 requests
     - per minute
   * - **POST/PUT endpoints**
     - 20 requests
     - per minute
   * - **Job executions**
     - 50 concurrent
     - per gateway
   * - **WebSocket connections**
     - 5
     - per client

## Security Considerations
--------------------------

1. All job configurations are validated for security
2. File operations are sandboxed to prevent system access
3. System actions require elevated permissions
4. Cron expressions are validated for safety
5. All job executions are logged for audit purposes

## Performance Considerations
------------------------------

.. list-table:: Performance Limits
   :widths: 30 40 30
   :header-rows: 1

   * - Parameter
     - Default Limit
     - Configurable
   * - **Minimum interval**
     - 1 second
     - No
   * - **Maximum concurrent jobs**
     - 10
     - Yes
   * - **Maximum jobs per gateway**
     - 100
     - Yes
   * - **Memory per job**
     - 50MB
     - Yes
   * - **Execution timeout**
     - 300 seconds
     - Yes

## Backup & Recovery
--------------------

- Job configurations are included in system backups
- Failed jobs can be retried with exponential backoff
- Job state is persisted across reboots
- Import/export functionality for migration

## Versioning
-------------

API version is included in the URL path (``/api/v1/``). Breaking changes will result in a new version number.

## Examples
-----------

### Python - Create Interval Job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Create an interval job
   job_config = {
       "name": "Temperature Poll",
       "type": "interval",
       "enabled": True,
       "schedule": {
           "interval_seconds": 30
       },
       "action": {
           "type": "publish",
           "topic": "sensors/temperature",
           "payload": "{\"temp\": ${TEMP}}"
       },
       "tags": ["sensors", "polling"]
   }
   
   response = requests.post(
       "https://univa-gateway/api/v1/scheduler/jobs",
       json=job_config,
       headers={"Authorization": "Bearer your_token"}
   )
   
   if response.status_code == 201:
       result = response.json()
       print(f"Job created: {result['job_id']}")

### JavaScript - WebSocket Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Connect to WebSocket for real-time updates
   const token = "your_jwt_token";
   const ws = new WebSocket(`wss://univa-gateway/api/v1/scheduler/ws`);
   
   ws.onopen = () => {
       console.log("Connected to scheduler WebSocket");
       ws.send(JSON.stringify({
           "type": "auth",
           "token": token
       }));
   };
   
   ws.onmessage = (event) => {
       const data = JSON.parse(event.data);
       
       switch(data.event) {
           case "job_started":
               console.log(`Job ${data.data.job_name} started`);
               break;
           case "job_completed":
               console.log(`Job ${data.data.job_name} completed`);
               break;
           case "job_failed":
               console.error(`Job ${data.data.job_name} failed: ${data.data.error}`);
               break;
       }
   };

### Python - Bulk Job Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Export jobs to backup file
   export_response = requests.post(
       "https://univa-gateway/api/v1/scheduler/jobs/export",
       json={"format": "json", "include_statistics": True},
       headers={"Authorization": "Bearer your_token"}
   )
   
   # Save backup
   with open("jobs_backup.json", "w") as f:
       f.write(export_response.text)
   
   # Import jobs
   with open("jobs_backup.json", "rb") as f:
       import_response = requests.post(
           "https://univa-gateway/api/v1/scheduler/jobs/import",
           files={"configFile": ("jobs.json", f, "application/json")},
           data={"strategy": "replace", "dry_run": False},
           headers={"Authorization": "Bearer your_token"}
       )
   
   result = import_response.json()
   print(f"Imported {result['imported']} jobs, skipped {result['skipped']}")

## Support
----------

For API support, contact:

- **Email**: scheduler-support@univagateway.com
- **Documentation**: https://docs.univagateway.com/api/scheduler
- **Status**: https://status.univagateway.com/scheduler

## Release Notes
----------------

### v1.0.0 (2025-03-12)
- Initial release of Scheduler & Automation API
- Support for interval, daily, weekly, cron schedules
- Multiple action types (publish, log, output, system, file, rule)
- Real-time WebSocket monitoring
- Conflict detection and resource management
- Bulk import/export functionality
- Performance metrics and analytics
Scheduler & Automation API
==========================

Overview
--------

The Scheduler & Automation API provides comprehensive scheduling capabilities for the Univa Gateway platform. It enables automated job execution based on time, intervals, events, or cron expressions, allowing for autonomous gateway behavior without manual intervention.

Base URL
--------

.. code-block:: text

   https://univa-gateway/api/v1/scheduler

Authentication
--------------

All endpoints require JWT authentication via the ``Authorization`` header:

.. code-block:: http

   Authorization: Bearer <jwt_token>

Quick Reference
---------------

.. list-table:: API Endpoints Quick Reference
   :widths: 30 40 30
   :header-rows: 1

   * - Endpoint
     - Method
     - Description
   * - ``/jobs``
     - GET
     - List all jobs with filtering
   * - ``/jobs``
     - POST
     - Create new job
   * - ``/jobs/{id}``
     - GET
     - Get job details
   * - ``/jobs/{id}``
     - PUT
     - Update job
   * - ``/jobs/{id}``
     - DELETE
     - Delete job
   * - ``/jobs/{id}/toggle``
     - POST
     - Enable/disable job
   * - ``/jobs/{id}/run``
     - POST
     - Run job immediately
   * - ``/jobs/{id}/history``
     - GET
     - Get job execution history
   * - ``/jobs/{id}/logs``
     - GET
     - Get job execution logs
   * - ``/jobs/{id}/metrics``
     - GET
     - Get job performance metrics
   * - ``/executions/{id}``
     - GET
     - Get execution status
   * - ``/gateways``
     - GET
     - List available gateways
   * - ``/conflicts/check``
     - POST
     - Check for scheduling conflicts
   * - ``/resources/usage``
     - GET
     - Get system resource usage
   * - ``/jobs/run/all``
     - POST
     - Run all enabled jobs
   * - ``/jobs/disable/all``
     - POST
     - Disable all jobs
   * - ``/jobs/reset/all``
     - POST
     - Reset all jobs to defaults
   * - ``/jobs/export``
     - POST
     - Export job configurations
   * - ``/jobs/import``
     - POST
     - Import job configurations
   * - ``/statistics``
     - GET
     - Get scheduler statistics
   * - ``/tags``
     - GET
     - Get available tags
   * - ``/ws``
     - WebSocket
     - Real-time job events

Jobs Management API
-------------------

Get All Jobs
~~~~~~~~~~~~

.. http:get:: /jobs

   Retrieve list of all scheduled jobs with filtering options.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional, string): Filter by status (``active``, ``inactive``, ``error``)
   * **type** (optional, string): Filter by job type (``interval``, ``daily``, ``weekly``, ``monthly``, ``cron``, ``sunrise``)
   * **tags** (optional, string): Comma-separated list of tags
   * **enabled** (optional, boolean): Filter by enabled/disabled state
   * **gateway_id** (optional, string): Filter by gateway
   * **limit** (optional, integer): Number of jobs to return (default: 50, max: 100)
   * **offset** (optional, integer): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
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
                "topic": "device/data/modbus"
              },
              "tags": ["modbus", "polling", "telemetry"],
              "created_at": "2025-03-01T08:00:00Z",
              "last_execution": "2025-03-12T15:10:20Z",
              "execution_count": 1205,
              "average_duration_ms": 125
            }
          ]
        },
        "pagination": {
          "limit": 50,
          "offset": 0,
          "has_more": false
        }
      }

Create New Job
~~~~~~~~~~~~~~

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
        "gateway_id": "univa-gw-01",
        "schedule": {
          "time": "01:00",
          "timezone": "UTC"
        },
        "action": {
          "type": "system",
          "operation": "health_check"
        },
        "tags": ["health", "diagnostics", "maintenance"]
      }

   **Required Fields**:

   * ``name`` (string): Job name (1-100 characters)
   * ``type`` (string): Job type (interval, daily, weekly, monthly, cron, sunrise)
   * ``action.type`` (string): Action type (publish, log, output, system, file, rule)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "job_id": "job_003",
          "name": "Health Check",
          "message": "Job created successfully",
          "next_execution": "2025-03-13T01:00:00Z"
        }
      }

Get Job Details
~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}

   Retrieve detailed information about a specific job.

   **Path Parameters**:

   * **id** (string, required): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "job": {
            "id": "job_001",
            "name": "Modbus Poll",
            "description": "Poll Modbus devices for sensor data",
            "type": "interval",
            "enabled": true,
            "status": "active",
            "gateway_id": "univa-gw-01",
            "configuration": {
              "schedule": {
                "interval_seconds": 10
              },
              "action": {
                "type": "publish",
                "topic": "device/data/modbus",
                "payload_template": "{\"timestamp\": \"${NOW}\", \"device_id\": \"${DEVICE_ID}\"}"
              }
            },
            "statistics": {
              "total_executions": 1205,
              "successful_executions": 1198,
              "failed_executions": 7,
              "average_duration_ms": 125,
              "last_execution": "2025-03-12T15:10:20Z",
              "last_status": "success"
            },
            "metadata": {
              "created_at": "2025-03-01T08:00:00Z",
              "updated_at": "2025-03-10T14:30:00Z",
              "tags": ["modbus", "polling", "telemetry"]
            }
          }
        }
      }

Update Job
~~~~~~~~~~

.. http:put:: /jobs/{id}

   Update an existing job configuration.

   **Path Parameters**:

   * **id** (string, required): Job ID

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
          "payload_template": "{\"timestamp\": \"${NOW}\", \"device_id\": \"${DEVICE_ID}\", \"interval\": \"30s\"}"
        },
        "tags": ["modbus", "polling", "telemetry", "optimized"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "message": "Job updated successfully",
          "job_id": "job_001",
          "next_execution": "2025-03-12T15:11:00Z"
        }
      }

Delete Job
~~~~~~~~~~

.. http:delete:: /jobs/{id}

   Permanently delete a scheduled job.

   **Path Parameters**:

   * **id** (string, required): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "message": "Job deleted successfully",
          "deleted_job": "job_001",
          "statistics": {
            "total_executions": 1205
          }
        }
      }

Toggle Job Status
~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/{id}/toggle

   Enable or disable a job without deleting it.

   **Path Parameters**:

   * **id** (string, required): Job ID

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
        "enabled": false,
        "reason": "Temporarily disabled for maintenance"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "message": "Job disabled successfully",
          "job_id": "job_001",
          "enabled": false
        }
      }

Get Job Execution History
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}/history

   Retrieve history of job executions.

   **Path Parameters**:

   * **id** (string, required): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **start_date** (optional, string): ISO date string
   * **end_date** (optional, string): ISO date string
   * **status** (optional, string): Filter by status (``success``, ``failed``, ``timeout``)
   * **limit** (optional, integer): Number of records (default: 50)
   * **offset** (optional, integer): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
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
              "message": "Modbus data collected successfully"
            }
          ]
        },
        "pagination": {
          "limit": 50,
          "offset": 0,
          "has_more": true
        }
      }

Get Job Execution Logs
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}/logs

   Retrieve detailed execution logs for a job.

   **Path Parameters**:

   * **id** (string, required): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **execution_id** (optional, string): Specific execution ID
   * **level** (optional, string): Log level filter (info, warning, error, debug)
   * **limit** (optional, integer): Number of log entries (default: 100)
   * **offset** (optional, integer): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "job_id": "job_001",
          "job_name": "Modbus Poll",
          "logs": [
            {
              "timestamp": "2025-03-12T15:10:30.123Z",
              "level": "info",
              "message": "Starting Modbus poll",
              "execution_id": "exec_20250312151030123"
            },
            {
              "timestamp": "2025-03-12T15:10:30.456Z",
              "level": "info",
              "message": "Connected to Modbus device",
              "execution_id": "exec_20250312151030123"
            }
          ]
        }
      }

Job Execution API
-----------------

Run Job Immediately
~~~~~~~~~~~~~~~~~~~

.. http:post:: /jobs/{id}/run

   Manually trigger job execution.

   **Path Parameters**:

   * **id** (string, required): Job ID

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
          "debug": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "execution_id": "exec_20250312151030123",
          "job_id": "job_001",
          "status": "started",
          "trigger": "manual",
          "start_time": "2025-03-12T15:10:30Z"
        }
      }

Get Execution Status
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /executions/{id}

   Check status of a job execution.

   **Path Parameters**:

   * **id** (string, required): Execution ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
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
              "message": "Modbus data collected successfully"
            }
          }
        }
      }

Gateway Management API
----------------------

Get Available Gateways
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /gateways

   Retrieve list of available gateways.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional, string): Filter by gateway status
   * **online** (optional, boolean): Filter by online/offline state

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "gateways": [
            {
              "id": "univa-gw-01",
              "name": "Univa-GW-01",
              "status": "online",
              "location": "Building A",
              "last_seen": "2025-03-12T15:10:30Z",
              "jobs_count": 4,
              "active_jobs": 3
            },
            {
              "id": "univa-gw-02",
              "name": "Univa-GW-02",
              "status": "offline",
              "location": "Building B",
              "last_seen": "2025-03-12T10:30:00Z",
              "jobs_count": 2,
              "active_jobs": 1
            }
          ]
        }
      }

Get Gateway Jobs
~~~~~~~~~~~~~~~~

.. http:get:: /gateways/{id}/jobs

   Get all jobs for a specific gateway.

   **Path Parameters**:

   * **id** (string, required): Gateway ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "gateway_id": "univa-gw-01",
          "gateway_name": "Univa-GW-01",
          "total_jobs": 4,
          "active_jobs": 3,
          "jobs": [
            {
              "id": "job_001",
              "name": "Modbus Poll",
              "type": "interval",
              "enabled": true,
              "next_execution": "2025-03-12T15:10:30Z"
            }
          ]
        }
      }

Conflict Detection API
----------------------

Check for Conflicts
~~~~~~~~~~~~~~~~~~~

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
        "gateway_id": "univa-gw-01",
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
        "data": {
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
      }

Get System Resource Usage
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /resources/usage

   Check system resource utilization by jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional, string): Specific gateway ID

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "resource_usage": {
            "gateway_id": "univa-gw-01",
            "max_concurrent_jobs": 5,
            "current_concurrent_jobs": 3,
            "cpu_usage_percent": 28.5,
            "memory_usage_mb": 124,
            "storage_usage_mb": 45.2,
            "network_usage_kbps": 125.6
          }
        }
      }

Bulk Operations API
-------------------

Run All Jobs
~~~~~~~~~~~~

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
        "gateway_id": "univa-gw-01",
        "exclude_tags": ["maintenance"],
        "sequential": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "execution_group_id": "group_20250312151500",
          "gateway_id": "univa-gw-01",
          "total_jobs": 3,
          "started_jobs": 3,
          "status": "started"
        }
      }

Disable All Jobs
~~~~~~~~~~~~~~~~

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
        "gateway_id": "univa-gw-01",
        "reason": "System maintenance",
        "duration_minutes": 60
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "message": "All jobs disabled",
          "gateway_id": "univa-gw-01",
          "disabled_count": 12,
          "disabled_until": "2025-03-12T16:00:00Z"
        }
      }

Reset All Jobs
~~~~~~~~~~~~~~

.. http:post:: /jobs/reset/all

   Reset all jobs to their default configurations.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /jobs/reset/all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "univa-gw-01",
        "keep_data": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "message": "All jobs reset to defaults",
          "gateway_id": "univa-gw-01",
          "reset_count": 12,
          "kept_execution_history": true
        }
      }

Export Jobs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

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
        "gateway_id": "univa-gw-01",
        "format": "json",
        "include_history": false,
        "include_statistics": true
      }

   **Response**: 

   File download with Content-Type: ``application/json``

   **Response Headers**:

   .. code-block:: http

      Content-Type: application/json
      Content-Disposition: attachment; filename="jobs_export_20250312.json"

Import Jobs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

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
      
      --boundary
      Content-Disposition: form-data; name="gateway_id"
      
      univa-gw-01
      --boundary
      Content-Disposition: form-data; name="strategy"
      
      merge
      --boundary--

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "import_id": "import_20250312152000",
          "gateway_id": "univa-gw-01",
          "processed": 8,
          "imported": 6,
          "skipped": 2,
          "errors": 0
        }
      }

Monitoring & Analytics API
--------------------------

Get Scheduler Statistics
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /statistics

   Get overall scheduler statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional, string): Specific gateway ID
   * **time_range** (optional, string): ``24h``, ``7d``, ``30d``, ``90d``

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "statistics": {
            "gateway_id": "univa-gw-01",
            "time_range": "24h",
            "total_jobs": 12,
            "active_jobs": 8,
            "total_executions": 1250,
            "successful_executions": 1235,
            "failed_executions": 15,
            "success_rate": 98.8,
            "average_execution_time_ms": 142,
            "peak_concurrent_jobs": 3
          }
        }
      }

Get Job Performance Metrics
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /jobs/{id}/metrics

   Get performance metrics for a specific job.

   **Path Parameters**:

   * **id** (string, required): Job ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **granularity** (optional, string): ``hourly``, ``daily``, ``weekly``
   * **start_date** (optional, string): ISO date string
   * **end_date** (optional, string): ISO date string

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "job_id": "job_001",
          "job_name": "Modbus Poll",
          "metrics": {
            "execution_times": {
              "average_ms": 125,
              "min_ms": 98,
              "max_ms": 210,
              "p95_ms": 185
            },
            "success_rate": 99.4,
            "resource_usage": {
              "average_cpu_percent": 12.5,
              "average_memory_mb": 15.2
            }
          }
        }
      }

Tag Management API
------------------

Get All Tags
~~~~~~~~~~~~

.. http:get:: /tags

   Retrieve all tags used across jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional, string): Specific gateway ID
   * **limit** (optional, integer): Number of tags to return

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "tags": [
            {
              "name": "modbus",
              "job_count": 2,
              "last_used": "2025-03-12T15:10:30Z"
            },
            {
              "name": "health",
              "job_count": 1,
              "last_used": "2025-03-12T01:00:00Z"
            }
          ]
        }
      }

Get Jobs by Tag
~~~~~~~~~~~~~~~

.. http:get:: /tags/{tag}/jobs

   Get all jobs with a specific tag.

   **Path Parameters**:

   * **tag** (string, required): Tag name

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional, string): Specific gateway ID

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "data": {
          "tag": "modbus",
          "jobs": [
            {
              "id": "job_001",
              "name": "Modbus Poll",
              "type": "interval",
              "enabled": true,
              "next_execution": "2025-03-12T15:10:30Z"
            }
          ]
        }
      }

WebSocket Events API
--------------------

For real-time job execution monitoring:

**Endpoint**: ``wss://univa-gateway/api/v1/scheduler/ws``

**Connection Headers**:

.. code-block:: http

   Authorization: Bearer <token>

**Connection Parameters**:

.. code-block:: json

   {
     "type": "subscribe",
     "gateway_id": "univa-gw-01",
     "events": ["job_started", "job_completed", "job_failed"]
   }

**Events**:

Job Started Event
~~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "event": "job_started",
     "timestamp": "2025-03-12T15:10:30Z",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "execution_id": "exec_20250312151030123",
       "trigger": "scheduled"
     }
   }

Job Completed Event
~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "event": "job_completed",
     "timestamp": "2025-03-12T15:10:32Z",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "execution_id": "exec_20250312151030123",
       "status": "success",
       "duration_ms": 120
     }
   }

Job Failed Event
~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "event": "job_failed",
     "timestamp": "2025-03-12T15:10:35Z",
     "data": {
       "job_id": "job_002",
       "job_name": "Daily Backup",
       "execution_id": "exec_20250312151035234",
       "error": "Connection timeout",
       "retry_count": 2
     }
   }

Schedule Types Configuration
----------------------------

The following schedule types are supported:

Interval Schedule
~~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "interval",
     "interval_seconds": 60,
     "jitter_percentage": 10,
     "start_immediately": true
   }

Daily Schedule
~~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "daily",
     "time": "06:00",
     "timezone": "UTC",
     "skip_weekends": false
   }

Weekly Schedule
~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "weekly",
     "days_of_week": ["mon", "wed", "fri"],
     "time": "09:00",
     "timezone": "America/New_York"
   }

Monthly Schedule
~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "monthly",
     "day_of_month": 15,
     "time": "00:00",
     "timezone": "UTC"
   }

Cron Schedule
~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "cron",
     "cron_expression": "0 9-17 * * 1-5",
     "timezone": "America/New_York"
   }

Sunrise/Sunset Schedule
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "type": "sunrise",
     "mode": "sunset",
     "offset_minutes": -30,
     "latitude": 40.7128,
     "longitude": -74.0060
   }

Action Types Configuration
--------------------------

Publish Action
~~~~~~~~~~~~~~

Publish data to MQTT topic.

.. code-block:: json

   {
     "type": "publish",
     "topic": "device/data",
     "payload_template": "{\"value\": ${DATA}}",
     "qos": 1,
     "retain": false
   }

Log Action
~~~~~~~~~~

Write log messages.

.. code-block:: json

   {
     "type": "log",
     "message": "Job executed successfully",
     "level": "info"
   }

Output Action
~~~~~~~~~~~~~

Control GPIO or relays.

.. code-block:: json

   {
     "type": "output",
     "relay": "relay_1",
     "value": "on",
     "duration_ms": 5000
   }

System Action
~~~~~~~~~~~~~

Execute system operations.

.. code-block:: json

   {
     "type": "system",
     "operation": "health_check",
     "parameters": {
       "check_services": true
     }
   }

File Action
~~~~~~~~~~~

Perform file operations.

.. code-block:: json

   {
     "type": "file",
     "operation": "rotate",
     "pattern": "*.log",
     "retain_days": 7
   }

Rule Action
~~~~~~~~~~~

Trigger rule engine.

.. code-block:: json

   {
     "type": "rule",
     "rule_node": "alert_generator",
     "parameters": {
       "severity": "high"
     }
   }

Error Handling
--------------

All error responses follow this format:

.. sourcecode:: http

   HTTP/1.1 400 Bad Request
   Content-Type: application/json
   
   {
     "success": false,
     "error": "INVALID_REQUEST",
     "message": "Human-readable error message",
     "code": "ERROR_CODE",
     "timestamp": "2025-03-12T15:10:30Z"
   }

Common Error Codes
~~~~~~~~~~~~~~~~~~

.. list-table:: Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **JOB_NOT_FOUND**
     - Specified job does not exist
   * - **SCHEDULE_CONFLICT**
     - Job schedule conflicts with existing job
   * - **INVALID_CRON_EXPRESSION**
     - Invalid cron expression format
   * - **MAX_JOBS_EXCEEDED**
     - Maximum number of jobs exceeded
   * - **RESOURCE_LIMIT_EXCEEDED**
     - Job would exceed system resource limits
   * - **ACTION_NOT_SUPPORTED**
     - Requested action type not supported
   * - **GATEWAY_OFFLINE**
     - Target gateway is offline
   * - **EXECUTION_TIMEOUT**
     - Job execution timed out
   * - **VALIDATION_ERROR**
     - Request validation failed
   * - **UNAUTHORIZED**
     - Insufficient permissions

Rate Limiting
-------------

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

Performance Limits
------------------

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

Examples
--------

Python - Create Daily Job
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   headers = {
       "Authorization": "Bearer your_token",
       "Content-Type": "application/json"
   }
   
   job_config = {
       "name": "Temperature Poll",
       "type": "interval",
       "enabled": True,
       "gateway_id": "univa-gw-01",
       "schedule": {
           "interval_seconds": 30
       },
       "action": {
           "type": "publish",
           "topic": "sensors/temperature",
           "payload_template": "{\"temp\": ${TEMP}}"
       },
       "tags": ["sensors", "polling"]
   }
   
   response = requests.post(
       "https://univa-gateway/api/v1/scheduler/jobs",
       json=job_config,
       headers=headers
   )
   
   if response.status_code == 201:
       result = response.json()
       print(f"Job created: {result['data']['job_id']}")

JavaScript - WebSocket Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   const token = "your_jwt_token";
   const ws = new WebSocket(`wss://univa-gateway/api/v1/scheduler/ws`);
   
   ws.onopen = () => {
       console.log("Connected to scheduler WebSocket");
       ws.send(JSON.stringify({
           "type": "subscribe",
           "gateway_id": "univa-gw-01",
           "events": ["job_started", "job_completed", "job_failed"]
       }));
   };
   
   ws.onmessage = (event) => {
       const data = JSON.parse(event.data);
       
       switch(data.event) {
           case "job_started":
               console.log(`Job ${data.data.job_name} started`);
               break;
           case "job_completed":
               console.log(`Job ${data.data.job_name} completed: ${data.data.status}`);
               break;
           case "job_failed":
               console.error(`Job ${data.data.job_name} failed: ${data.data.error}`);
               break;
       }
   };

Python - Bulk Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Run all jobs on a gateway
   run_response = requests.post(
       "https://univa-gateway/api/v1/scheduler/jobs/run/all",
       json={"gateway_id": "univa-gw-01", "sequential": True},
       headers={"Authorization": "Bearer your_token"}
   )
   
   # Get statistics
   stats_response = requests.get(
       "https://univa-gateway/api/v1/scheduler/statistics",
       params={"gateway_id": "univa-gw-01", "time_range": "24h"},
       headers={"Authorization": "Bearer your_token"}
   )
   
   stats = stats_response.json()
   print(f"Success rate: {stats['data']['statistics']['success_rate']}%")

API Versioning
--------------

The API uses URL versioning. The current version is v1.

- Base URL: ``/api/v1/scheduler``
- Breaking changes will increment the version number
- Version is required in the URL path

Deprecation Policy
------------------

- Deprecated endpoints will be marked in documentation
- Deprecated endpoints will return ``X-API-Deprecated`` header
- Minimum 6 months notice before removal
- Alternative endpoints will be provided

Changelog
---------

v1.2.0 (2025-03-15)
~~~~~~~~~~~~~~~~~~~~

- Added gateway management endpoints
- Added job execution logs endpoint
- Added tag management API
- Added reset all jobs endpoint
- Enhanced conflict detection
- Improved WebSocket events

v1.1.0 (2025-02-28)
~~~~~~~~~~~~~~~~~~~~

- Added bulk operations API
- Added resource monitoring endpoints
- Enhanced error handling
- Added rate limiting headers

v1.0.0 (2025-01-15)
~~~~~~~~~~~~~~~~~~~~

- Initial release
- Basic job management
- Schedule types support
- Action types configuration
- WebSocket monitoring

Support
-------

- **Documentation**: https://docs.univagateway.com/api/scheduler
- **API Status**: https://status.univagateway.com/scheduler
- **Support Email**: scheduler-support@univagateway.com
- **Issue Tracker**: https://github.com/univagateway/scheduler-api/issues
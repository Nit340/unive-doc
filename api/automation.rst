Scheduler & Automation Management
=================================

This document describes the scheduler and automation management page and its related API endpoints for managing scheduled jobs, automation tasks, and timed operations.

Page Route (Frontend)
---------------------

.. http:get:: /scheduler-automation

   **Description**: Renders the complete scheduler and automation management page with all jobs, schedules, gateways, and monitoring data embedded in the HTML.

   **Headers**:

   .. code-block:: http

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Scheduler & Automation - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-scheduler='{...}' data-jobs='[...]' data-gateways='[...]' data-metrics='{...}'>
          <!-- Scheduler & Automation page with:
               SECTION 1: Job Management & Status Overview
               SECTION 2: Job Creation & Configuration
               SECTION 3: Gateway Management
               SECTION 4: Schedule Types & Templates
               SECTION 5: Conflict Detection & Resource Monitoring
               SECTION 6: Bulk Operations & Import/Export
               SECTION 7: Execution History & Analytics
               SECTION 8: Live Monitoring & WebSocket Events
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded scheduler and job data
   - All jobs, gateways, schedules, and monitoring data are embedded
   - JavaScript reads this data and renders the complete scheduler management interface
   - No separate API calls needed on initial page load
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all scheduler and automation operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Base Path: ``/api/v1/scheduler``

System Status (Overview)
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/status

   **Description**: Get overall scheduler system status and statistics.
   
   **UI Element**: Status overview dashboard
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "system_status": "healthy",
          "total_jobs": 45,
          "active_jobs": 38,
          "failed_jobs": 2,
          "success_rate": 96.7,
          "last_execution": "2025-03-15T10:25:30Z",
          "next_scheduled": "2025-03-15T10:30:00Z",
          "resource_usage": {
            "cpu": 24.5,
            "memory": "1.2 GB",
            "concurrent_jobs": 3
          }
        },
        "success": true,
        "message": "Scheduler status retrieved successfully"
      }

Job Management (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/jobs

   **Description**: Retrieve list of all scheduled jobs with filtering options.
   
   **UI Element**: SECTION 1 - Jobs table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **status** (string): Filter by status (active, inactive, error) *(optional)*
   * **type** (string): Filter by job type (interval, daily, weekly, monthly, cron, sunrise) *(optional)*
   * **tags** (string): Comma-separated list of tags *(optional)*
   * **enabled** (boolean): Filter by enabled/disabled state *(optional)*
   * **limit** (integer): Number of jobs to return (default: 50, max: 100) *(optional)*
   * **offset** (integer): Pagination offset *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        "success": true,
        "message": "Jobs retrieved successfully"
      }

.. http:post:: /api/v1/scheduler/jobs

   **Description**: Create a new scheduled job.
   
   **UI Element**: SECTION 1 - "Create New Job" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "job_003",
          "name": "Health Check",
          "next_execution": "2025-03-13T01:00:00Z"
        },
        "success": true,
        "message": "Job created successfully"
      }

.. http:get:: /api/v1/scheduler/jobs/{job_id}

   **Description**: Retrieve detailed information about a specific job.
   
   **UI Element**: SECTION 1 - Job details modal
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Job details retrieved successfully"
      }

.. http:put:: /api/v1/scheduler/jobs/{job_id}

   **Description**: Update an existing job configuration.
   
   **UI Element**: SECTION 1 - "Edit Job" button
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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
        "data": {
          "job_id": "job_001",
          "next_execution": "2025-03-12T15:11:00Z"
        },
        "success": true,
        "message": "Job updated successfully"
      }

.. http:delete:: /api/v1/scheduler/jobs/{job_id}

   **Description**: Permanently delete a scheduled job.
   
   **UI Element**: SECTION 1 - "Delete Job" button
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "deleted_job": "job_001",
          "statistics": {
            "total_executions": 1205
          }
        },
        "success": true,
        "message": "Job deleted successfully"
      }

.. http:post:: /api/v1/scheduler/jobs/{job_id}/toggle

   **Description**: Enable or disable a job without deleting it.
   
   **UI Element**: SECTION 1 - Job toggle switch
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enabled": false,
        "reason": "Temporarily disabled for maintenance"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "job_001",
          "enabled": false
        },
        "success": true,
        "message": "Job disabled successfully"
      }

Job Configuration (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/templates

   **Description**: Get job configuration templates and action types.
   
   **UI Element**: SECTION 2 - Job creation form templates
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "schedule_types": {
            "interval": {
              "description": "Execute at fixed time intervals",
              "parameters": ["interval_seconds", "jitter_percentage"]
            },
            "daily": {
              "description": "Execute once per day at specific time",
              "parameters": ["time", "timezone", "skip_weekends"]
            },
            "cron": {
              "description": "Execute based on cron expression",
              "parameters": ["cron_expression", "timezone"]
            }
          },
          "action_types": {
            "publish": {
              "description": "Publish data to MQTT topic",
              "parameters": ["topic", "payload_template", "qos"]
            },
            "system": {
              "description": "Execute system operations",
              "parameters": ["operation", "parameters"]
            },
            "file": {
              "description": "Perform file operations",
              "parameters": ["operation", "pattern", "retain_days"]
            }
          }
        },
        "success": true,
        "message": "Job templates retrieved successfully"
      }

.. http:get:: /api/v1/scheduler/tags

   **Description**: Retrieve all tags used across jobs.
   
   **UI Element**: SECTION 2 - Tag selection dropdown
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **limit** (integer): Number of tags to return *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Tags retrieved successfully"
      }

Gateway Management (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/gateways

   **Description**: Retrieve list of available gateways.
   
   **UI Element**: SECTION 3 - Gateway selection and management
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **status** (string): Filter by gateway status *(optional)*
   * **online** (boolean): Filter by online/offline state *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Gateways retrieved successfully"
      }

.. http:get:: /api/v1/scheduler/gateways/{gateway_id}/jobs

   **Description**: Get all jobs for a specific gateway.
   
   **UI Element**: SECTION 3 - Gateway job list
   
   **Path Parameters**:

   * **gateway_id** (string): Gateway identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Gateway jobs retrieved successfully"
      }

Schedule Types (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/scheduler/schedules/validate

   **Description**: Validate schedule configuration.
   
   **UI Element**: SECTION 4 - Schedule configuration validation
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "type": "cron",
        "cron_expression": "0 9-17 * * 1-5",
        "timezone": "America/New_York"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "valid": true,
          "next_executions": [
            "2025-03-16T09:00:00Z",
            "2025-03-16T10:00:00Z"
          ],
          "description": "Every hour from 9 AM to 5 PM on weekdays"
        },
        "success": true,
        "message": "Schedule validation completed"
      }

Conflict Detection (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/scheduler/conflicts/check

   **Description**: Detect scheduling conflicts between jobs.
   
   **UI Element**: SECTION 5 - Conflict detection analysis
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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
        },
        "success": true,
        "message": "Conflict check completed"
      }

.. http:get:: /api/v1/scheduler/resources/usage

   **Description**: Check system resource utilization by jobs.
   
   **UI Element**: SECTION 5 - Resource monitoring dashboard
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **gateway_id** (string): Specific gateway ID *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Resource usage retrieved successfully"
      }

Bulk Operations (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/scheduler/jobs/run/all

   **Description**: Manually trigger all enabled jobs.
   
   **UI Element**: SECTION 6 - "Run All Jobs" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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
        "data": {
          "execution_group_id": "group_20250312151500",
          "gateway_id": "univa-gw-01",
          "total_jobs": 3,
          "started_jobs": 3,
          "status": "started"
        },
        "success": true,
        "message": "All jobs execution started"
      }

.. http:post:: /api/v1/scheduler/jobs/disable/all

   **Description**: Disable all scheduled jobs.
   
   **UI Element**: SECTION 6 - "Disable All Jobs" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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
        "data": {
          "gateway_id": "univa-gw-01",
          "disabled_count": 12,
          "disabled_until": "2025-03-12T16:00:00Z"
        },
        "success": true,
        "message": "All jobs disabled successfully"
      }

.. http:post:: /api/v1/scheduler/jobs/reset/all

   **Description**: Reset all jobs to their default configurations.
   
   **UI Element**: SECTION 6 - "Reset All Jobs" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "gateway_id": "univa-gw-01",
        "keep_data": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "gateway_id": "univa-gw-01",
          "reset_count": 12,
          "kept_execution_history": true
        },
        "success": true,
        "message": "All jobs reset to defaults"
      }

.. http:post:: /api/v1/scheduler/jobs/export

   **Description**: Export all job configurations.
   
   **UI Element**: SECTION 6 - "Export Configurations" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "gateway_id": "univa-gw-01",
        "format": "json",
        "include_history": false,
        "include_statistics": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      Content-Disposition: attachment; filename="jobs_export_20250312.json"
      
      {
        "export_type": "jobs_configuration",
        "timestamp": "2025-03-15T10:30:00Z",
        "gateway_id": "univa-gw-01",
        "jobs": [...],
        "statistics": {...}
      }

.. http:post:: /api/v1/scheduler/jobs/import

   **Description**: Import jobs from configuration file.
   
   **UI Element**: SECTION 6 - "Import Configurations" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **configFile** (required): Configuration file
   * **gateway_id** (string): Target gateway ID
   * **strategy** (string): Import strategy (merge, replace) *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "import_id": "import_20250312152000",
          "gateway_id": "univa-gw-01",
          "processed": 8,
          "imported": 6,
          "skipped": 2,
          "errors": 0
        },
        "success": true,
        "message": "Jobs imported successfully"
      }

Execution History (SECTION 7)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/jobs/{job_id}/history

   **Description**: Retrieve history of job executions.
   
   **UI Element**: SECTION 7 - Job execution history table
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **start_date** (string): ISO date string *(optional)*
   * **end_date** (string): ISO date string *(optional)*
   * **status** (string): Filter by status (success, failed, timeout) *(optional)*
   * **limit** (integer): Number of records (default: 50) *(optional)*
   * **offset** (integer): Pagination offset *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        "success": true,
        "message": "Job execution history retrieved successfully"
      }

.. http:get:: /api/v1/scheduler/jobs/{job_id}/logs

   **Description**: Retrieve detailed execution logs for a job.
   
   **UI Element**: SECTION 7 - Job execution logs viewer
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **execution_id** (string): Specific execution ID *(optional)*
   * **level** (string): Log level filter (info, warning, error, debug) *(optional)*
   * **limit** (integer): Number of log entries (default: 100) *(optional)*
   * **offset** (integer): Pagination offset *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Job logs retrieved successfully"
      }

.. http:get:: /api/v1/scheduler/jobs/{job_id}/metrics

   **Description**: Get performance metrics for a specific job.
   
   **UI Element**: SECTION 7 - Job performance metrics
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **granularity** (string): hourly, daily, weekly *(optional)*
   * **start_date** (string): ISO date string *(optional)*
   * **end_date** (string): ISO date string *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Job metrics retrieved successfully"
      }

.. http:post:: /api/v1/scheduler/jobs/{job_id}/run

   **Description**: Manually trigger job execution.
   
   **UI Element**: SECTION 7 - "Run Job Now" button
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

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
        "data": {
          "execution_id": "exec_20250312151030123",
          "job_id": "job_001",
          "status": "started",
          "trigger": "manual",
          "start_time": "2025-03-12T15:10:30Z"
        },
        "success": true,
        "message": "Job execution started"
      }

.. http:get:: /api/v1/scheduler/executions/{execution_id}

   **Description**: Check status of a job execution.
   
   **UI Element**: SECTION 7 - Execution status modal
   
   **Path Parameters**:

   * **execution_id** (string): Execution identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Execution status retrieved successfully"
      }

.. http:get:: /api/v1/scheduler/statistics

   **Description**: Get overall scheduler statistics.
   
   **UI Element**: SECTION 7 - Statistics dashboard
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **gateway_id** (string): Specific gateway ID *(optional)*
   * **time_range** (string): 24h, 7d, 30d, 90d *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
        },
        "success": true,
        "message": "Scheduler statistics retrieved successfully"
      }

Live Monitoring (SECTION 8)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/scheduler/monitor/active

   **Description**: Get currently active job executions.
   
   **UI Element**: SECTION 8 - Active executions monitor
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "active_executions": [
            {
              "execution_id": "exec_20250312151030123",
              "job_id": "job_001",
              "job_name": "Modbus Poll",
              "start_time": "2025-03-12T15:10:30Z",
              "duration_ms": 15000,
              "progress": 65,
              "status": "running"
            }
          ],
          "total_active": 3,
          "peak_today": 5
        },
        "success": true,
        "message": "Active executions retrieved successfully"
      }

WebSocket Real-time Updates
---------------------------

Connection Endpoint
~~~~~~~~~~~~~~~~~~~
::

   ws://{gateway-ip}/api/v1/scheduler/ws
   wss://{gateway-ip}/api/v1/scheduler/ws (secure)

Authentication
~~~~~~~~~~~~~~
Connect with authentication token:

.. code-block:: json

   {
     "type": "auth",
     "token": "your_jwt_token"
   }

Message Types
~~~~~~~~~~~~~

Job Started Event
^^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "job_started",
     "timestamp": "2025-03-12T15:10:30Z",
     "data": {
       "job_id": "job_001",
       "job_name": "Modbus Poll",
       "execution_id": "exec_20250312151030123",
       "trigger": "scheduled"
     }
   }

Job Completed Event
^^^^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "job_completed",
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
^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "job_failed",
     "timestamp": "2025-03-12T15:10:35Z",
     "data": {
       "job_id": "job_002",
       "job_name": "Daily Backup",
       "execution_id": "exec_20250312151035234",
       "error": "Connection timeout",
       "retry_count": 2
     }
   }

Schedule Execution Event
^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "schedule_event",
     "timestamp": "2025-03-12T02:00:00Z",
     "data": {
       "schedule_id": "schedule_nightly",
       "job_count": 3,
       "status": "started"
     }
   }

Resource Alert Event
^^^^^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "resource_alert",
     "timestamp": "2025-03-12T15:15:00Z",
     "data": {
       "level": "warning",
       "resource": "cpu",
       "usage_percent": 85,
       "message": "High CPU usage detected"
     }
   }

Route Summary
-------------

.. list-table:: Scheduler & Automation Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/scheduler-automation`` - Main scheduler page
     - Yes
   * - Status
     - GET
     - ``/api/v1/scheduler/status`` - System status
     - Yes
   * - Jobs
     - GET
     - ``/api/v1/scheduler/jobs`` - List all jobs
     - Yes
   * - Jobs
     - POST
     - ``/api/v1/scheduler/jobs`` - Create job
     - Yes
   * - Jobs
     - GET
     - ``/api/v1/scheduler/jobs/{id}`` - Get job details
     - Yes
   * - Jobs
     - PUT
     - ``/api/v1/scheduler/jobs/{id}`` - Update job
     - Yes
   * - Jobs
     - DELETE
     - ``/api/v1/scheduler/jobs/{id}`` - Delete job
     - Yes
   * - Jobs
     - POST
     - ``/api/v1/scheduler/jobs/{id}/toggle`` - Toggle job
     - Yes
   * - Templates
     - GET
     - ``/api/v1/scheduler/templates`` - Get templates
     - Yes
   * - Tags
     - GET
     - ``/api/v1/scheduler/tags`` - Get tags
     - Yes
   * - Gateways
     - GET
     - ``/api/v1/scheduler/gateways`` - Get gateways
     - Yes
   * - Gateways
     - GET
     - ``/api/v1/scheduler/gateways/{id}/jobs`` - Get gateway jobs
     - Yes
   * - Schedules
     - POST
     - ``/api/v1/scheduler/schedules/validate`` - Validate schedule
     - Yes
   * - Conflicts
     - POST
     - ``/api/v1/scheduler/conflicts/check`` - Check conflicts
     - Yes
   * - Resources
     - GET
     - ``/api/v1/scheduler/resources/usage`` - Get resource usage
     - Yes
   * - Bulk Ops
     - POST
     - ``/api/v1/scheduler/jobs/run/all`` - Run all jobs
     - Yes
   * - Bulk Ops
     - POST
     - ``/api/v1/scheduler/jobs/disable/all`` - Disable all jobs
     - Yes
   * - Bulk Ops
     - POST
     - ``/api/v1/scheduler/jobs/reset/all`` - Reset all jobs
     - Yes
   * - Bulk Ops
     - POST
     - ``/api/v1/scheduler/jobs/export`` - Export jobs
     - Yes
   * - Bulk Ops
     - POST
     - ``/api/v1/scheduler/jobs/import`` - Import jobs
     - Yes
   * - History
     - GET
     - ``/api/v1/scheduler/jobs/{id}/history`` - Get job history
     - Yes
   * - History
     - GET
     - ``/api/v1/scheduler/jobs/{id}/logs`` - Get job logs
     - Yes
   * - History
     - GET
     - ``/api/v1/scheduler/jobs/{id}/metrics`` - Get job metrics
     - Yes
   * - History
     - POST
     - ``/api/v1/scheduler/jobs/{id}/run`` - Run job manually
     - Yes
   * - History
     - GET
     - ``/api/v1/scheduler/executions/{id}`` - Get execution status
     - Yes
   * - History
     - GET
     - ``/api/v1/scheduler/statistics`` - Get statistics
     - Yes
   * - Monitor
     - GET
     - ``/api/v1/scheduler/monitor/active`` - Get active executions
     - Yes
   * - Real-time
     - WS
     - ``/api/v1/scheduler/ws`` - WebSocket for real-time updates
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /scheduler-automation``
   - Server renders HTML with embedded scheduler and job data
   - All 8 sections populated with current data

2. **User views system status**:
   - Scheduler health and statistics
   - Active jobs and success rates
   - Resource usage monitoring

3. **User manages jobs** (SECTION 1):
   - View all jobs in table with filtering
   - "Create New Job" → ``POST /api/v1/scheduler/jobs``
   - "Edit Job" → ``PUT /api/v1/scheduler/jobs/{id}``
   - "Delete Job" → ``DELETE /api/v1/scheduler/jobs/{id}``
   - Toggle job status → ``POST /api/v1/scheduler/jobs/{id}/toggle``

4. **User configures jobs** (SECTION 2):
   - Select from job templates → ``GET /api/v1/scheduler/templates``
   - Manage tags → ``GET /api/v1/scheduler/tags``
   - Validate schedule → ``POST /api/v1/scheduler/schedules/validate``

5. **User manages gateways** (SECTION 3):
   - View available gateways → ``GET /api/v1/scheduler/gateways``
   - View gateway jobs → ``GET /api/v1/scheduler/gateways/{id}/jobs``

6. **User checks conflicts** (SECTION 5):
   - Detect scheduling conflicts → ``POST /api/v1/scheduler/conflicts/check``
   - Monitor resources → ``GET /api/v1/scheduler/resources/usage``

7. **User performs bulk operations** (SECTION 6):
   - "Run All Jobs" → ``POST /api/v1/scheduler/jobs/run/all``
   - "Disable All Jobs" → ``POST /api/v1/scheduler/jobs/disable/all``
   - "Reset All Jobs" → ``POST /api/v1/scheduler/jobs/reset/all``
   - "Export Configurations" → ``POST /api/v1/scheduler/jobs/export``
   - "Import Configurations" → ``POST /api/v1/scheduler/jobs/import``

8. **User reviews history** (SECTION 7):
   - View execution history → ``GET /api/v1/scheduler/jobs/{id}/history``
   - Check job logs → ``GET /api/v1/scheduler/jobs/{id}/logs``
   - View metrics → ``GET /api/v1/scheduler/jobs/{id}/metrics``
   - "Run Job Now" → ``POST /api/v1/scheduler/jobs/{id}/run``
   - Check execution status → ``GET /api/v1/scheduler/executions/{id}``
   - View statistics → ``GET /api/v1/scheduler/statistics``

9. **User monitors live** (SECTION 8):
   - Active executions → ``GET /api/v1/scheduler/monitor/active``
   - WebSocket connection → ``/api/v1/scheduler/ws``

Error Codes
-----------

.. list-table:: Scheduler Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - JOB_NOT_FOUND
     - Specified job does not exist
   * - SCHEDULE_CONFLICT
     - Job schedule conflicts with existing job
   * - INVALID_CRON_EXPRESSION
     - Invalid cron expression format
   * - MAX_JOBS_EXCEEDED
     - Maximum number of jobs exceeded
   * - RESOURCE_LIMIT_EXCEEDED
     - Job would exceed system resource limits
   * - ACTION_NOT_SUPPORTED
     - Requested action type not supported
   * - GATEWAY_OFFLINE
     - Target gateway is offline
   * - EXECUTION_TIMEOUT
     - Job execution timed out
   * - VALIDATION_ERROR
     - Request validation failed
   * - IMPORT_FAILED
     - Job import failed
   * - EXPORT_FAILED
     - Job export failed
   * - OPERATION_IN_PROGRESS
     - Another operation is in progress

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Authorization: Bearer <token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages scheduler for the current gateway specified in `X-Gateway-ID` header.

Support Information
-------------------

- **Scheduler Support**: scheduler-support@univa.com
- **Automation Help**: +1 (555) 789-0999
- **Support Hours**: 24/7 for critical automation failures
- **Documentation**: https://docs.univa.com/scheduler
- **Status Page**: https://status.univa.com/scheduler

---
*Document last updated: March 15, 2025*
*API Version: 1.0.0*
*Scheduler Module Version: 2.5.0*
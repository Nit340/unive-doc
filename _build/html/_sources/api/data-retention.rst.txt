Data Retention API
==================

APIs for managing storage policies, data lifecycle, and retention configuration.

Overview
--------

The Data Retention API manages storage policies, data lifecycle, and retention configuration for the Univa Gateway platform. This system ensures optimal storage usage through FIFO rotation, priority-based retention, and automated purge mechanisms while preserving critical operational data.

Base URL
--------

::

   https://univa-gateway/api/v1/retention

Authentication
--------------

All endpoints require JWT authentication via the ``Authorization`` header:

.. code-block:: http

   Authorization: Bearer <jwt_token>

Storage Management API
----------------------

Get Storage Status
~~~~~~~~~~~~~~~~~~

.. http:get:: /retention/storage/status

   Retrieve current storage usage and configuration.

   **Query Parameters**:
   
   * **gateway_id** (optional): Gateway ID to get storage status for specific gateway

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "storage": {
          "total_capacity_mb": 4096,
          "used_mb": 1380,
          "remaining_mb": 2716,
          "usage_percentage": 33.7,
          "status": "healthy",
          "thresholds": {
            "high_usage_warning": 85,
            "critical_purge_trigger": 95
          }
        },
        "fifo_buffer": {
          "enabled": true,
          "status": "active",
          "priority_flags": {
            "critical_safety_logs": "protected",
            "event_logs": "high_priority",
            "telemetry_data": "medium_priority"
          }
        },
        "last_check": "2025-03-12T14:30:00Z"
      }

Refresh Storage Status
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/storage/refresh

   Force immediate storage status refresh.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Storage status refreshed",
        "refresh_time": "2025-03-12T15:45:00Z"
      }

Update Storage Thresholds
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /retention/storage/thresholds

   Configure storage warning and purge thresholds.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /retention/storage/thresholds HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "high_usage_warning": 85,
        "critical_purge_trigger": 95,
        "enable_auto_purge": true,
        "purge_strategy": "fifo", // "fifo", "priority", "size_based"
        "exclude_critical_logs": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Storage thresholds updated successfully",
        "new_thresholds": {
          "high_usage_warning": 85,
          "critical_purge_trigger": 95,
          "auto_purge_enabled": true
        }
      }

Data Retention Policies API
---------------------------

Get All Policies
~~~~~~~~~~~~~~~~

.. http:get:: /retention/policies

   Retrieve all configured data retention policies.

   **Query Parameters**:
   
   * **gateway_id** (optional): Gateway ID to get policies for specific gateway
   * **include_size** (optional): Include current size data (boolean, default: true)

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "policies": [
          {
            "category_id": "telemetry",
            "category_name": "Telemetry Data",
            "enabled": true,
            "retention_days": 7,
            "max_size_mb": 500,
            "compression": "weekly",
            "purge_priority": "medium",
            "sync_before_delete": true,
            "estimated_current_size_mb": 320,
            "last_purge": "2025-03-10T02:00:00Z"
          },
          {
            "category_id": "events",
            "category_name": "Event Logs",
            "enabled": true,
            "retention_days": 30,
            "max_size_mb": 300,
            "compression": "daily",
            "purge_priority": "high",
            "sync_before_delete": true,
            "estimated_current_size_mb": 280,
            "last_purge": "2025-03-11T02:00:00Z"
          },
          {
            "category_id": "alerts",
            "category_name": "Alert Logs",
            "enabled": true,
            "retention_days": 60,
            "max_size_mb": 200,
            "compression": "none",
            "purge_priority": "medium",
            "sync_before_delete": true,
            "estimated_current_size_mb": 150,
            "last_purge": "2025-03-09T02:00:00Z"
          },
          {
            "category_id": "craneiq_safety",
            "category_name": "CraneIQ Safety Logs",
            "enabled": true,
            "retention_days": 90,
            "max_size_mb": 700,
            "compression": "daily",
            "purge_priority": "critical",
            "sync_before_delete": true,
            "protected": true,
            "estimated_current_size_mb": 450,
            "last_purge": "2025-03-05T02:00:00Z"
          },
          {
            "category_id": "ota_history",
            "category_name": "OTA Update History",
            "enabled": true,
            "retention_days": 120,
            "max_size_mb": 100,
            "compression": "none",
            "purge_priority": "medium",
            "sync_before_delete": false,
            "estimated_current_size_mb": 45,
            "last_purge": "2025-02-15T02:00:00Z"
          }
        ]
      }

Update Retention Policy
~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /retention/policies/{category_id}

   Update retention policy for a specific data category.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **category_id** (required): One of ``telemetry``, ``events``, ``alerts``, ``craneiq_safety``, ``ota_history``

   **Request**:

   .. sourcecode:: http

      PUT /retention/policies/telemetry HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "enabled": true,
        "retention_days": 30,
        "max_size_mb": 500,
        "compression": "weekly", // "none", "daily", "weekly"
        "purge_priority": "medium", // "critical", "high", "medium", "low"
        "sync_before_delete": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Retention policy updated for telemetry data",
        "policy": {
          "category_id": "telemetry",
          "retention_days": 30,
          "max_size_mb": 500,
          "compression": "weekly",
          "purge_priority": "medium",
          "sync_before_delete": true,
          "next_purge": "2025-04-12T02:00:00Z"
        }
      }

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 403 Forbidden
      Content-Type: application/json
      
      {
        "success": false,
        "error": "Policy protected",
        "message": "CraneIQ safety logs policy cannot be modified",
        "code": "PROTECTED_POLICY"
      }

Get Policy Status
~~~~~~~~~~~~~~~~~

.. http:get:: /retention/policies/status

   Get current status of all policies including size and last purge.

   **Query Parameters**:
   
   * **gateway_id** (optional): Gateway ID
   * **category_ids** (optional): Comma-separated category IDs

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "policies_status": [
          {
            "category_id": "telemetry",
            "enabled": true,
            "current_size_mb": 320,
            "last_purge": "2025-03-10T02:00:00Z",
            "next_purge": "2025-03-17T02:00:00Z",
            "records_count": 45000
          },
          {
            "category_id": "craneiq_safety",
            "enabled": true,
            "current_size_mb": 450,
            "last_purge": "2025-03-05T02:00:00Z",
            "next_purge": "2025-03-12T02:00:00Z",
            "records_count": 12000
          }
        ]
      }

Batch Update Policies
~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /retention/policies/batch

   Update multiple retention policies at once.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /retention/policies/batch HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "policies": [
          {
            "category_id": "telemetry",
            "retention_days": 14,
            "max_size_mb": 600
          },
          {
            "category_id": "events",
            "retention_days": 45,
            "purge_priority": "high"
          }
        ],
        "apply_immediately": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "updated_policies": 2,
        "failed_policies": 0,
        "message": "Batch update completed successfully",
        "summary": {
          "telemetry": "Updated",
          "events": "Updated"
        }
      }

Offline Buffering API
---------------------

Get Offline Buffer Status
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /retention/offline-buffer

   Retrieve offline buffering configuration and status.

   **Query Parameters**:
   
   * **gateway_id** (optional): Gateway ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "offline_buffering": {
          "enabled": true,
          "status": "active",
          "max_queue_size_mb": 100,
          "current_queue_size_mb": 0,
          "pending_records": 0,
          "retry_interval_seconds": 30,
          "max_retry_time_minutes": 20,
          "last_sync": "2025-03-12T14:25:00Z",
          "sync_status": "idle"
        },
        "buffer_health": "healthy",
        "estimated_buffer_duration": "0 minutes"
      }

Configure Offline Buffering
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /retention/offline-buffer/config

   Configure offline data buffering settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /retention/offline-buffer/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "enabled": true,
        "max_queue_size_mb": 100,
        "retry_interval_seconds": 30,
        "max_retry_time_minutes": 20,
        "auto_sync_on_reconnect": true,
        "data_categories": ["telemetry", "events", "alerts", "craneiq_safety"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Offline buffering configuration updated",
        "config": {
          "enabled": true,
          "max_queue_size_mb": 100,
          "estimated_capacity": "approximately 5000 records"
        }
      }

Force Buffer Sync
~~~~~~~~~~~~~~~~~

.. http:post:: /retention/offline-buffer/sync

   Manually trigger sync of buffered data.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/offline-buffer/sync HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "force": false, // Force sync even if offline
        "categories": ["all"] // or specific categories
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "sync_id": "sync_20250312_152045",
        "status": "in_progress",
        "records_to_sync": 245,
        "estimated_duration": "00:01:30",
        "progress_url": "/api/v1/retention/offline-buffer/sync/progress/sync_20250312_152045"
      }

Purge Operations API
--------------------

Purge Old Data
~~~~~~~~~~~~~~

.. http:post:: /retention/purge

   Manually purge data older than retention policy.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/purge HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "categories": ["all"], // or specific category IDs
        "older_than_days": null, // Optional: override retention policy
        "exclude_critical": true,
        "dry_run": false,
        "confirmation": {
          "user_id": "admin",
          "reason": "Manual cleanup before system maintenance"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "purge_id": "purge_20250312_160015",
        "status": "in_progress",
        "categories": ["telemetry", "events", "alerts"],
        "estimated_records": 12450,
        "estimated_freed_space_mb": 850,
        "excluded_categories": ["craneiq_safety"],
        "progress_url": "/api/v1/retention/purge/progress/purge_20250312_160015"
      }

Get Purge Progress
~~~~~~~~~~~~~~~~~~

.. http:get:: /retention/purge/progress/{purge_id}

   Check status of ongoing purge operation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **purge_id** (required): ID of the purge operation

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "purge_id": "purge_20250312_160015",
        "status": "completed",
        "progress": 100,
        "records_deleted": 12450,
        "freed_space_mb": 850,
        "duration_seconds": 45,
        "completion_time": "2025-03-12T16:01:00Z",
        "categories_purged": ["telemetry", "events", "alerts"]
      }

Clear Non-Critical Data
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/purge/non-critical

   Clear all non-critical data categories.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/purge/non-critical HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "confirmation": {
          "user_id": "admin",
          "timestamp": "2025-03-12T18:00:00Z"
        },
        "backup_before_clear": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "operation_id": "clear_non_critical_20250312_180000",
        "status": "started",
        "categories_to_clear": ["telemetry", "events", "alerts", "ota_history"],
        "categories_preserved": ["craneiq_safety"],
        "estimated_freed_space_mb": 795,
        "backup_created": true,
        "backup_id": "backup_pre_clear_20250312_180000"
      }

Clear All Logs
~~~~~~~~~~~~~~

.. http:post:: /retention/purge/all-logs

   Clear all log files.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/purge/all-logs HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "exclude_critical_logs": true,
        "confirmation": {
          "user_id": "admin"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "operation_id": "clear_all_logs_20250312_180000",
        "status": "started",
        "excluded_categories": ["craneiq_safety"],
        "estimated_freed_space_mb": 850
      }

Emergency Storage Cleanup
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/purge/emergency

   Perform emergency cleanup when storage is critical.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/purge/emergency HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "target_percentage": 75, // Target usage after cleanup
        "preserve_critical": true,
        "preserve_last_n_days": 7,
        "confirmation": {
          "emergency_code": "STORAGE_CRITICAL_95",
          "user_id": "admin"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "emergency_purge_id": "emergency_purge_20250312_163000",
        "status": "started",
        "current_usage_percentage": 95,
        "target_usage_percentage": 75,
        "estimated_freed_space_mb": 820,
        "priority_order": ["ota_history", "telemetry", "events", "alerts"],
        "preserved_data": ["craneiq_safety", "last_7_days_all_categories"]
      }

Analytics & Reporting API
-------------------------

Get Storage Analytics
~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /retention/analytics

   Retrieve detailed storage analytics and forecasts.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional): Gateway ID
   * **time_range** (optional): Time period ("7d", "30d", "90d", default: "30d")
   * **include_forecast** (optional): Include forecast data (boolean, default: true)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "analytics": {
          "time_range": "30d",
          "storage_trend": {
            "start_usage_mb": 1250,
            "end_usage_mb": 1380,
            "daily_growth_mb": 4.33,
            "growth_percentage": 10.4
          },
          "category_breakdown": {
            "telemetry": { "size_mb": 320, "percentage": 23.2 },
            "craneiq_safety": { "size_mb": 450, "percentage": 32.6 },
            "events": { "size_mb": 280, "percentage": 20.3 },
            "alerts": { "size_mb": 150, "percentage": 10.9 },
            "ota_history": { "size_mb": 45, "percentage": 3.3 },
            "other": { "size_mb": 135, "percentage": 9.8 }
          },
          "forecast": {
            "days_until_full": 45,
            "estimated_full_date": "2025-04-26",
            "recommended_action": "Increase telemetry compression to weekly"
          }
        },
        "last_updated": "2025-03-12T14:45:00Z"
      }

Generate Retention Report
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/reports/generate

   Generate detailed retention policy report.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/reports/generate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "report_type": "comprehensive", // "summary", "comprehensive", "compliance"
        "format": "pdf", // "json", "pdf", "csv"
        "include_recommendations": true,
        "time_range": "90d"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "report_id": "report_20250312_170000",
        "status": "generating",
        "estimated_completion": "2025-03-12T17:02:00Z",
        "download_url": "/api/v1/retention/reports/download/report_20250312_170000",
        "size_estimate_mb": 2.5
      }

Export & Diagnostics API
------------------------

Export Stored Data
~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/export

   Export stored data for analysis or backup.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/export HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "categories": ["telemetry", "events", "alerts"],
        "date_from": "2025-02-01",
        "date_to": "2025-03-12",
        "format": "json", // "json", "csv", "parquet"
        "compression": "gzip", // "none", "gzip", "zip"
        "include_metadata": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "export_20250312_171500",
        "status": "processing",
        "estimated_size_mb": 750,
        "estimated_records": 50000,
        "download_url": "/api/v1/retention/export/download/export_20250312_171500",
        "estimated_completion": "2025-03-12T17:30:00Z"
      }

Export Logs
~~~~~~~~~~~

.. http:post:: /retention/export/logs

   Export log files only.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/export/logs HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "log_types": ["system", "application", "security"],
        "date_from": "2025-02-01",
        "date_to": "2025-03-12",
        "format": "text",
        "compression": "gzip"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "export_logs_20250312_171500",
        "status": "processing",
        "estimated_size_mb": 150,
        "download_url": "/api/v1/retention/export/logs/download/export_logs_20250312_171500"
      }

Run Storage Diagnostics
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/diagnostics

   Run comprehensive storage diagnostics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/diagnostics HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "checks": ["integrity", "performance", "configuration"],
        "verbose": true,
        "fix_issues": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "diagnostic_id": "diag_20250312_172000",
        "status": "running",
        "checks": [
          {
            "name": "storage_integrity",
            "status": "in_progress",
            "progress": 45
          },
          {
            "name": "policy_validation",
            "status": "pending",
            "progress": 0
          }
        ],
        "results_url": "/api/v1/retention/diagnostics/results/diag_20250312_172000"
      }

System Actions API
------------------

Full Storage Reset
~~~~~~~~~~~~~~~~~~

.. http:post:: /retention/system/reset

   Complete storage reset (requires admin approval).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /retention/system/reset HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-01",
        "confirmation": {
          "user_id": "admin",
          "password": "encrypted_admin_password",
          "emergency_code": "FULL_RESET_AUTHORIZED"
        },
        "backup_before_reset": true,
        "preserve_configuration": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "reset_id": "full_reset_20250312_181500",
        "status": "authorized",
        "warning": "This will erase ALL data and restart the system",
        "estimated_duration": "00:05:00",
        "backup_created": true,
        "backup_id": "backup_pre_reset_20250312_181500",
        "restart_required": true,
        "restart_scheduled": "2025-03-12T18:20:00Z"
      }

Compliance & Audit API
----------------------

Get Retention Compliance
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /retention/compliance

   Check compliance with retention policies.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional): Gateway ID
   * **check_categories** (optional): Comma-separated category IDs

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "compliance_check": {
          "timestamp": "2025-03-12T18:30:00Z",
          "overall_compliance": 95.7,
          "checks": [
            {
              "category": "telemetry",
              "status": "compliant",
              "retention_days": 7,
              "oldest_record_days": 6,
              "size_within_limit": true
            },
            {
              "category": "craneiq_safety",
              "status": "compliant",
              "retention_days": 90,
              "oldest_record_days": 87,
              "protected": true,
              "sync_status": "up_to_date"
            }
          ],
          "violations": 0,
          "warnings": 1,
          "recommendations": [
            "Consider increasing telemetry compression to reduce storage growth"
          ]
        }
      }

Audit Log
~~~~~~~~~

.. http:get:: /retention/audit

   Get audit log of retention operations.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **gateway_id** (optional): Gateway ID
   * **start_date** (optional): ISO date string
   * **end_date** (optional): ISO date string
   * **operation_type** (optional): "purge", "policy_change", "export", "reset"
   * **limit** (optional): Number of records (default: 100)
   * **offset** (optional): Pagination offset

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "audit_log": [
          {
            "timestamp": "2025-03-12T02:00:00Z",
            "operation": "scheduled_purge",
            "user": "system",
            "categories": ["telemetry", "events"],
            "records_deleted": 3245,
            "freed_space_mb": 220,
            "status": "completed"
          },
          {
            "timestamp": "2025-03-11T14:30:00Z",
            "operation": "policy_update",
            "user": "admin",
            "changes": {
              "telemetry.retention_days": "7 → 14",
              "telemetry.max_size_mb": "500 → 600"
            },
            "status": "applied"
          }
        ],
        "total_records": 245,
        "pagination": {
          "limit": 100,
          "offset": 0,
          "has_more": true
        }
      }

WebSocket Events
----------------

Connection Endpoint
~~~~~~~~~~~~~~~~~~~

::

   wss://univa-gateway/api/v1/retention/ws

Connection Status
~~~~~~~~~~~~~~~~~

.. http:get:: /retention/ws/status

   Get WebSocket connection status.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "connected": true,
        "last_message": "2025-03-12T15:30:00Z",
        "active_clients": 3,
        "connection_id": "ws_conn_abc123"
      }

Message Types
~~~~~~~~~~~~~

**Storage Threshold Warning**:

.. code-block:: json

   {
     "event": "storage_threshold_warning",
     "gateway_id": "Univa-GW-01",
     "data": {
       "current_usage_percentage": 87,
       "threshold": 85,
       "recommended_action": "purge_old_data",
       "estimated_time_until_full": "3 days"
     }
   }

**Purge Progress**:

.. code-block:: json

   {
     "event": "purge_progress",
     "gateway_id": "Univa-GW-01",
     "data": {
       "purge_id": "purge_20250312_160015",
       "progress": 65,
       "records_deleted": 8092,
       "freed_space_mb": 552,
       "current_category": "telemetry"
     }
   }

**Policy Update Notification**:

.. code-block:: json

   {
     "event": "policy_updated",
     "gateway_id": "Univa-GW-01",
     "data": {
       "category_id": "telemetry",
       "changes": {
         "retention_days": 14,
         "max_size_mb": 600
       },
       "user": "admin",
       "timestamp": "2025-03-12T15:30:00Z"
     }
   }

**Storage Critical Alert**:

.. code-block:: json

   {
     "event": "storage_critical",
     "gateway_id": "Univa-GW-01",
     "data": {
       "current_usage_percentage": 95,
       "threshold": 95,
       "emergency_action": "emergency_purge_required",
       "available_space_mb": 204,
       "time_until_full": "2 hours"
     }
   }

**Real-time Storage Update**:

.. code-block:: json

   {
     "event": "storage_update",
     "gateway_id": "Univa-GW-01",
     "data": {
       "used_mb": 1395,
       "remaining_mb": 2701,
       "usage_percentage": 34.1,
       "timestamp": "2025-03-12T15:45:00Z"
     }
   }

Error Handling
--------------

Error Response Format
~~~~~~~~~~~~~~~~~~~~~

.. sourcecode:: http

   HTTP/1.1 400 Bad Request
   Content-Type: application/json
   
   {
     "success": false,
     "error": "Error Code",
     "message": "Human-readable error message",
     "code": "ERROR_CODE",
     "gateway_id": "Univa-GW-01",
     "timestamp": "2025-03-12T16:00:00Z",
     "request_id": "req_abc123def456"
   }

Common Error Codes
~~~~~~~~~~~~~~~~~~

.. list-table:: Data Retention Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **STORAGE_CRITICAL**
     - Storage usage above critical threshold
   * - **PROTECTED_CATEGORY**
     - Attempt to modify protected data category
   * - **INSUFFICIENT_STORAGE**
     - Not enough space for operation
   * - **RETENTION_VIOLATION**
     - Policy would violate compliance requirements
   * - **BACKUP_REQUIRED**
     - Backup required before destructive operation
   * - **ADMIN_APPROVAL_REQUIRED**
     - Operation requires administrator approval
   * - **INVALID_POLICY**
     - Invalid retention policy configuration
   * - **SYNC_IN_PROGRESS**
     - Data sync already in progress
   * - **EXPORT_TOO_LARGE**
     - Export exceeds maximum allowed size
   * - **RATE_LIMIT_EXCEEDED**
     - API rate limit exceeded
   * - **GATEWAY_NOT_FOUND**
     - Specified gateway ID not found
   * - **CATEGORY_NOT_FOUND**
     - Data category not found

Examples
--------

Python - Storage Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Get storage status
   headers = {"Authorization": "Bearer your_token"}
   
   status_response = requests.get(
       "https://univa-gateway/api/v1/retention/storage/status?gateway_id=Univa-GW-01",
       headers=headers
   )
   
   status = status_response.json()
   print(f"Storage Usage: {status['storage']['usage_percentage']}%")
   print(f"Remaining: {status['storage']['remaining_mb']}MB")
   
   # Update storage thresholds
   threshold_request = {
       "gateway_id": "Univa-GW-01",
       "high_usage_warning": 80,
       "critical_purge_trigger": 90,
       "enable_auto_purge": True,
       "purge_strategy": "priority",
       "exclude_critical_logs": True
   }
   
   threshold_response = requests.put(
       "https://univa-gateway/api/v1/retention/storage/thresholds",
       json=threshold_request,
       headers=headers
   )
   
   print(f"Thresholds updated: {threshold_response.json()['message']}")
   
   # Get retention policies
   policies_response = requests.get(
       "https://univa-gateway/api/v1/retention/policies?gateway_id=Univa-GW-01",
       headers=headers
   )
   
   policies = policies_response.json()
   for policy in policies['policies']:
       print(f"{policy['category_name']}: {policy['retention_days']} days")

Python - Data Purge Operation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Manual purge operation
   import requests
   import time
   
   purge_request = {
       "gateway_id": "Univa-GW-01",
       "categories": ["telemetry", "events"],
       "older_than_days": 30,
       "exclude_critical": True,
       "dry_run": False,
       "confirmation": {
           "user_id": "admin",
           "reason": "Monthly data cleanup"
       }
   }
   
   purge_response = requests.post(
       "https://univa-gateway/api/v1/retention/purge",
       json=purge_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   purge_result = purge_response.json()
   purge_id = purge_result['purge_id']
   print(f"Purge started: {purge_id}")
   
   # Monitor purge progress
   while True:
       progress_response = requests.get(
           f"https://univa-gateway/api/v1/retention/purge/progress/{purge_id}",
           headers={"Authorization": "Bearer your_token"}
       )
       
       progress = progress_response.json()
       print(f"Purge progress: {progress.get('progress', 0)}%")
       
       if progress['status'] == 'completed':
           print(f"Purge completed: {progress['records_deleted']} records deleted")
           print(f"Freed space: {progress['freed_space_mb']}MB")
           break
       elif progress['status'] == 'failed':
           print(f"Purge failed: {progress.get('error', 'Unknown error')}")
           break
       
       time.sleep(5)

Python - Export Data
~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Export data for analysis
   import requests
   
   export_request = {
       "gateway_id": "Univa-GW-01",
       "categories": ["telemetry", "events"],
       "date_from": "2025-02-01",
       "date_to": "2025-03-12",
       "format": "json",
       "compression": "gzip",
       "include_metadata": True
   }
   
   export_response = requests.post(
       "https://univa-gateway/api/v1/retention/export",
       json=export_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   export_result = export_response.json()
   export_id = export_result['export_id']
   print(f"Export started: {export_id}")
   
   # Wait for completion and download
   import time
   while True:
       # In production, you would poll the status endpoint
       time.sleep(30)
       
       # Check if export is ready
       # Download from export_result['download_url']

JavaScript - Real-time Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket connection for real-time retention monitoring
   const ws = new WebSocket('wss://univa-gateway/api/v1/retention/ws');
   
   ws.onopen = function() {
       console.log('Connected to Data Retention WebSocket');
       // Send gateway identification
       ws.send(JSON.stringify({
           action: "subscribe",
           gateway_id: "Univa-GW-01"
       }));
   };
   
   ws.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       switch(message.event) {
           case 'storage_threshold_warning':
               if (message.gateway_id === "Univa-GW-01") {
                   showStorageWarning(
                       message.data.current_usage_percentage,
                       message.data.threshold,
                       message.data.recommended_action
                   );
               }
               break;
               
           case 'storage_update':
               if (message.gateway_id === "Univa-GW-01") {
                   updateStorageDisplay(
                       message.data.used_mb,
                       message.data.remaining_mb,
                       message.data.usage_percentage
                   );
               }
               break;
               
           case 'purge_progress':
               if (message.gateway_id === "Univa-GW-01") {
                   updatePurgeProgress(
                       message.data.purge_id,
                       message.data.progress,
                       message.data.current_category
                   );
                   updatePurgeStats(
                       message.data.records_deleted,
                       message.data.freed_space_mb
                   );
               }
               break;
               
           case 'policy_updated':
               if (message.gateway_id === "Univa-GW-01") {
                   showPolicyUpdateNotification(
                       message.data.category_id,
                       message.data.changes,
                       message.data.user
                   );
               }
               break;
               
           case 'storage_critical':
               if (message.gateway_id === "Univa-GW-01") {
                   showCriticalAlert(
                       message.data.current_usage_percentage,
                       message.data.emergency_action
                   );
                   triggerEmergencyActions();
               }
               break;
       }
   };
   
   ws.onerror = function(error) {
       console.error('WebSocket error:', error);
   };
   
   ws.onclose = function() {
       console.log('WebSocket connection closed');
   };
   
   // UI update functions
   function updateStorageDisplay(used, remaining, percentage) {
       document.getElementById('storage-used').textContent = used + ' MB';
       document.getElementById('storage-remaining').textContent = remaining + ' MB';
       document.getElementById('storage-percentage').textContent = percentage + '%';
       document.getElementById('storage-progress').style.width = percentage + '%';
   }

JavaScript - Retention Dashboard Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Load storage analytics
   async function loadStorageAnalytics() {
       const token = localStorage.getItem('token');
       const gatewayId = document.getElementById('gateway-select').value;
       
       try {
           const response = await fetch(`/api/v1/retention/analytics?gateway_id=${gatewayId}&time_range=30d`, {
               headers: {
                   'Authorization': `Bearer ${token}`
               }
           });
           
           const analytics = await response.json();
           renderStorageChart(analytics.analytics.category_breakdown);
           updateForecastInfo(analytics.analytics.forecast);
           updateStorageTrends(analytics.analytics.storage_trend);
       } catch (error) {
           console.error('Error loading analytics:', error);
       }
   }
   
   // Update retention policy
   async function updateRetentionPolicy(categoryId, policyData) {
       const token = localStorage.getItem('token');
       const gatewayId = document.getElementById('gateway-select').value;
       
       try {
           const response = await fetch(`/api/v1/retention/policies/${categoryId}`, {
               method: 'PUT',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify({
                   gateway_id: gatewayId,
                   ...policyData
               })
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification(`Policy updated for ${categoryId}`);
               return result.policy;
           } else {
               throw new Error(result.message || 'Policy update failed');
           }
       } catch (error) {
           console.error('Error updating policy:', error);
           showError('Failed to update policy');
       }
   }
   
   // Run manual purge
   async function runManualPurge(categories, options) {
       const token = localStorage.getItem('token');
       const gatewayId = document.getElementById('gateway-select').value;
       
       const purgeRequest = {
           gateway_id: gatewayId,
           categories: categories,
           older_than_days: options.olderThanDays || null,
           exclude_critical: options.excludeCritical || true,
           dry_run: options.dryRun || false,
           confirmation: {
               user_id: 'admin',
               reason: options.reason || 'Manual cleanup'
           }
       };
       
       try {
           const response = await fetch('/api/v1/retention/purge', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(purgeRequest)
           });
           
           const result = await response.json();
           
           if (result.success) {
               startPurgeMonitoring(result.purge_id);
               return result;
           } else {
               throw new Error(result.message || 'Purge failed to start');
           }
       } catch (error) {
           console.error('Error starting purge:', error);
           showError('Failed to start purge operation');
       }
   }
   
   // Get WebSocket connection status
   async function getWebSocketStatus() {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch('/api/v1/retention/ws/status', {
               headers: {
                   'Authorization': `Bearer ${token}`
               }
           });
           
           const status = await response.json();
           updateConnectionStatus(status.connected, status.last_message);
       } catch (error) {
           console.error('Error getting WebSocket status:', error);
       }
   }

Best Practices
--------------

Storage Management
~~~~~~~~~~~~~~~~~~

1. **Monitor Regularly**
   - Check storage usage daily
   - Set up threshold alerts
   - Review growth trends weekly

2. **Configure Thresholds**
   - Warning at 80-85% usage
   - Critical action at 90-95% usage
   - Always preserve critical safety data

3. **Optimize Policies**
   - Balance retention vs storage
   - Use compression where appropriate
   - Prioritize data based on importance

Retention Strategy
~~~~~~~~~~~~~~~~~~

1. **Category-based Retention**
   - Critical safety data: 90+ days
   - Event logs: 30-60 days
   - Telemetry data: 7-30 days
   - OTA history: 90-180 days

2. **Compression Strategy**
   - Critical data: daily compression
   - Telemetry: weekly compression
   - Historical data: aggressive compression

3. **Purge Strategy**
   - Use FIFO for non-critical data
   - Priority-based for mixed importance
   - Always exclude protected categories

Emergency Procedures
~~~~~~~~~~~~~~~~~~~~

1. **Critical Storage**
   - Trigger emergency purge at 95%
   - Preserve last 7 days of all data
   - Never delete critical safety logs

2. **Backup Before Operations**
   - Always backup before bulk deletions
   - Verify backup integrity
   - Maintain backup retention

3. **Audit Trail**
   - Log all retention operations
   - Track user actions
   - Generate compliance reports

Security Considerations
-----------------------

Authentication & Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Role-based Access**
   - Admin: Full access
   - Operator: Read-only + limited purge
   - Viewer: Read-only access

2. **Critical Data Protection**
   - CraneIQ safety logs are protected
   - Requires admin approval for changes
   - Cannot be deleted via standard purge

3. **Audit Requirements**
   - All operations logged
   - Compliance reports
   - Regular security reviews

Data Protection
~~~~~~~~~~~~~~~

1. **Encryption**
   - Data at rest encryption
   - Secure transmission
   - Encrypted backups

2. **Integrity Checks**
   - Verify data before purge
   - Checksum validation
   - Regular integrity scans

3. **Compliance**
   - Follow data retention laws
   - Maintain audit trails
   - Regular compliance checks

Audit & Compliance
~~~~~~~~~~~~~~~~~~

1. **Logging Requirements**
   - Retention operation logs: 180 days
   - Policy change logs: 365 days
   - Security audit logs: 730 days

2. **Reporting**
   - Monthly compliance reports
   - Quarterly audit reviews
   - Annual compliance certification

3. **Documentation**
   - Retention policy documentation
   - Emergency procedure guides
   - Compliance documentation

System Requirements
-------------------

Minimum Requirements
~~~~~~~~~~~~~~~~~~~~

.. list-table:: System Requirements
   :widths: 40 60
   :header-rows: 1

   * - Component
     - Requirement
   * - **Storage**
     - 4GB minimum, 8GB recommended
   * - **Memory**
     - 2GB RAM minimum
   * - **CPU**
     - Multi-core for compression operations
   * - **Network**
     - Stable for cloud sync operations
   * - **Database**
     - SQLite or PostgreSQL support

Performance Considerations
--------------------------

1. **Purge Operations**
   - Small purges: 1-5 minutes
   - Large purges: 10-30 minutes
   - Emergency purges: 5-15 minutes

2. **Compression Impact**
   - CPU: 10-50% during compression
   - Memory: 500MB-1GB
   - Storage savings: 40-70%

3. **Export Operations**
   - Export speed: 10-100 MB/s
   - Compression ratio: 2:1 to 5:1
   - Network impact: Medium to High

Support & Troubleshooting
-------------------------

Getting Help
~~~~~~~~~~~~

1. **Documentation**
   - API reference
   - User guides
   - Troubleshooting guides

2. **Support Channels**
   - Email: retention-support@univagateway.com
   - Phone: +1-800-RETENTION
   - Online portal

3. **Debug Information**
   - Generate diagnostic reports
   - Share operation logs
   - Provide error codes

Common Solutions
~~~~~~~~~~~~~~~~

1. **Storage Full Errors**
   - Run manual purge
   - Increase storage thresholds
   - Review retention policies

2. **Purge Failures**
   - Check protected categories
   - Verify backup requirements
   - Check permissions

3. **Sync Issues**
   - Check network connectivity
   - Verify cloud credentials
   - Review buffer configuration

Notes
-----

.. warning::

   **Critical Safety Data**: CraneIQ safety logs are protected and cannot be modified without admin approval

.. warning::

   **Data Loss**: Purge operations permanently delete data. Always backup before bulk operations.

.. important::

   **Compliance**: Retention policies must comply with local regulations and safety standards.

.. note::

   **Monitoring**: Set up alerts for storage thresholds and policy violations.

.. note::

   **Testing**: Test retention policies in staging before production deployment.

.. note::

   **Documentation**: Maintain detailed records of all retention policy changes.

Rate Limiting
-------------

.. list-table:: Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Endpoint Type
     - Requests/Minute
     - Notes
   * - **GET endpoints**
     - 100
     - Status, policies, analytics
   * - **POST/PUT endpoints**
     - 20
     - Updates, purges, exports
   * - **Purge operations**
     - 1 concurrent
     - Only one purge at a time
   * - **Export operations**
     - 2GB per hour
     - Size limit for exports
   * - **WebSocket connections**
     - 10 per gateway
     - Automatic reconnection

Compliance Notes
----------------

1. **Safety Regulations**
   - CraneIQ safety logs have special protection
   - Retention periods based on safety requirements
   - Cannot be overridden without authorization

2. **Data Retention Laws**
   - Follow local data retention regulations
   - Maintain required audit trails
   - Document compliance procedures

3. **Audit Requirements**
   - All operations must be logged
   - Regular compliance reporting
   - Third-party audit support

Versioning
----------

API version is included in the URL path (``/api/v1/``). Breaking changes will result in a new version number.

Support
-------

For API support, contact:

- **Email**: retention-support@univagateway.com
- **Documentation**: https://docs.univagateway.com/api/retention
- **Emergency Contact**: +1-800-RETENTION
- **API Status**: https://status.univagateway.com

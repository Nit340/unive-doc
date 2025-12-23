Data Retention Management API
==============================

Page Route (Frontend)
---------------------

.. http:get:: /retention-management

   **Description**: Renders the complete data retention management page with all storage policies, data lifecycle configurations, and retention settings embedded in the HTML.

   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9

   **Response**::

      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Data Retention Management - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-storage-status='{...}' data-policies='[...]' data-analytics='{...}' data-audit='[...]'>
          <!-- Data retention page with:
               SECTION 1: Storage Status Dashboard
               SECTION 2: Retention Policies Configuration
               SECTION 3: Offline Buffering Settings
               SECTION 4: Purge Operations
               SECTION 5: Analytics & Reports
               SECTION 6: Compliance & Audit
               FOOTER: Export, Diagnostics, System Actions
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded storage and policy data
   - All retention configurations, policies, and analytics are embedded
   - JavaScript reads this data and renders the complete retention interface
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**::

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all data retention operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Storage Status Dashboard (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/retention-management/status

   **Description**: Get current storage status and dashboard data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "total_capacity_mb": 4096,
        "used_mb": 1380,
        "remaining_mb": 2716,
        "usage_percentage": 33.7,
        "status": "healthy",
        "thresholds": {
          "high_usage_warning": 85,
          "critical_purge_trigger": 95
        }
      }

.. http:put:: /api/retention-management/thresholds

   **Description**: Update storage warning and purge thresholds.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/retention-management/thresholds HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "high_usage_warning": 85,
        "critical_purge_trigger": 95,
        "enable_auto_purge": true,
        "purge_strategy": "fifo",
        "exclude_critical_logs": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Storage thresholds updated successfully",
        "new_thresholds": {
          "high_usage_warning": 85,
          "critical_purge_trigger": 95,
          "auto_purge_enabled": true
        }
      }

.. http:post:: /api/retention-management/refresh-status

   **Description**: Force immediate storage status refresh.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "refreshed": true,
        "refresh_time": "2024-03-20T15:45:00Z"
      }

Retention Policies Configuration (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/retention-management/policies

   **Description**: Get all configured data retention policies.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "policies": [
          {
            "category_id": "telemetry",
            "category_name": "Telemetry Data",
            "enabled": true,
            "retention_days": 7,
            "max_size_mb": 500,
            "compression": "weekly",
            "purge_priority": "medium",
            "estimated_current_size_mb": 320,
            "last_purge": "2024-03-10T02:00:00Z"
          },
          {
            "category_id": "craneiq_safety",
            "category_name": "CraneIQ Safety Logs",
            "enabled": true,
            "retention_days": 90,
            "max_size_mb": 700,
            "compression": "daily",
            "purge_priority": "critical",
            "protected": true,
            "estimated_current_size_mb": 450,
            "last_purge": "2024-03-05T02:00:00Z"
          }
        ]
      }

.. http:put:: /api/retention-management/policies/{category_id}

   **Description**: Update retention policy for a specific data category.
   
   **Path Parameters**:
   
   * **category_id** (string): Category identifier (telemetry, events, alerts, craneiq_safety, ota_history)
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/retention-management/policies/telemetry HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "enabled": true,
        "retention_days": 30,
        "max_size_mb": 500,
        "compression": "weekly",
        "purge_priority": "medium"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "category_id": "telemetry",
        "message": "Retention policy updated for telemetry data",
        "next_purge": "2024-04-12T02:00:00Z"
      }

.. http:get:: /api/retention-management/policies/status

   **Description**: Get current status of all policies including size and last purge.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "policies_status": [
          {
            "category_id": "telemetry",
            "enabled": true,
            "current_size_mb": 320,
            "last_purge": "2024-03-10T02:00:00Z",
            "next_purge": "2024-03-17T02:00:00Z",
            "records_count": 45000
          },
          {
            "category_id": "craneiq_safety",
            "enabled": true,
            "current_size_mb": 450,
            "last_purge": "2024-03-05T02:00:00Z",
            "next_purge": "2024-03-12T02:00:00Z",
            "records_count": 12000
          }
        ]
      }

.. http:put:: /api/retention-management/policies/batch

   **Description**: Update multiple retention policies at once.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/retention-management/policies/batch HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
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
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "updated_policies": 2,
        "message": "Batch update completed successfully",
        "summary": {
          "telemetry": "Updated",
          "events": "Updated"
        }
      }

Offline Buffering Settings (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/retention-management/offline-buffer

   **Description**: Get offline buffering configuration and status.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "status": "active",
        "max_queue_size_mb": 100,
        "current_queue_size_mb": 0,
        "pending_records": 0,
        "retry_interval_seconds": 30,
        "last_sync": "2024-03-20T14:25:00Z",
        "sync_status": "idle"
      }

.. http:put:: /api/retention-management/offline-buffer/config

   **Description**: Configure offline data buffering settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/retention-management/offline-buffer/config HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "enabled": true,
        "max_queue_size_mb": 100,
        "retry_interval_seconds": 30,
        "max_retry_time_minutes": 20,
        "auto_sync_on_reconnect": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "configured": true,
        "message": "Offline buffering configuration updated",
        "estimated_capacity": "approximately 5000 records"
      }

.. http:post:: /api/retention-management/offline-buffer/sync

   **Description**: Manually trigger sync of buffered data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/offline-buffer/sync HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "force": false,
        "categories": ["all"]
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "synced": true,
        "sync_id": "sync_20240320_152045",
        "status": "in_progress",
        "records_to_sync": 245,
        "estimated_duration": "00:01:30"
      }

Purge Operations (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/retention-management/purge

   **Description**: Manually purge data older than retention policy.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/purge HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "categories": ["telemetry", "events", "alerts"],
        "older_than_days": null,
        "exclude_critical": true,
        "dry_run": false,
        "confirmation": {
          "user_id": "admin",
          "reason": "Manual cleanup"
        }
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "purged": true,
        "purge_id": "purge_20240320_160015",
        "status": "in_progress",
        "categories": ["telemetry", "events", "alerts"],
        "estimated_records": 12450,
        "estimated_freed_space_mb": 850,
        "excluded_categories": ["craneiq_safety"]
      }

.. http:get:: /api/retention-management/purge/progress/{purge_id}

   **Description**: Check status of ongoing purge operation.
   
   **Path Parameters**:
   
   * **purge_id** (string): Purge operation identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "purge_id": "purge_20240320_160015",
        "status": "completed",
        "progress": 100,
        "records_deleted": 12450,
        "freed_space_mb": 850,
        "duration_seconds": 45,
        "categories_purged": ["telemetry", "events", "alerts"]
      }

.. http:post:: /api/retention-management/purge/non-critical

   **Description**: Clear all non-critical data categories.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/purge/non-critical HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "confirmation": {
          "user_id": "admin",
          "timestamp": "2024-03-20T18:00:00Z"
        },
        "backup_before_clear": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cleared": true,
        "operation_id": "clear_non_critical_20240320_180000",
        "status": "started",
        "categories_to_clear": ["telemetry", "events", "alerts", "ota_history"],
        "categories_preserved": ["craneiq_safety"],
        "estimated_freed_space_mb": 795
      }

.. http:post:: /api/retention-management/purge/emergency

   **Description**: Perform emergency cleanup when storage is critical.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/purge/emergency HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "target_percentage": 75,
        "preserve_critical": true,
        "preserve_last_n_days": 7,
        "confirmation": {
          "emergency_code": "STORAGE_CRITICAL_95",
          "user_id": "admin"
        }
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "emergency_purged": true,
        "emergency_purge_id": "emergency_purge_20240320_163000",
        "status": "started",
        "current_usage_percentage": 95,
        "target_usage_percentage": 75,
        "estimated_freed_space_mb": 820,
        "priority_order": ["ota_history", "telemetry", "events", "alerts"]
      }

Analytics & Reports (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/retention-management/analytics

   **Description**: Get detailed storage analytics and forecasts.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Query Parameters**:
   
   * **time_range** (optional): Time period (7d, 30d, 90d, default: 30d)
   * **include_forecast** (optional): Include forecast data (default: true)
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
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
            "estimated_full_date": "2024-04-26",
            "recommended_action": "Increase telemetry compression to weekly"
          }
        }
      }

.. http:post:: /api/retention-management/reports/generate

   **Description**: Generate detailed retention policy report.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/reports/generate HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "report_type": "comprehensive",
        "format": "pdf",
        "include_recommendations": true,
        "time_range": "90d"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "generated": true,
        "report_id": "report_20240320_170000",
        "status": "generating",
        "download_url": "/api/retention-management/reports/download/report_20240320_170000",
        "size_estimate_mb": 2.5
      }

Compliance & Audit (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/retention-management/compliance

   **Description**: Check compliance with retention policies.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "compliance_check": {
          "timestamp": "2024-03-20T18:30:00Z",
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

.. http:get:: /api/retention-management/audit

   **Description**: Get audit log of retention operations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Query Parameters**:
   
   * **start_date** (optional): Start date (YYYY-MM-DD)
   * **end_date** (optional): End date (YYYY-MM-DD)
   * **operation_type** (optional): Operation type filter
   * **limit** (optional): Records limit (default: 100)
   * **offset** (optional): Pagination offset
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "audit_log": [
          {
            "timestamp": "2024-03-20T02:00:00Z",
            "operation": "scheduled_purge",
            "user": "system",
            "categories": ["telemetry", "events"],
            "records_deleted": 3245,
            "freed_space_mb": 220,
            "status": "completed"
          },
          {
            "timestamp": "2024-03-19T14:30:00Z",
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
        "has_more": true
      }

Footer Actions
~~~~~~~~~~~~~~

.. http:post:: /api/retention-management/export

   **Description**: Export stored data for analysis or backup.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/export HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "categories": ["telemetry", "events", "alerts"],
        "date_from": "2024-02-01",
        "date_to": "2024-03-20",
        "format": "json",
        "compression": "gzip",
        "include_metadata": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "exported": true,
        "export_id": "export_20240320_171500",
        "status": "processing",
        "estimated_size_mb": 750,
        "download_url": "/api/retention-management/export/download/export_20240320_171500"
      }

.. http:post:: /api/retention-management/diagnostics

   **Description**: Run comprehensive storage diagnostics.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/diagnostics HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "checks": ["integrity", "performance", "configuration"],
        "verbose": true,
        "fix_issues": false
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "diagnostics_run": true,
        "diagnostic_id": "diag_20240320_172000",
        "status": "running",
        "results_url": "/api/retention-management/diagnostics/results/diag_20240320_172000"
      }

.. http:post:: /api/retention-management/system/reset

   **Description**: Complete storage reset (requires admin approval).
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/retention-management/system/reset HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "confirmation": {
          "user_id": "admin",
          "password": "encrypted_admin_password",
          "emergency_code": "FULL_RESET_AUTHORIZED"
        },
        "backup_before_reset": true,
        "preserve_configuration": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "reset_id": "full_reset_20240320_181500",
        "status": "authorized",
        "warning": "This will erase ALL data and restart the system",
        "backup_created": true,
        "restart_required": true
      }

Route Summary
-------------

.. list-table:: Data Retention Management Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/retention-management`` - Main retention page
     - Yes
   * - Status
     - GET
     - ``/api/retention-management/status`` - Storage status
     - Yes
   * - Status
     - PUT
     - ``/api/retention-management/thresholds`` - Update thresholds
     - Yes
   * - Status
     - POST
     - ``/api/retention-management/refresh-status`` - Refresh status
     - Yes
   * - Policies
     - GET
     - ``/api/retention-management/policies`` - Get all policies
     - Yes
   * - Policies
     - PUT
     - ``/api/retention-management/policies/{id}`` - Update policy
     - Yes
   * - Policies
     - GET
     - ``/api/retention-management/policies/status`` - Policy status
     - Yes
   * - Policies
     - PUT
     - ``/api/retention-management/policies/batch`` - Batch update
     - Yes
   * - Offline
     - GET
     - ``/api/retention-management/offline-buffer`` - Buffer status
     - Yes
   * - Offline
     - PUT
     - ``/api/retention-management/offline-buffer/config`` - Configure buffer
     - Yes
   * - Offline
     - POST
     - ``/api/retention-management/offline-buffer/sync`` - Manual sync
     - Yes
   * - Purge
     - POST
     - ``/api/retention-management/purge`` - Manual purge
     - Yes
   * - Purge
     - GET
     - ``/api/retention-management/purge/progress/{id}`` - Purge progress
     - Yes
   * - Purge
     - POST
     - ``/api/retention-management/purge/non-critical`` - Clear non-critical
     - Yes
   * - Purge
     - POST
     - ``/api/retention-management/purge/emergency`` - Emergency purge
     - Yes
   * - Analytics
     - GET
     - ``/api/retention-management/analytics`` - Storage analytics
     - Yes
   * - Analytics
     - POST
     - ``/api/retention-management/reports/generate`` - Generate report
     - Yes
   * - Compliance
     - GET
     - ``/api/retention-management/compliance`` - Compliance check
     - Yes
   * - Compliance
     - GET
     - ``/api/retention-management/audit`` - Audit log
     - Yes
   * - Footer
     - POST
     - ``/api/retention-management/export`` - Export data
     - Yes
   * - Footer
     - POST
     - ``/api/retention-management/diagnostics`` - Run diagnostics
     - Yes
   * - Footer
     - POST
     - ``/api/retention-management/system/reset`` - Full reset
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /retention-management``
   - Server renders HTML with embedded retention data
   - All 6 sections populated with current configuration

2. **User views storage status** (SECTION 1):
   - Storage usage charts and thresholds
   - System health indicators
   - "Update Thresholds" → ``PUT /api/retention-management/thresholds``
   - "Refresh Status" → ``POST /api/retention-management/refresh-status``

3. **User configures retention policies** (SECTION 2):
   - Policy table with category settings
   - "Update Policy" → ``PUT /api/retention-management/policies/{id}``
   - "Batch Update" → ``PUT /api/retention-management/policies/batch``
   - "View Status" → ``GET /api/retention-management/policies/status``

4. **User manages offline buffering** (SECTION 3):
   - Buffer configuration and status
   - "Configure Buffer" → ``PUT /api/retention-management/offline-buffer/config``
   - "Sync Now" → ``POST /api/retention-management/offline-buffer/sync``
   - Real-time queue monitoring

5. **User performs purge operations** (SECTION 4):
   - Manual purge form with options
   - "Purge Data" → ``POST /api/retention-management/purge``
   - "Check Progress" → ``GET /api/retention-management/purge/progress/{id}``
   - "Clear Non-Critical" → ``POST /api/retention-management/purge/non-critical``
   - "Emergency Cleanup" → ``POST /api/retention-management/purge/emergency``

6. **User views analytics** (SECTION 5):
   - Storage trends and forecasts
   - Category breakdown charts
   - "View Analytics" → ``GET /api/retention-management/analytics``
   - "Generate Report" → ``POST /api/retention-management/reports/generate``

7. **User checks compliance** (SECTION 6):
   - Compliance status dashboard
   - Audit log viewer with filters
   - "Check Compliance" → ``GET /api/retention-management/compliance``
   - "View Audit Log" → ``GET /api/retention-management/audit``

8. **User uses footer actions**:
   - "Export Data" → ``POST /api/retention-management/export``
   - "Run Diagnostics" → ``POST /api/retention-management/diagnostics``
   - "Full Reset" → ``POST /api/retention-management/system/reset``

Error Codes
-----------

.. list-table:: Data Retention Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - STORAGE_CRITICAL
     - Storage usage above critical threshold
   * - PROTECTED_CATEGORY
     - Attempt to modify protected data category
   * - INSUFFICIENT_STORAGE
     - Not enough space for operation
   * - RETENTION_VIOLATION
     - Policy would violate compliance requirements
   * - BACKUP_REQUIRED
     - Backup required before destructive operation
   * - ADMIN_APPROVAL_REQUIRED
     - Operation requires administrator approval
   * - INVALID_POLICY
     - Invalid retention policy configuration
   * - SYNC_IN_PROGRESS
     - Data sync already in progress
   * - EXPORT_TOO_LARGE
     - Export exceeds maximum allowed size
   * - PURGE_ACTIVE
     - Purge operation already in progress
   * - BUFFER_FULL
     - Offline buffer is full
   * - COMPLIANCE_FAILED
     - Operation would violate compliance

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Cookie: session_token=<token>
   X-Gateway-ID: GW-3920A9

**Important**: This API manages data retention for the current gateway specified in `X-Gateway-ID` header. Critical safety data has special protection.

Data Categories
---------------

### Supported Categories

.. list-table:: Data Categories
   :widths: 30 40 30
   :header-rows: 1

   * - Category ID
     - Description
     - Protection Level
   * - **telemetry**
     - Telemetry data from sensors
     - Medium
   * - **events**
     - System and application events
     - High
   * - **alerts**
     - Alert and notification logs
     - Medium
   * - **craneiq_safety**
     - CraneIQ safety critical logs
     - Critical (Protected)
   * - **ota_history**
     - OTA update history and logs
     - Medium
   * - **system_logs**
     - System operation logs
     - Low
   * - **debug_logs**
     - Debug and trace logs
     - Low

### Protection Levels

1. **Critical**: CraneIQ safety logs - Cannot be deleted, special protection
2. **High**: Event logs - Extended retention, sync before delete
3. **Medium**: Telemetry, alerts - Standard retention, auto-purge
4. **Low**: System logs, debug - Short retention, first to purge

Retention Periods
-----------------

### Default Retention Periods

.. list-table:: Default Retention
   :widths: 40 30 30
   :header-rows: 1

   * - Category
     - Default Days
     - Maximum Days
   * - CraneIQ Safety Logs
     - 90
     - 365
   * - Event Logs
     - 30
     - 180
   * - Alert Logs
     - 60
     - 365
   * - Telemetry Data
     - 7
     - 30
   * - OTA History
     - 120
     - 365
   * - System Logs
     - 14
     - 30
   * - Debug Logs
     - 7
     - 14

### Compression Settings

1. **None**: No compression (for frequently accessed data)
2. **Daily**: Daily compression (for event logs)
3. **Weekly**: Weekly compression (for telemetry data)
4. **Monthly**: Monthly compression (for historical data)

Storage Management
------------------

### Threshold Configuration

.. list-table:: Storage Thresholds
   :widths: 40 30 30
   :header-rows: 1

   * - Threshold Type
     - Default Value
     - Action
   * - High Usage Warning
     - 85%
     - Alert notification
   * - Critical Purge Trigger
     - 95%
     - Auto-purge non-critical data
   * - Emergency Action
     - 98%
     - Emergency cleanup required
   * - Minimum Free Space
     - 100MB
     - Prevent system failure

### Purge Strategies

1. **FIFO (First-In, First-Out)**: Delete oldest data first
2. **Priority-based**: Delete based on category priority
3. **Size-based**: Delete largest files first
4. **Hybrid**: Combination of multiple strategies

Offline Buffering
-----------------

### Buffer Configuration

- **Maximum Queue Size**: 100MB (configurable)
- **Retry Interval**: 30 seconds (configurable)
- **Maximum Retry Time**: 20 minutes (configurable)
- **Auto-sync**: Enabled on network reconnect

### Buffer Health Status

1. **Healthy**: Queue size < 50%
2. **Warning**: Queue size 50-80%
3. **Critical**: Queue size > 80%
4. **Full**: Queue size 100%

Rate Limiting
-------------

.. list-table:: Retention Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Operation Type
     - Requests/Minute
     - Notes
   * - **Status queries**
     - 60
     - GET endpoints
   * - **Policy updates**
     - 10
     - PUT endpoints
   * - **Purge operations**
     - 2
     - Only one concurrent purge
   * - **Export operations**
     - 5
     - Size limit: 2GB per export
   * - **Diagnostic runs**
     - 1
     - One concurrent diagnostic
   * - **System operations**
     - 2
     - Reset, emergency actions

Troubleshooting
---------------

### Common Issues

1. **Storage Full Errors**
   - Check retention policies
   - Run manual purge
   - Increase storage thresholds
   - Review data growth patterns

2. **Purge Operation Failed**
   - Check protected categories
   - Verify sufficient permissions
   - Check system resources
   - Review error logs

3. **Sync Issues**
   - Check network connectivity
   - Verify buffer configuration
   - Check destination availability
   - Review sync logs

4. **Compliance Violations**
   - Review retention policies
   - Check audit logs
   - Verify protection settings
   - Run compliance check

### Debug Information

When contacting support, provide:
- Gateway ID and firmware version
- Storage status and thresholds
- Retention policy configuration
- Error messages and codes
- Audit log entries
- Diagnostic report

Deprecation Notes
-----------------

No endpoints are currently deprecated. All endpoints are fully supported in the current version.

Glossary
--------

.. glossary::

   Retention Policy
      Rules defining how long data is kept before deletion.

   Purge Priority
      Order in which data is deleted during cleanup operations.

   Offline Buffering
      Temporary storage of data when destination is unavailable.

   Compression
      Reducing data size to save storage space.

   FIFO Rotation
      First-In, First-Out data deletion strategy.

   Protected Category
      Data category with special retention protections.

   Compliance Check
      Verification that policies meet regulatory requirements.

   Audit Log
      Record of all retention-related operations.

   Storage Threshold
      Usage level triggering specific actions or alerts.

   Emergency Purge
      Forced cleanup when storage is critically full.


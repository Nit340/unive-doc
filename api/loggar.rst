Device-Level Logging API
========================

.. contents::
   :depth: 3
   :local:

Page Route (Frontend)
---------------------

.. http:get:: /logging-management

   **Description**: Renders the complete device-level logging management page with all logging configurations, categories, destinations, and system status embedded in the HTML.

   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9

   **Response**::

      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Device-Level Logging - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-global-status='{...}' data-categories='[...]' data-destinations='{...}' data-rotation='{...}'>
          <!-- Logging management page with:
               SECTION 1: Global Status Dashboard
               SECTION 2: Log Categories Configuration
               SECTION 3: Output Destinations
               SECTION 4: Log Rotation Settings
               SECTION 5: Gateway Management
               SECTION 6: Operations & Testing
               FOOTER: Save All, Reset, System Actions
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded logging configuration data
   - All logging settings, categories, destinations, and rotation settings are embedded
   - JavaScript reads this data and renders the complete logging interface
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**::

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all logging operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Global Status Dashboard (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/logging-management/status

   **Description**: Get global logging system status and dashboard data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "global_log_level": "INFO",
        "storage": {
          "total_mb": 300,
          "used_mb": 156,
          "available_mb": 144,
          "usage_percentage": 52
        },
        "files": {
          "active": 8,
          "compressed": 12,
          "total_lines_today": 42156
        },
        "errors_24h": {
          "critical": 0,
          "error": 12,
          "warning": 47
        }
      }

.. http:put:: /api/logging-management/global-settings

   **Description**: Update global logging settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/logging-management/global-settings HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "enabled": true,
        "global_log_level": "INFO",
        "max_storage_mb": 300,
        "rotation_mode": "fifo-compress"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Global logging settings updated",
        "requires_restart": false
      }

.. http:get:: /api/logging-management/storage-analytics

   **Description**: Get detailed storage analytics and trends.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "daily_usage": [
          {"date": "2024-03-18", "size_mb": 145},
          {"date": "2024-03-19", "size_mb": 152},
          {"date": "2024-03-20", "size_mb": 156}
        ],
        "category_breakdown": [
          {"category": "system", "size_mb": 45},
          {"category": "network", "size_mb": 32},
          {"category": "modbus", "size_mb": 28}
        ],
        "predictions": {
          "full_in_days": 21,
          "recommended_action": "Increase storage limit"
        }
      }

Log Categories Configuration (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/logging-management/categories

   **Description**: Get all log categories with their configurations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "categories": [
          {
            "id": "system",
            "name": "System Logs",
            "enabled": true,
            "log_level": "INFO",
            "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
            "interval_ms": 1000,
            "max_file_size_mb": 10,
            "retention_days": 30,
            "lines_today": 12500
          },
          {
            "id": "network",
            "name": "Network Logs",
            "enabled": true,
            "log_level": "DEBUG",
            "log_format": "TIMESTAMP INTERFACE PROTOCOL",
            "interval_ms": 5000,
            "max_file_size_mb": 15,
            "retention_days": 14,
            "lines_today": 8500
          }
        ],
        "total_categories": 12,
        "enabled_count": 8
      }

.. http:get:: /api/logging-management/categories/{category_id}/config

   **Description**: Get detailed configuration for a specific category.
   
   **Path Parameters**:
   
   * **category_id** (string): Category identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "category": {
          "id": "system",
          "name": "System Logs",
          "description": "Kernel, watchdog, crash reports",
          "enabled": true,
          "config": {
            "log_level": "INFO",
            "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
            "interval_ms": 1000,
            "max_file_size_mb": 10,
            "retention_days": 30,
            "buffer_size_kb": 1024,
            "flush_interval_ms": 5000
          },
          "advanced_settings": {
            "include_process_id": true,
            "include_thread_id": false,
            "file_path": "/var/log/system.log"
          }
        }
      }

.. http:put:: /api/logging-management/categories/{category_id}/config

   **Description**: Update configuration for a specific category.
   
   **Path Parameters**:
   
   * **category_id** (string): Category identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/logging-management/categories/system/config HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "enabled": true,
        "log_level": "INFO",
        "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
        "interval_ms": 1000,
        "max_file_size_mb": 10,
        "retention_days": 30
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "category_id": "system",
        "message": "Category configuration updated successfully"
      }

.. http:post:: /api/logging-management/categories/batch-update

   **Description**: Update multiple categories at once.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/categories/batch-update HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "updates": [
          {
            "category_id": "system",
            "enabled": true,
            "log_level": "INFO"
          },
          {
            "category_id": "network",
            "enabled": true,
            "log_level": "DEBUG"
          }
        ]
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "updated_categories": 2,
        "results": [
          {"category_id": "system", "success": true},
          {"category_id": "network", "success": true}
        ]
      }

.. http:post:: /api/logging-management/categories/apply-global-level

   **Description**: Apply global log level to all enabled categories.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/categories/apply-global-level HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "global_level": "INFO",
        "exclude_categories": ["security", "debug"]
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "applied": true,
        "message": "Global level applied to 7 categories",
        "applied_categories": ["system", "network", "can", "modbus", "mqtt", "sensor", "ruleEngine"]
      }

Output Destinations (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/logging-management/destinations

   **Description**: Get all configured output destinations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "destinations": {
          "local": {
            "enabled": true,
            "path": "/var/log",
            "available_mb": 144,
            "status": "healthy"
          },
          "cloud": {
            "enabled": true,
            "endpoint": "https://logs.yourcloud.com/api/v1/logs",
            "status": "connected",
            "last_sync": "2024-03-20T14:30:00Z"
          },
          "syslog": {
            "enabled": true,
            "server": "192.168.1.100:514",
            "protocol": "udp",
            "status": "connected"
          },
          "mqtt": {
            "enabled": true,
            "topic": "iot/device/logs",
            "broker": "mqtt://localhost:1883",
            "status": "connected"
          }
        }
      }

.. http:put:: /api/logging-management/destinations/{destination_type}

   **Description**: Configure a specific output destination.
   
   **Path Parameters**:
   
   * **destination_type** (string): Destination type (local, cloud, syslog, mqtt)
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/logging-management/destinations/cloud HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "enabled": true,
        "endpoint": "https://logs.yourcloud.com/api/v1/logs",
        "api_key": "your_api_key_here"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "configured": true,
        "destination_type": "cloud",
        "message": "Cloud destination configured successfully"
      }

.. http:post:: /api/logging-management/destinations/test

   **Description**: Test all configured destinations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/destinations/test HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "destinations": ["cloud", "syslog", "mqtt"],
        "test_message": "Logging destination test"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "tested": true,
        "results": {
          "cloud": {"success": true, "latency_ms": 145},
          "syslog": {"success": true, "latency_ms": 35},
          "mqtt": {"success": true, "latency_ms": 42}
        }
      }

Log Rotation Settings (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/logging-management/rotation

   **Description**: Get log rotation configuration.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "mode": "fifo-compress",
        "max_file_size_mb": 20,
        "max_log_files": 10,
        "compress_older_days": 7,
        "delete_oldest": true,
        "schedule": "daily",
        "time": "02:00",
        "last_rotation": "2024-03-20T02:00:00Z",
        "next_rotation": "2024-03-21T02:00:00Z"
      }

.. http:put:: /api/logging-management/rotation

   **Description**: Update log rotation settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      PUT /api/logging-management/rotation HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "max_file_size_mb": 20,
        "max_log_files": 10,
        "compress_older_days": 7,
        "delete_oldest": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Rotation settings updated",
        "next_rotation": "2024-03-21T02:00:00Z"
      }

.. http:post:: /api/logging-management/rotation/manual

   **Description**: Trigger manual log rotation.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/rotation/manual HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "reason": "Manual rotation triggered from UI",
        "compress_immediately": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "rotated": true,
        "files_rotated": 8,
        "files_compressed": 8,
        "original_size_mb": 156,
        "compressed_size_mb": 52,
        "message": "Log rotation complete"
      }

Gateway Management (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/logging-management/gateways

   **Description**: Get list of available gateways for logging management.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "gateways": [
          {
            "id": "Univa-GW-01",
            "name": "Univa-GW-01",
            "status": "online",
            "ip_address": "192.168.1.101",
            "log_status": {
              "enabled": true,
              "storage_used_mb": 156
            }
          },
          {
            "id": "Univa-GW-02",
            "name": "Univa-GW-02",
            "status": "online",
            "ip_address": "192.168.1.102",
            "log_status": {
              "enabled": true,
              "storage_used_mb": 89
            }
          }
        ],
        "current_gateway": "Univa-GW-01"
      }

.. http:post:: /api/logging-management/gateways/switch

   **Description**: Switch to a different gateway context.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/gateways/switch HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-02",
        "save_current": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "switched": true,
        "gateway_id": "Univa-GW-02",
        "message": "Gateway context switched successfully",
        "requires_reload": true
      }

.. http:get:: /api/logging-management/gateways/{gateway_id}/sync-status

   **Description**: Get synchronization status for a gateway.
   
   **Path Parameters**:
   
   * **gateway_id** (string): Gateway identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-02",
        "sync_status": "in_sync",
        "last_sync": "2024-03-20T14:30:00Z",
        "pending_changes": 0,
        "configuration_match": true
      }

Operations & Testing (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/logging-management/test

   **Description**: Test logging configuration by generating test entries.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/test HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "categories": ["system", "network", "modbus"],
        "levels": ["INFO", "ERROR", "WARNING"],
        "test_messages_count": 10
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "tested": true,
        "messages_generated": 30,
        "files_created": 3,
        "lines_written": 30,
        "errors_found": 0,
        "message": "Logging test completed successfully"
      }

.. http:post:: /api/logging-management/validate

   **Description**: Validate all logging configurations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "validated": true,
        "issues_found": 0,
        "validations": [
          {"check": "storage_available", "status": "pass", "message": "Sufficient storage available"},
          {"check": "destinations_reachable", "status": "pass", "message": "All destinations reachable"},
          {"check": "configuration_valid", "status": "pass", "message": "All configurations valid"}
        ]
      }

.. http:post:: /api/logging-management/restart-service

   **Description**: Restart the logging service.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/restart-service HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "graceful": true,
        "timeout_seconds": 30
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "restarted": true,
        "restart_id": "restart_20240320_162000",
        "estimated_downtime_seconds": 5,
        "message": "Logging service restart scheduled"
      }

Footer Actions
~~~~~~~~~~~~~~

.. http:post:: /api/logging-management/save-all

   **Description**: Save all logging configurations.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/save-all HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "global": {
          "enabled": true,
          "log_level": "INFO",
          "max_storage_mb": 300
        },
        "categories": {
          "system": {
            "enabled": true,
            "log_level": "INFO",
            "interval_ms": 1000
          },
          "network": {
            "enabled": true,
            "log_level": "DEBUG",
            "interval_ms": 5000
          }
        },
        "destinations": {
          "local": true,
          "cloud": "https://logs.yourcloud.com/api/v1/logs",
          "syslog": "192.168.1.100:514"
        },
        "rotation": {
          "max_file_size_mb": 20,
          "max_files": 10
        }
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "saved": true,
        "message": "All logging configurations saved successfully",
        "saved_sections": ["global", "categories", "destinations", "rotation"],
        "restart_required": true
      }

.. http:post:: /api/logging-management/reset

   **Description**: Reset logging configuration to defaults.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/logging-management/reset HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "confirmation": "I confirm to reset all logging settings to defaults",
        "backup_current": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "message": "Logging configuration reset to factory defaults",
        "backup_created": true,
        "restart_required": true
      }

.. http:post:: /api/logging-management/export-config

   **Description**: Export logging configuration.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      Content-Disposition: attachment; filename="logging_config_export.json"
      
      {
        "global_settings": { ... },
        "categories": [ ... ],
        "destinations": { ... },
        "rotation_settings": { ... }
      }

.. http:post:: /api/logging-management/import-config

   **Description**: Import logging configuration from file.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "imported": true,
        "message": "Logging configuration imported successfully",
        "sections_imported": ["global", "categories", "destinations"],
        "restart_required": true
      }

Route Summary
-------------

.. list-table:: Logging Management Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/logging-management`` - Main logging page
     - Yes
   * - Status
     - GET
     - ``/api/logging-management/status`` - Global status
     - Yes
   * - Status
     - PUT
     - ``/api/logging-management/global-settings`` - Update global settings
     - Yes
   * - Status
     - GET
     - ``/api/logging-management/storage-analytics`` - Storage analytics
     - Yes
   * - Categories
     - GET
     - ``/api/logging-management/categories`` - Get all categories
     - Yes
   * - Categories
     - GET
     - ``/api/logging-management/categories/{id}/config`` - Get category config
     - Yes
   * - Categories
     - PUT
     - ``/api/logging-management/categories/{id}/config`` - Update category config
     - Yes
   * - Categories
     - POST
     - ``/api/logging-management/categories/batch-update`` - Batch update categories
     - Yes
   * - Categories
     - POST
     - ``/api/logging-management/categories/apply-global-level`` - Apply global level
     - Yes
   * - Destinations
     - GET
     - ``/api/logging-management/destinations`` - Get destinations
     - Yes
   * - Destinations
     - PUT
     - ``/api/logging-management/destinations/{type}`` - Configure destination
     - Yes
   * - Destinations
     - POST
     - ``/api/logging-management/destinations/test`` - Test destinations
     - Yes
   * - Rotation
     - GET
     - ``/api/logging-management/rotation`` - Get rotation settings
     - Yes
   * - Rotation
     - PUT
     - ``/api/logging-management/rotation`` - Update rotation settings
     - Yes
   * - Rotation
     - POST
     - ``/api/logging-management/rotation/manual`` - Manual rotation
     - Yes
   * - Gateways
     - GET
     - ``/api/logging-management/gateways`` - List gateways
     - Yes
   * - Gateways
     - POST
     - ``/api/logging-management/gateways/switch`` - Switch gateway
     - Yes
   * - Gateways
     - GET
     - ``/api/logging-management/gateways/{id}/sync-status`` - Get sync status
     - Yes
   * - Operations
     - POST
     - ``/api/logging-management/test`` - Test logging
     - Yes
   * - Operations
     - POST
     - ``/api/logging-management/validate`` - Validate config
     - Yes
   * - Operations
     - POST
     - ``/api/logging-management/restart-service`` - Restart service
     - Yes
   * - Footer
     - POST
     - ``/api/logging-management/save-all`` - Save all configs
     - Yes
   * - Footer
     - POST
     - ``/api/logging-management/reset`` - Reset to defaults
     - Yes
   * - Footer
     - POST
     - ``/api/logging-management/export-config`` - Export config
     - Yes
   * - Footer
     - POST
     - ``/api/logging-management/import-config`` - Import config
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /logging-management``
   - Server renders HTML with embedded logging data
   - All 6 sections populated with current configuration

2. **User views global status** (SECTION 1):
   - Storage usage charts and analytics
   - Error statistics and system health
   - "Update Settings" → ``PUT /api/logging-management/global-settings``
   - "View Analytics" → ``GET /api/logging-management/storage-analytics``

3. **User configures log categories** (SECTION 2):
   - Category table with enable/disable toggles
   - "Configure" button → ``GET /api/logging-management/categories/{id}/config``
   - "Update Config" → ``PUT /api/logging-management/categories/{id}/config``
   - "Batch Update" → ``POST /api/logging-management/categories/batch-update``
   - "Apply Global Level" → ``POST /api/logging-management/categories/apply-global-level``

4. **User manages output destinations** (SECTION 3):
   - Destination cards with connection status
   - "Configure Destination" → ``PUT /api/logging-management/destinations/{type}``
   - "Test Destinations" → ``POST /api/logging-management/destinations/test``
   - Real-time status indicators

5. **User sets rotation settings** (SECTION 4):
   - Rotation configuration form
   - Schedule and compression settings
   - "Update Rotation" → ``PUT /api/logging-management/rotation``
   - "Rotate Now" → ``POST /api/logging-management/rotation/manual``

6. **User switches between gateways** (SECTION 5):
   - Gateway selector dropdown
   - "Switch Gateway" → ``POST /api/logging-management/gateways/switch``
   - "Check Sync Status" → ``GET /api/logging-management/gateways/{id}/sync-status``

7. **User performs operations** (SECTION 6):
   - "Test Logging" → ``POST /api/logging-management/test``
   - "Validate Config" → ``POST /api/logging-management/validate``
   - "Restart Service" → ``POST /api/logging-management/restart-service``

8. **User uses footer actions**:
   - "Save All" → ``POST /api/logging-management/save-all``
   - "Reset to Defaults" → ``POST /api/logging-management/reset``
   - "Export Config" → ``POST /api/logging-management/export-config``
   - "Import Config" → ``POST /api/logging-management/import-config``

Error Codes
-----------

.. list-table:: Logging Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - STORAGE_FULL
     - Log storage is full
   * - DESTINATION_UNREACHABLE
     - Log destination cannot be reached
   * - INVALID_LOG_FORMAT
     - Log format specification is invalid
   * - CONFIGURATION_INVALID
     - Logging configuration is invalid
   * - ROTATION_FAILED
     - Log rotation failed
   * - GATEWAY_OFFLINE
     - Target gateway is offline
   * - VALIDATION_ERROR
     - Configuration validation failed
   * - PERMISSION_DENIED
     - Insufficient permissions for operation
   * - CATEGORY_DISABLED
     - Log category is disabled
   * - CONFIGURATION_LOCKED
     - Configuration is being modified by another user

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Cookie: session_token=<token>
   X-Gateway-ID: GW-3920A9

**Important**: This API manages logging configuration for the current gateway specified in `X-Gateway-ID` header. Gateway switching allows management of multiple gateways.

Log Categories
--------------

### Available Categories

.. list-table:: Log Categories
   :widths: 30 40 30
   :header-rows: 1

   * - Category ID
     - Description
     - Default Level
   * - **system**
     - Kernel, watchdog, crash reports
     - INFO
   * - **network**
     - Ethernet, Wi-Fi, 4G, packet drops
     - DEBUG
   * - **can**
     - CAN bus messages, errors, timing
     - WARN
   * - **modbus**
     - Modbus requests, responses, errors
     - ERROR
   * - **mqtt**
     - MQTT publish/subscribe, connections
     - INFO
   * - **security**
     - Authentication, authorization, attacks
     - CRITICAL
   * - **sensor**
     - Sensor readings, calibration, errors
     - INFO
   * - **ruleEngine**
     - Rule execution, conditions, actions
     - DEBUG
   * - **debug**
     - Debug information for developers
     - TRACE

### Log Levels

1. **CRITICAL**: System is unusable
2. **ERROR**: Error conditions
3. **WARNING**: Warning conditions
4. **INFO**: Informational messages
5. **DEBUG**: Debug-level messages
6. **TRACE**: Trace-level messages

Output Destinations
-------------------

### Supported Destinations

1. **Local Storage**: Default debugging location (/var/log)
2. **Cloud Upload**: For cloud dashboards and remote monitoring
3. **Syslog Server**: Industrial IT environments (IP:Port)
4. **MQTT Topic**: Pipe logs to external monitoring systems
5. **WebSocket**: Enable live log viewer in Web UI

### Destination Priorities
- Local storage is always enabled as fallback
- Multiple destinations can be active simultaneously
- Failed destinations are automatically retried

Storage Management
------------------

### Storage Requirements
- **Minimum storage**: 100MB for basic logging
- **Recommended storage**: 1GB for 30 days retention
- **Maximum storage**: Configurable up to 10GB

### Retention Policies
- Logs automatically rotated based on size and age
- Compression applied to logs older than 7 days
- Oldest logs deleted when storage limit reached

### Performance Impact
- **Log writing**: <5% CPU usage during normal operation
- **Compression**: 10-15% CPU during rotation
- **Network destinations**: Varies based on bandwidth

Rate Limiting
-------------

.. list-table:: Logging Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Endpoint Type
     - Requests/Minute
     - Notes
   * - **Configuration changes**
     - 10
     - PUT/POST endpoints
   * - **Data queries**
     - 60
     - GET endpoints
   * - **Batch operations**
     - 5
     - Save-all, batch updates
   * - **System operations**
     - 2
     - Restart, reset operations
   * - **Gateway switching**
     - 5
     - Per user limit

Security Considerations
-----------------------

### Access Control
- Log configuration requires admin privileges
- Sensitive destinations (cloud, syslog) require additional authentication
- All configuration changes are logged for audit

### Data Protection
- Cloud destinations use HTTPS with certificate validation
- API keys encrypted in configuration
- Local log files have restricted permissions

### Audit Trail
- All configuration changes logged with user attribution
- Failed authentication attempts logged
- Unauthorized access attempts trigger alerts

Troubleshooting
---------------

### Common Issues

1. **Logs not appearing**
   - Check category is enabled
   - Verify log level setting
   - Check storage availability

2. **Destination not working**
   - Test destination connectivity
   - Verify authentication credentials
   - Check network connectivity

3. **Storage full**
   - Increase storage limit
   - Reduce retention period
   - Manually rotate logs

4. **Performance issues**
   - Reduce logging frequency
   - Disable debug/trace levels
   - Optimize log formats

### Debug Information

When contacting support, provide:
- Gateway ID and firmware version
- Error message and code
- Log configuration export
- Storage usage statistics
- Network connectivity status

Support Information
-------------------

- **Email**: logging-support@univa.com
- **Phone**: +1 (555) 234-5678
- **Support Hours**: Monday-Friday 8am-8pm EST
- **Documentation**: https://docs.univa.com/logging

Version History
---------------

.. list-table:: API Version History
   :widths: 15 85
   :header-rows: 1

   * - Version
     - Changes
   * - 1.0.0
     - Initial release of Device-Level Logging API
   * - 1.1.0
     - Added multi-gateway management
   * - 1.2.0
     - Enhanced destination testing and validation
   * - 1.3.0
     - Added storage analytics and predictions

Deprecation Notes
-----------------

No endpoints are currently deprecated. All endpoints are fully supported in the current version.

Glossary
--------

.. glossary::

   Log Category
      Group of related log messages (system, network, etc.)

   Log Level
      Severity of log messages (CRITICAL to TRACE)

   Output Destination
      Where logs are sent (local, cloud, syslog, etc.)

   Log Rotation
      Process of archiving old logs and creating new log files

   Retention Period
      How long logs are kept before deletion

   Buffer Size
      Memory allocated for log entries before writing to disk

   Flush Interval
      How often buffered logs are written to storage

   Compression
      Reducing log file size using algorithms like gzip

   Gateway Context
      The specific gateway being managed

   Configuration Sync
      Synchronizing settings between multiple gateways

---
*Document last updated: March 20, 2024*
*API Version: 1.3.0*
*Gateway Version: 2.5.1*
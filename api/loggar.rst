Device-Level Logging API
======================================

APIs for granular control over system logging at the hardware, protocol, and subsystem levels.

Overview
--------

The Device-Level Logging API provides granular control over system logging at the hardware, protocol, and subsystem levels. This API enables configuration of detailed logging for troubleshooting, monitoring, and debugging gateway operations across multiple categories including system, network, CAN bus, Modbus, MQTT, and security events.

Base URL
--------

::

   https://univa-gateway/api/v1/logging

Authentication
--------------

All endpoints require JWT authentication via the ``Authorization`` header:

.. code-block:: http

   Authorization: Bearer <jwt_token>

.. note::

   Advanced logging configuration requires admin-level privileges.

Master Configuration API
------------------------

Get Global Logging Status
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/status

   Retrieve global logging system status and configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "status": {
          "enabled": true,
          "global_log_level": "INFO",
          "storage": {
            "total_mb": 300,
            "used_mb": 156,
            "available_mb": 144,
            "usage_percentage": 52,
            "status": "healthy"
          },
          "rotation": {
            "mode": "fifo-compress",
            "last_rotation": "2025-03-12T14:00:00Z",
            "next_rotation": "2025-03-13T02:00:00Z"
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
        },
        "timestamp": "2025-03-12T15:00:00Z"
      }

Update Global Settings
~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/settings/global

   Update global logging system settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/settings/global HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "global_log_level": "INFO",
        "max_storage_mb": 300,
        "rotation_mode": "fifo-compress",
        "enable_auto_rotation": true,
        "rotation_schedule": "daily",
        "rotation_time": "02:00"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Global logging settings updated",
        "new_settings": {
          "enabled": true,
          "global_log_level": "INFO",
          "max_storage_mb": 300,
          "rotation_mode": "fifo-compress",
          "estimated_log_files": 15
        },
        "requires_restart": false
      }

Get Global Dashboard Data
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/dashboard

   Get dashboard data including storage usage, recent errors, and rotation info.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "dashboard": {
          "storage": {
            "used_mb": 156,
            "total_mb": 300,
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
          },
          "rotation": {
            "last_rotation_ago": "2 hours",
            "next_rotation_in": "~22 hours",
            "oldest_log_days": 14
          }
        },
        "timestamp": "2025-03-12T15:00:00Z"
      }

Category Management API
-----------------------

List Log Categories with UI Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/categories/ui

   Get all log categories formatted for UI display.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "categories": [
          {
            "id": "system",
            "name": "System Logs",
            "description": "Kernel, watchdog, crash reports",
            "enabled": true,
            "log_level": "INFO",
            "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
            "interval_ms": 1000,
            "max_file_size_mb": 10,
            "retention_days": 30,
            "last_activity": "2025-03-12T14:55:00Z",
            "lines_today": 12500,
            "ui_badge": "INFO",
            "ui_badge_color": "blue",
            "ui_interval_display": "1s"
          },
          {
            "id": "network",
            "name": "Network Logs",
            "description": "Ethernet, Wi-Fi, 4G, packet drops",
            "enabled": true,
            "log_level": "DEBUG",
            "log_format": "TIMESTAMP INTERFACE PROTOCOL SOURCE DESTINATION",
            "interval_ms": 5000,
            "max_file_size_mb": 15,
            "retention_days": 14,
            "last_activity": "2025-03-12T14:56:00Z",
            "lines_today": 8500,
            "ui_badge": "DEBUG",
            "ui_badge_color": "green",
            "ui_interval_display": "5s"
          }
        ]
      }

Get Category Configuration for UI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/categories/{category_id}/ui

   Get detailed configuration for a specific log category formatted for UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **category_id** (required): Category ID

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
            "flush_interval_ms": 5000,
            "include_timestamps": true,
            "include_process_id": true,
            "include_thread_id": false,
            "file_path": "/var/log/system.log",
            "compression": "gzip"
          },
          "ui_config": {
            "format_options": [
              {"value": "TIMESTAMP LEVEL MODULE MESSAGE", "label": "Standard (Timestamp + Level + Module + Message)"},
              {"value": "TIMESTAMP LEVEL MESSAGE", "label": "Basic (Timestamp + Level + Message)"},
              {"value": "TIMESTAMP MESSAGE", "label": "Simple (Timestamp + Message)"},
              {"value": "LEVEL MODULE MESSAGE", "label": "Compact (Level + Module + Message - No Timestamp)"},
              {"value": "MODULE MESSAGE", "label": "Minimal (Module + Message)"}
            ],
            "level_options": ["CRITICAL", "ERROR", "WARNING", "INFO", "DEBUG", "TRACE"],
            "advanced_fields": {
              "include_detailed_data": {
                "label": "Include Process Info",
                "description": "Include process ID and thread information",
                "value": true
              },
              "include_debug_info": {
                "label": "Include Thread IDs",
                "description": "Include thread identification in logs",
                "value": false
              }
            }
          }
        }
      }

Update Category Configuration (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/{category_id}/ui

   Update configuration for a specific log category from UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **category_id** (required): Category ID

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/system/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "log_level": "INFO",
        "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
        "interval_ms": 1000,
        "max_file_size_mb": 10,
        "retention_days": 30,
        "buffer_size_kb": 1024,
        "flush_interval_ms": 5000,
        "advanced_settings": {
          "include_detailed_data": true,
          "include_debug_info": false
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Category configuration updated successfully",
        "category": {
          "id": "system",
          "enabled": true,
          "log_level": "INFO",
          "ui_badge": "INFO",
          "ui_badge_color": "blue",
          "ui_interval_display": "1s",
          "estimated_storage_mb_per_day": 25,
          "restart_required": false
        }
      }

Batch Toggle Categories
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/categories/batch-toggle

   Toggle multiple log categories at once.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/categories/batch-toggle HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "categories": ["system", "network", "can", "modbus", "mqtt", "security", "sensor", "ruleEngine", "debug"],
        "enabled": true,
        "apply_global_level": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "updated": 9,
        "results": {
          "system": {"enabled": true, "log_level": "INFO"},
          "network": {"enabled": true, "log_level": "DEBUG"},
          "can": {"enabled": true, "log_level": "WARN"},
          "modbus": {"enabled": true, "log_level": "ERROR"},
          "mqtt": {"enabled": true, "log_level": "INFO"},
          "security": {"enabled": true, "log_level": "CRITICAL"},
          "sensor": {"enabled": false, "log_level": "INFO"},
          "ruleEngine": {"enabled": false, "log_level": "DEBUG"},
          "debug": {"enabled": false, "log_level": "TRACE"}
        }
      }

Apply Global Level to All Categories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/categories/apply-global

   Apply global log level to all enabled categories.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/categories/apply-global HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "global_level": "INFO",
        "exclude_categories": ["security", "debug"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Global level applied to 7 categories",
        "applied_categories": ["system", "network", "can", "modbus", "mqtt", "sensor", "ruleEngine"],
        "excluded_categories": ["security", "debug"],
        "new_level": "INFO"
      }

Output Destinations API
-----------------------

Get Output Destinations for UI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/destinations/ui

   Get all configured logging output destinations formatted for UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "destinations": {
          "local": {
            "enabled": true,
            "path": "/var/log",
            "available_mb": 144,
            "status": "healthy",
            "files_count": 20,
            "ui_label": "Local Storage",
            "ui_description": "Default debugging location (/var/log)"
          },
          "usb": {
            "enabled": false,
            "path": "/media/usb/logs",
            "available_mb": 0,
            "status": "not_connected",
            "files_count": 0,
            "ui_label": "External USB Storage",
            "ui_description": "Field engineers onsite collection"
          },
          "cloud": {
            "enabled": true,
            "endpoint": "https://logs.yourcloud.com/api/v1/logs",
            "status": "connected",
            "last_sync": "2025-03-12T14:30:00Z",
            "pending_uploads": 0,
            "ui_label": "Cloud Upload Endpoint",
            "ui_description": "For cloud dashboards and remote monitoring",
            "placeholder": "https://logs.yourcloud.com/api/v1/logs"
          },
          "syslog": {
            "enabled": true,
            "server": "192.168.1.100:514",
            "protocol": "udp",
            "status": "connected",
            "facility": "local7",
            "ui_label": "Syslog Server",
            "ui_description": "Industrial IT environments (IP:Port)",
            "placeholder": "192.168.1.100:514"
          },
          "mqtt": {
            "enabled": true,
            "topic": "iot/device/logs",
            "broker": "mqtt://localhost:1883",
            "status": "connected",
            "qos": 1,
            "ui_label": "MQTT Topic Output",
            "ui_description": "Pipe logs to external monitoring systems",
            "placeholder": "iot/device/logs"
          },
          "websocket": {
            "enabled": true,
            "clients_connected": 3,
            "status": "active",
            "ui_label": "WebSocket Streaming",
            "ui_description": "Enable live log viewer in Web UI"
          }
        }
      }

Configure Output Destination (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/destinations/{destination_type}/ui

   Configure a specific output destination from UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **destination_type** (required): Destination type

   **Request**:

   .. sourcecode:: http

      PUT /logging/destinations/cloud/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "endpoint": "https://logs.yourcloud.com/api/v1/logs"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "destination": {
          "type": "cloud",
          "enabled": true,
          "endpoint": "https://logs.yourcloud.com/api/v1/logs",
          "status": "configured",
          "ui_label": "Cloud Upload Endpoint",
          "ui_description": "For cloud dashboards and remote monitoring"
        }
      }

Log Rotation API
----------------

Get Rotation Configuration for UI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/rotation/ui

   Get log rotation settings formatted for UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rotation": {
          "enabled": true,
          "mode": "fifo-compress",
          "max_file_size_mb": 20,
          "max_log_files": 10,
          "compress_older_days": 7,
          "delete_oldest": true,
          "schedule": "daily",
          "time": "02:00",
          "last_rotation": "2025-03-12T02:00:00Z",
          "next_rotation": "2025-03-13T02:00:00Z",
          "compression": {
            "algorithm": "gzip",
            "level": 6,
            "remove_original": true
          },
          "ui_settings": {
            "max_file_size": {
              "label": "Max Log File Size",
              "description": "Individual log file size limit",
              "unit": "MB",
              "min": 1,
              "max": 100
            },
            "max_log_files": {
              "label": "Max Log Files",
              "description": "Number of log files to keep",
              "min": 1,
              "max": 100
            },
            "compress_logs": {
              "label": "Compress Older Files",
              "description": "Gzip compress logs older than 7 days",
              "value": true
            },
            "delete_oldest": {
              "label": "Delete Oldest When Full",
              "description": "Automatically delete oldest logs when storage is full",
              "value": true
            }
          }
        }
      }

Update Rotation Settings (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/rotation/ui

   Update log rotation configuration from UI.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/rotation/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "max_file_size_mb": 20,
        "max_log_files": 10,
        "compress_older_days": 7,
        "delete_oldest": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Rotation settings updated",
        "rotation": {
          "max_file_size_mb": 20,
          "max_log_files": 10,
          "compress_older_days": 7,
          "delete_oldest": true,
          "next_rotation": "2025-03-13T02:00:00Z"
        }
      }

Log Operations API
------------------

Test Logging Configuration (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/test/ui

   Test logging configuration by generating test log entries.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/test/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "categories": ["system", "network", "modbus"],
        "levels": ["INFO", "ERROR", "WARNING"],
        "test_messages_count": 10,
        "verify_destinations": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "test_id": "log_test_20250312_161500",
        "status": "completed",
        "messages_generated": 30,
        "destinations_tested": {
          "local": "success",
          "cloud": "success",
          "syslog": "success",
          "mqtt": "success",
          "websocket": "success"
        },
        "verification": {
          "files_created": 3,
          "lines_written": 30,
          "errors_found": 0,
          "latency_ms": {
            "local": 12,
            "cloud": 145,
            "syslog": 35,
            "mqtt": 42
          }
        },
        "message": "Logging configuration test successful! All destinations are reachable."
      }

Trigger Manual Rotation (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/rotation/manual/ui

   Manually trigger log rotation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/rotation/manual/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "reason": "Manual rotation triggered from UI",
        "compress_immediately": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rotation_id": "rotation_manual_20250312_160000",
        "status": "completed",
        "files_rotated": 8,
        "files_compressed": 8,
        "original_size_mb": 156,
        "compressed_size_mb": 52,
        "message": "Log rotation complete! Old logs have been archived."
      }

Restart Logging Service (UI Format)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/restart/ui

   Restart the logging service to apply configuration changes.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/restart/ui HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "graceful": true,
        "timeout_seconds": 30,
        "backup_config": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "restart_id": "restart_20250312_162000",
        "status": "scheduled",
        "graceful": true,
        "estimated_downtime_seconds": 5,
        "backup_created": true,
        "backup_path": "/var/log/config_backup_20250312_162000.json",
        "restart_time": "2025-03-12T16:20:05Z",
        "message": "Logging service restart scheduled. Changes will take effect after restart."
      }

Save All Configurations
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/save-all

   Save all logging configurations at once.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/save-all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "global": {
          "enabled": true,
          "log_level": "INFO",
          "max_storage_mb": 300,
          "rotation_mode": "fifo-compress"
        },
        "categories": {
          "system": {
            "enabled": true,
            "log_level": "INFO",
            "interval_ms": 1000,
            "max_file_size_mb": 10,
            "retention_days": 30
          },
          "network": {
            "enabled": true,
            "log_level": "DEBUG",
            "interval_ms": 5000,
            "max_file_size_mb": 15,
            "retention_days": 14
          }
        },
        "destinations": {
          "local": true,
          "usb": false,
          "cloud": "https://logs.yourcloud.com/api/v1/logs",
          "syslog": "192.168.1.100:514",
          "mqtt": "iot/device/logs",
          "websocket": true
        },
        "rotation": {
          "max_file_size_mb": 20,
          "max_files": 10,
          "compress": true,
          "delete_oldest": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "All logging configurations saved successfully",
        "saved_sections": ["global", "categories", "destinations", "rotation"],
        "restart_required": true,
        "restart_scheduled": true,
        "restart_time": "2025-03-12T16:30:00Z"
      }

Reset to Default Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/reset

   Reset logging configuration to factory defaults.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/reset HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "confirmation": "I confirm to reset all logging settings to defaults",
        "backup_current": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Logging configuration reset to factory defaults",
        "backup_created": true,
        "backup_path": "/var/log/config_backup_pre_reset_20250312_163000.json",
        "restart_required": true,
        "restart_scheduled": true
      }

Gateway Management API
----------------------

Switch Gateway Context
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/gateway/switch

   Switch to a different gateway context.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/gateway/switch HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_id": "Univa-GW-02",
        "save_current": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Gateway context switched to Univa-GW-02",
        "gateway": {
          "id": "Univa-GW-02",
          "name": "Univa-GW-02",
          "status": "online",
          "ip_address": "192.168.1.102",
          "last_seen": "2025-03-12T15:05:00Z"
        },
        "configuration_loaded": true,
        "requires_reload": true
      }

Get Available Gateways
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/gateways/available

   Get list of available gateways for selection.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateways": [
          {
            "id": "Univa-GW-01",
            "name": "Univa-GW-01",
            "status": "online",
            "ip_address": "192.168.1.101",
            "last_seen": "2025-03-12T15:00:00Z",
            "log_status": {
              "enabled": true,
              "storage_used_mb": 156,
              "storage_total_mb": 300
            }
          },
          {
            "id": "Univa-GW-02",
            "name": "Univa-GW-02",
            "status": "online",
            "ip_address": "192.168.1.102",
            "last_seen": "2025-03-12T15:02:00Z",
            "log_status": {
              "enabled": true,
              "storage_used_mb": 89,
              "storage_total_mb": 300
            }
          },
          {
            "id": "Univa-GW-03",
            "name": "Univa-GW-03",
            "status": "offline",
            "ip_address": "192.168.1.103",
            "last_seen": "2025-03-12T08:30:00Z",
            "log_status": {
              "enabled": false,
              "storage_used_mb": 0,
              "storage_total_mb": 300
            }
          }
        ]
      }

Category-Specific Configuration API
-----------------------------------

Get Category UI Configuration Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/categories/{category_id}/ui-template

   Get UI configuration template for a specific category.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **category_id** (required): Category ID

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "category": {
          "id": "system",
          "name": "System Logs",
          "description": "Kernel, watchdog, crash reports",
          "ui_template": {
            "title": "System Logs Configuration",
            "subtitle": "Configure kernel, watchdog, crash reports, and system events",
            "sections": [
              {
                "title": "Basic Settings",
                "fields": [
                  {
                    "type": "toggle",
                    "id": "enabled",
                    "label": "Enable Logging",
                    "description": "Turn logging on/off for this category",
                    "value": true
                  },
                  {
                    "type": "select",
                    "id": "log_level",
                    "label": "Log Level",
                    "description": "Minimum severity level to log",
                    "options": [
                      {"value": "CRITICAL", "label": "CRITICAL"},
                      {"value": "ERROR", "label": "ERROR"},
                      {"value": "WARNING", "label": "WARNING"},
                      {"value": "INFO", "label": "INFO"},
                      {"value": "DEBUG", "label": "DEBUG"},
                      {"value": "TRACE", "label": "TRACE"}
                    ],
                    "value": "INFO"
                  },
                  {
                    "type": "select",
                    "id": "log_format",
                    "label": "Log Format",
                    "description": "Structure of each log entry",
                    "options": [
                      {"value": "TIMESTAMP LEVEL MODULE MESSAGE", "label": "Standard (Timestamp + Level + Module + Message)"},
                      {"value": "TIMESTAMP LEVEL MESSAGE", "label": "Basic (Timestamp + Level + Message)"},
                      {"value": "TIMESTAMP MESSAGE", "label": "Simple (Timestamp + Message)"},
                      {"value": "LEVEL MODULE MESSAGE", "label": "Compact (Level + Module + Message - No Timestamp)"},
                      {"value": "MODULE MESSAGE", "label": "Minimal (Module + Message)"}
                    ],
                    "value": "TIMESTAMP LEVEL MODULE MESSAGE"
                  },
                  {
                    "type": "number",
                    "id": "interval_ms",
                    "label": "Log Interval (ms)",
                    "description": "How often to write logs (0 = immediate)",
                    "min": 0,
                    "max": 60000,
                    "unit": "ms",
                    "value": 1000
                  }
                ]
              },
              {
                "title": "Advanced Settings",
                "fields": [
                  {
                    "type": "number",
                    "id": "max_file_size_mb",
                    "label": "Max File Size",
                    "description": "Maximum size of individual log files",
                    "min": 1,
                    "max": 100,
                    "unit": "MB",
                    "value": 10
                  },
                  {
                    "type": "number",
                    "id": "retention_days",
                    "label": "Retention Period",
                    "description": "How long to keep logs before deletion",
                    "min": 1,
                    "max": 365,
                    "unit": "days",
                    "value": 30
                  },
                  {
                    "type": "number",
                    "id": "buffer_size_kb",
                    "label": "Buffer Size",
                    "description": "Memory buffer for log entries before writing to disk",
                    "min": 128,
                    "max": 16384,
                    "unit": "KB",
                    "value": 1024
                  },
                  {
                    "type": "number",
                    "id": "flush_interval_ms",
                    "label": "Flush Interval",
                    "description": "How often to flush buffer to disk",
                    "min": 100,
                    "max": 30000,
                    "unit": "ms",
                    "value": 5000
                  }
                ]
              },
              {
                "title": "Additional Information",
                "fields": [
                  {
                    "type": "toggle",
                    "id": "include_detailed_data",
                    "label": "Include Process Info",
                    "description": "Include process ID and thread information",
                    "value": true
                  },
                  {
                    "type": "toggle",
                    "id": "include_debug_info",
                    "label": "Include Thread IDs",
                    "description": "Include thread identification in logs",
                    "value": false
                  }
                ]
              }
            ]
          }
        }
      }

Real-Time Updates API
---------------------

Subscribe to Configuration Updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/updates/subscribe

   Subscribe to real-time configuration updates via Server-Sent Events (SSE).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. code-block:: http

      HTTP/1.1 200 OK
      Content-Type: text/event-stream
      Cache-Control: no-cache
      Connection: keep-alive
      
      event: config_update
      data: {"type": "category_updated", "category": "system", "enabled": true, "log_level": "INFO"}
      
      event: storage_update
      data: {"used_mb": 157, "total_mb": 300, "usage_percentage": 52.3}
      
      event: rotation_update
      data: {"last_rotation": "2025-03-12T16:00:00Z", "next_rotation": "2025-03-13T02:00:00Z"}

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
     "timestamp": "2025-03-12T16:00:00Z",
     "request_id": "req_abc123def456",
     "validation_errors": [
       {"field": "max_storage_mb", "error": "Value must be between 10 and 1000"}
     ]
   }

Common Error Codes
~~~~~~~~~~~~~~~~~~

.. list-table:: Logging Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **CATEGORY_DISABLED**
     - Log category is disabled
   * - **STORAGE_FULL**
     - Log storage is full
   * - **DESTINATION_UNREACHABLE**
     - Log destination cannot be reached
   * - **INVALID_LOG_FORMAT**
     - Log format specification is invalid
   * - **PERMISSION_DENIED**
     - Insufficient permissions for operation
   * - **CONFIGURATION_INVALID**
     - Logging configuration is invalid
   * - **ROTATION_FAILED**
     - Log rotation failed
   * - **BUFFER_OVERFLOW**
     - Log buffer overflow
   * - **GATEWAY_SWITCH_FAILED**
     - Failed to switch gateway context
   * - **GATEWAY_OFFLINE**
     - Target gateway is offline
   * - **CONFIGURATION_LOCKED**
     - Configuration is being modified by another user
   * - **VALIDATION_ERROR**
     - Configuration validation failed

Examples
--------

JavaScript - Complete UI Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Initialize logging dashboard
   async function initializeLoggingDashboard() {
       try {
           // Load dashboard data
           const dashboardResponse = await fetch('/api/v1/logging/dashboard', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const dashboardData = await dashboardResponse.json();
           renderDashboard(dashboardData.dashboard);
           
           // Load categories
           const categoriesResponse = await fetch('/api/v1/logging/categories/ui', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const categoriesData = await categoriesResponse.json();
           renderCategories(categoriesData.categories);
           
           // Load destinations
           const destinationsResponse = await fetch('/api/v1/logging/destinations/ui', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const destinationsData = await destinationsResponse.json();
           renderDestinations(destinationsData.destinations);
           
           // Load rotation settings
           const rotationResponse = await fetch('/api/v1/logging/rotation/ui', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const rotationData = await rotationResponse.json();
           renderRotationSettings(rotationData.rotation);
           
           // Load available gateways
           const gatewaysResponse = await fetch('/api/v1/logging/gateways/available', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const gatewaysData = await gatewaysResponse.json();
           populateGatewaySelector(gatewaysData.gateways);
           
           // Subscribe to updates
           subscribeToUpdates();
           
       } catch (error) {
           console.error('Failed to initialize dashboard:', error);
           showError('Failed to load logging configuration');
       }
   }
   
   // Open category configuration modal
   async function openCategoryConfigModal(categoryId) {
       try {
           const response = await fetch(`/api/v1/logging/categories/${categoryId}/ui-template`, {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           const data = await response.json();
           
           if (data.success) {
               renderConfigModal(data.category.ui_template);
               showModal();
           }
       } catch (error) {
           console.error('Failed to load category config:', error);
           showError('Failed to load configuration template');
       }
   }
   
   // Save category configuration
   async function saveCategoryConfig(categoryId, config) {
       try {
           const response = await fetch(`/api/v1/logging/categories/${categoryId}/ui`, {
               method: 'PUT',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(config)
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification(`Configuration saved for ${categoryId}`);
               updateCategoryUI(categoryId, result.category);
               return result;
           } else {
               throw new Error(result.message || 'Save failed');
           }
       } catch (error) {
           console.error('Failed to save category config:', error);
           showError(`Failed to save configuration: ${error.message}`);
       }
   }
   
   // Save all configurations
   async function saveAllConfigurations() {
       const allConfig = {
           global: {
               enabled: document.getElementById('enableLogging').checked,
               log_level: document.getElementById('globalLogLevel').value,
               max_storage_mb: parseInt(document.getElementById('maxStorage').value),
               rotation_mode: document.getElementById('rotationMode').value
           },
           categories: getCategoriesConfig(),
           destinations: getDestinationsConfig(),
           rotation: getRotationConfig()
       };
       
       try {
           const response = await fetch('/api/v1/logging/operations/save-all', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(allConfig)
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification('All configurations saved successfully!');
               if (result.restart_required) {
                   showRestartNotification(result.restart_time);
               }
               return result;
           } else {
               throw new Error(result.message || 'Save failed');
           }
       } catch (error) {
           console.error('Failed to save all configurations:', error);
           showError(`Failed to save configurations: ${error.message}`);
       }
   }
   
   // Test logging configuration
   async function testLoggingConfig() {
       try {
           const response = await fetch('/api/v1/logging/operations/test/ui', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify({
                   categories: ['system', 'network', 'modbus'],
                   levels: ['INFO', 'ERROR', 'WARNING'],
                   test_messages_count: 10,
                   verify_destinations: true
               })
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification(result.message);
               return result;
           } else {
               throw new Error(result.message || 'Test failed');
           }
       } catch (error) {
           console.error('Failed to test logging:', error);
           showError(`Test failed: ${error.message}`);
       }
   }
   
   // Rotate logs now
   async function rotateLogsNow() {
       try {
           const response = await fetch('/api/v1/logging/rotation/manual/ui', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify({
                   reason: 'Manual rotation triggered from UI',
                   compress_immediately: true
               })
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification(result.message);
               updateStorageDisplay();
               return result;
           } else {
               throw new Error(result.message || 'Rotation failed');
           }
       } catch (error) {
           console.error('Failed to rotate logs:', error);
           showError(`Rotation failed: ${error.message}`);
       }
   }
   
   // Switch gateway
   async function switchGateway(gatewayId) {
       try {
           const response = await fetch('/api/v1/logging/gateway/switch', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify({
                   gateway_id: gatewayId,
                   save_current: true
               })
           });
           
           const result = await response.json();
           
           if (result.success) {
               if (result.requires_reload) {
                   location.reload();
               } else {
                   updateGatewayInfo(result.gateway);
               }
               return result;
           } else {
               throw new Error(result.message || 'Gateway switch failed');
           }
       } catch (error) {
           console.error('Failed to switch gateway:', error);
           showError(`Failed to switch gateway: ${error.message}`);
           // Reset dropdown to current gateway
           document.getElementById('gateway-select').value = currentGatewayId;
       }
   }
   
   // Subscribe to real-time updates
   function subscribeToUpdates() {
       const eventSource = new EventSource('/api/v1/logging/updates/subscribe?token=' + token);
       
       eventSource.addEventListener('config_update', function(event) {
           const data = JSON.parse(event.data);
           handleConfigUpdate(data);
       });
       
       eventSource.addEventListener('storage_update', function(event) {
           const data = JSON.parse(event.data);
           updateStorageDisplay(data);
       });
       
       eventSource.addEventListener('rotation_update', function(event) {
           const data = JSON.parse(event.data);
           updateRotationDisplay(data);
       });
       
       eventSource.addEventListener('error', function(event) {
           console.error('SSE connection error:', event);
           // Attempt to reconnect
           setTimeout(subscribeToUpdates, 5000);
       });
   }

Best Practices for UI Integration
---------------------------------

Data Loading Strategy
~~~~~~~~~~~~~~~~~~~~~

1. **Initial Load**:
   - Load dashboard data first for quick display
   - Load categories, destinations, and rotation in parallel
   - Show loading states for each section

2. **Caching**:
   - Cache configuration data locally
   - Use ETag headers for conditional requests
   - Implement optimistic updates

3. **Error Handling**:
   - Show partial data when some requests fail
   - Implement retry logic for failed requests
   - Provide offline fallback options

Real-time Updates
~~~~~~~~~~~~~~~~~

1. **Server-Sent Events**:
   - Use SSE for configuration updates
   - Handle reconnection automatically
   - Update UI components incrementally

2. **Optimistic Updates**:
   - Update UI immediately on user action
   - Show loading states during API calls
   - Roll back changes on failure

Validation
~~~~~~~~~~

1. **Client-side Validation**:
   - Validate input formats before submission
   - Show inline validation errors
   - Prevent invalid submissions

2. **Server-side Validation**:
   - Handle validation errors gracefully
   - Show detailed error messages
   - Suggest corrections when possible

Performance Optimization
~~~~~~~~~~~~~~~~~~~~~~~

1. **Lazy Loading**:
   - Load category templates on demand
   - Paginate large log lists
   - Defer non-critical data loading

2. **Batching**:
   - Batch multiple configuration updates
   - Use compound endpoints for related data
   - Minimize round trips

Security Considerations
-----------------------

1. **Input Sanitization**:
   - Validate all user inputs
   - Escape special characters in log messages
   - Prevent injection attacks

2. **Access Control**:
   - Implement role-based access
   - Log all configuration changes
   - Audit user actions

3. **Data Protection**:
   - Encrypt sensitive configuration data
   - Secure API keys and credentials
   - Implement rate limiting

Troubleshooting
---------------

Common Issues
~~~~~~~~~~~~~

1. **Configuration Not Applying**:
   - Check if restart is required
   - Verify permissions
   - Check for validation errors

2. **Storage Issues**:
   - Monitor storage usage
   - Configure rotation properly
   - Clean up old logs

3. **Connectivity Problems**:
   - Check network connectivity
   - Verify destination URLs
   - Test with minimal configuration

Debugging Tools
~~~~~~~~~~~~~~~

1. **Test Endpoints**:
   - Use `/logging/operations/test/ui` to verify configuration
   - Test individual destinations
   - Generate test log entries

2. **Diagnostic Endpoints**:
   - Check system health
   - View error logs
   - Monitor performance metrics

3. **Log Files**:
   - Access system logs for debugging
   - Check API request logs
   - Review error logs

Support
-------

For API support, contact:

- **Email**: logging-support@univagateway.com
- **Documentation**: https://docs.univagateway.com/api/logging
- **Emergency**: +1-800-LOGGING (564-4644)
- **API Status**: https://status.univagateway.com/api

Version History
---------------

.. list-table:: API Version History
   :widths: 20 30 50
   :header-rows: 1

   * - Version
     - Date
     - Changes
   * - **v1.0**
     - 2025-03-01
     - Initial release with basic logging API
   * - **v1.1**
     - 2025-03-12
     - Added UI-specific endpoints, real-time updates, and gateway management
   * - **v1.2**
     - 2025-03-15
     - Added batch operations and enhanced error handling

Compliance
----------

This API complies with:

1. **RESTful Design Principles**
2. **JSON API Specification**
3. **OAuth 2.0 / JWT Authentication**
4. **GDPR Data Protection**
5. **ISO 27001 Security Standards**

Deprecation Policy
------------------

Endpoints marked as deprecated will:

1. Remain functional for 6 months
2. Return deprecation warnings in headers
3. Have replacement endpoints documented
4. Be removed after the deprecation period

Rate Limiting
-------------

.. list-table:: Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Endpoint Type
     - Requests/Minute
     - Notes
   * - **Configuration changes**
     - 10
     - PUT/POST endpoints
   * - **Data queries**
     - 100
     - GET endpoints
   * - **UI-specific endpoints**
     - 60
     - /ui suffix endpoints
   * - **Real-time updates**
     - 5 connections
     - Per user limit
   * - **Bulk operations**
     - 5
     - Save-all, batch updates

License
-------

Copyright © 2025 Univa Gateway. All rights reserved.

This API is proprietary and confidential. Unauthorized use, copying, or distribution is prohibited.
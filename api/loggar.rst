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

Category Management API
-----------------------

List Log Categories
~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/categories

   Get all available log categories and their status.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **enabled_only** (optional): Show only enabled categories (true/false)
   * **level_filter** (optional): Filter by minimum log level

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
            "lines_today": 12500
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
            "lines_today": 8500
          },
          {
            "id": "can",
            "name": "CAN Bus Logs",
            "description": "RX/TX errors, invalid frames",
            "enabled": true,
            "log_level": "WARN",
            "log_format": "TIMESTAMP CAN_ID DIRECTION DATA LENGTH",
            "interval_ms": 100,
            "max_file_size_mb": 20,
            "retention_days": 7,
            "last_activity": "2025-03-12T14:57:00Z",
            "lines_today": 32000
          }
        ],
        "total_categories": 9,
        "enabled_categories": 6,
        "disabled_categories": 3
      }

Get Category Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/categories/{category_id}

   Get detailed configuration for a specific log category.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **category_id** (required): One of ``system``, ``network``, ``can``, ``modbus``, ``mqtt``, ``security``, ``sensor``, ``ruleEngine``, ``debug``

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
            "compression": "gzip",
            "rotation_strategy": "size_and_time"
          },
          "advanced_settings": {
            "enable_stack_traces": false,
            "enable_memory_tracking": false,
            "max_message_length": 2048,
            "queue_size": 1000
          },
          "statistics": {
            "current_file_size_mb": 2.5,
            "lines_written_today": 12500,
            "avg_lines_per_hour": 520,
            "last_write": "2025-03-12T15:00:00Z",
            "errors_today": 12,
            "warnings_today": 45
          }
        }
      }

Update Category Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/{category_id}

   Update configuration for a specific log category.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **category_id** (required): Category ID as listed above

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/system HTTP/1.1
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
        "advanced": {
          "include_timestamps": true,
          "include_process_id": true,
          "include_thread_id": false,
          "enable_stack_traces": false
        },
        "category_specific": {
          "include_packet_data": false,
          "include_interface_stats": true
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
          "estimated_storage_mb_per_day": 25,
          "restart_required": false,
          "validation": {
            "format_valid": true,
            "storage_available": true,
            "permissions_ok": true
          }
        }
      }

Batch Update Categories
~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/batch

   Update multiple log categories at once.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/batch HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "updates": [
          {
            "category_id": "system",
            "log_level": "INFO",
            "enabled": true
          },
          {
            "category_id": "network",
            "log_level": "DEBUG",
            "interval_ms": 2000
          },
          {
            "category_id": "modbus",
            "retention_days": 30
          }
        ],
        "apply_immediately": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "updated": 3,
        "failed": 0,
        "results": {
          "system": "success",
          "network": "success",
          "modbus": "success"
        },
        "restart_required": false
      }

Enable/Disable Category
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/categories/{category_id}/toggle

   Quickly enable or disable a log category.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **category_id** (required): Category ID as listed above

   **Request**:

   .. sourcecode:: http

      POST /logging/categories/system/toggle HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "reason": "Enabling for troubleshooting",
        "apply_global_level": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "System logs enabled",
        "category": {
          "id": "system",
          "enabled": true,
          "log_level": "INFO",
          "estimated_daily_volume_mb": 25,
          "next_rotation": "2025-03-13T02:00:00Z"
        }
      }

Category-Specific Configuration
-------------------------------

System Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/system/config

   Configure kernel, watchdog, and system-level logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/system/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "INFO",
        "log_format": "TIMESTAMP LEVEL MODULE MESSAGE",
        "interval_ms": 1000,
        "include_process_id": true,
        "include_thread_id": false,
        "enable_kernel_logging": true,
        "enable_watchdog_logging": true,
        "enable_crash_reports": true,
        "max_stack_trace_depth": 10,
        "buffer_size_kb": 1024
      }

Network Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/network/config

   Configure network interface and packet logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/network/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "DEBUG",
        "log_format": "TIMESTAMP INTERFACE PROTOCOL SOURCE DESTINATION",
        "interval_ms": 5000,
        "interfaces": ["eth0", "wlan0", "wwan0"],
        "protocols": ["tcp", "udp", "icmp"],
        "include_packet_data": false,
        "include_interface_stats": true,
        "log_packet_drops": true,
        "log_connection_errors": true,
        "log_dhcp_events": true,
        "max_packet_log_size": 1500
      }

CAN Bus Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/can/config

   Configure CAN bus communication logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/can/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "WARN",
        "log_format": "TIMESTAMP CAN_ID DIRECTION DATA LENGTH",
        "interval_ms": 100,
        "can_interfaces": ["can0", "can1"],
        "baud_rates": [500000, 250000],
        "include_raw_data": false,
        "include_error_frames": true,
        "log_tx_errors": true,
        "log_rx_errors": true,
        "log_bus_errors": true,
        "filter_can_ids": ["0x100-0x1FF"],
        "max_frame_log_size": 64
      }

Modbus Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/modbus/config

   Configure Modbus protocol logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/modbus/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "ERROR",
        "log_format": "TIMESTAMP DEVICE FUNCTION REGISTER VALUE",
        "interval_ms": 500,
        "protocols": ["rtu", "tcp"],
        "include_raw_bytes": false,
        "include_slave_responses": true,
        "log_read_errors": true,
        "log_write_errors": true,
        "log_timeout_errors": true,
        "log_parity_errors": true,
        "device_ids": [1, 2, 3],
        "register_ranges": ["40001-49999"]
      }

MQTT Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/mqtt/config

   Configure MQTT broker logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/mqtt/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "INFO",
        "log_format": "TIMESTAMP TOPIC QOS PAYLOAD",
        "interval_ms": 2000,
        "brokers": ["mqtt://localhost:1883", "mqtts://cloud.example.com:8883"],
        "include_payload": false,
        "include_qos_details": true,
        "log_connection_events": true,
        "log_publish_events": true,
        "log_subscribe_events": true,
        "log_message_delivery": true,
        "topics": ["iot/device/#", "telemetry/#"],
        "max_payload_log_size": 256
      }

Security Logs Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/categories/security/config

   Configure security and authentication logging.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/categories/security/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_level": "CRITICAL",
        "log_format": "TIMESTAMP SOURCE EVENT USER RESULT",
        "interval_ms": 0,
        "include_ip_address": true,
        "include_user_agent": false,
        "log_authentication_attempts": true,
        "log_failed_logins": true,
        "log_access_violations": true,
        "log_configuration_changes": true,
        "log_firewall_events": true,
        "alert_on_critical": true,
        "retention_days": 90
      }

Output Destinations API
-----------------------

Get Output Destinations
~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/destinations

   Get all configured logging output destinations.

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
            "files_count": 20
          },
          "usb": {
            "enabled": false,
            "path": "/media/usb/logs",
            "available_mb": 0,
            "status": "not_connected",
            "files_count": 0
          },
          "cloud": {
            "enabled": true,
            "endpoint": "https://logs.yourcloud.com/api/v1/logs",
            "status": "connected",
            "last_sync": "2025-03-12T14:30:00Z",
            "pending_uploads": 0
          },
          "syslog": {
            "enabled": true,
            "server": "192.168.1.100:514",
            "protocol": "udp",
            "status": "connected",
            "facility": "local7"
          },
          "mqtt": {
            "enabled": true,
            "topic": "iot/device/logs",
            "broker": "mqtt://localhost:1883",
            "status": "connected",
            "qos": 1
          },
          "websocket": {
            "enabled": true,
            "clients_connected": 3,
            "status": "active"
          }
        }
      }

Configure Output Destination
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/destinations/{destination_type}

   Configure a specific output destination.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **destination_type** (required): One of ``local``, ``usb``, ``cloud``, ``syslog``, ``mqtt``, ``websocket``

   **Request**:

   .. sourcecode:: http

      PUT /logging/destinations/cloud HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "endpoint": "https://logs.yourcloud.com/api/v1/logs",
        "api_key": "encrypted_api_key_here",
        "batch_size": 100,
        "batch_interval_ms": 10000,
        "compression": "gzip",
        "encryption": true,
        "categories": ["system", "security", "error"],
        "retry_policy": {
          "max_attempts": 3,
          "retry_delay_ms": 5000,
          "exponential_backoff": true
        }
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
          "test_connection": "success",
          "estimated_latency_ms": 45
        }
      }

Test Destination Connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/destinations/{destination_type}/test

   Test connectivity to a logging destination.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **destination_type** (required): Destination type

   **Request**:

   .. sourcecode:: http

      POST /logging/destinations/cloud/test HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "test_data": "Test log entry from Univa Gateway",
        "timeout_ms": 5000
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "destination": "cloud",
        "status": "connected",
        "latency_ms": 42,
        "bandwidth_kbps": 1250,
        "authentication": "valid",
        "recommendations": []
      }

Log Rotation API
----------------

Get Rotation Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/rotation

   Get log rotation settings.

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
          }
        },
        "statistics": {
          "files_rotated_today": 2,
          "compressed_files": 12,
          "total_saved_mb": 245,
          "compression_ratio": 0.35
        }
      }

Update Rotation Settings
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /logging/rotation

   Update log rotation configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /logging/rotation HTTP/1.1
      Authorization: Bearer <token>
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
        "compression": {
          "algorithm": "gzip",
          "level": 6,
          "remove_original": true
        },
        "notify_on_rotation": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Rotation settings updated",
        "rotation": {
          "mode": "fifo-compress",
          "estimated_daily_rotation": 2,
          "estimated_storage_savings_mb": 50,
          "next_rotation": "2025-03-13T02:00:00Z"
        }
      }

Trigger Manual Rotation
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/rotation/manual

   Manually trigger log rotation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/rotation/manual HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "reason": "Manual rotation before maintenance",
        "categories": ["all"], // or specific categories
        "compress_immediately": true,
        "archive_name": "logs_pre_maintenance_20250312"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rotation_id": "rotation_manual_20250312_160000",
        "status": "in_progress",
        "files_to_rotate": 8,
        "estimated_size_mb": 156,
        "archive_path": "/var/log/archives/logs_pre_maintenance_20250312.tar.gz",
        "progress_url": "/api/v1/logging/rotation/progress/rotation_manual_20250312_160000"
      }

Get Rotation Progress
~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/rotation/progress/{rotation_id}

   Check status of rotation operation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **rotation_id** (required): ID of the rotation operation

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rotation_id": "rotation_manual_20250312_160000",
        "status": "completed",
        "progress": 100,
        "files_rotated": 8,
        "files_compressed": 8,
        "original_size_mb": 156,
        "compressed_size_mb": 52,
        "compression_ratio": 0.33,
        "duration_seconds": 12,
        "archive_path": "/var/log/archives/logs_pre_maintenance_20250312.tar.gz",
        "archive_size_mb": 52
      }

Log Statistics API
------------------

Get Real-Time Statistics
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/statistics/realtime

   Get real-time logging statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **time_window** (optional): Time window in minutes (default: 5)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "statistics": {
          "timestamp": "2025-03-12T15:00:00Z",
          "time_window_minutes": 5,
          "overall": {
            "lines_per_second": 8.5,
            "bytes_per_second": 5120,
            "active_categories": 6,
            "total_lines": 42156,
            "storage_used_mb": 156
          },
          "by_category": {
            "system": {
              "lines_per_second": 1.2,
              "bytes_per_second": 850,
              "lines_total": 12500,
              "current_file_size_mb": 2.5
            },
            "network": {
              "lines_per_second": 0.8,
              "bytes_per_second": 1200,
              "lines_total": 8500,
              "current_file_size_mb": 3.2
            },
            "can": {
              "lines_per_second": 5.0,
              "bytes_per_second": 2500,
              "lines_total": 32000,
              "current_file_size_mb": 8.5
            }
          },
          "by_level": {
            "critical": 0,
            "error": 12,
            "warning": 47,
            "info": 32500,
            "debug": 9500,
            "trace": 97
          }
        }
      }

Get Historical Statistics
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/statistics/historical

   Get historical logging statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **start_date** (required): ISO start date
   * **end_date** (required): ISO end date
   * **interval** (optional): "hourly", "daily", "weekly" (default: "daily")

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "statistics": {
          "time_range": {
            "start": "2025-03-10T00:00:00Z",
            "end": "2025-03-12T23:59:59Z"
          },
          "interval": "daily",
          "data": [
            {
              "date": "2025-03-10",
              "total_lines": 38500,
              "storage_used_mb": 145,
              "top_categories": [
                {"category": "can", "lines": 21000},
                {"category": "system", "lines": 9500},
                {"category": "network", "lines": 8000}
              ],
              "errors": 15,
              "warnings": 42
            },
            {
              "date": "2025-03-11",
              "total_lines": 39500,
              "storage_used_mb": 150,
              "top_categories": [
                {"category": "can", "lines": 22000},
                {"category": "modbus", "lines": 10000},
                {"category": "system", "lines": 7500}
              ],
              "errors": 18,
              "warnings": 38
            }
          ],
          "summary": {
            "average_lines_per_day": 40052,
            "average_errors_per_day": 15,
            "storage_growth_mb_per_day": 5.5,
            "busiest_category": "can",
            "quietest_category": "security"
          }
        }
      }

Get Error Statistics
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/statistics/errors

   Get detailed error statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **severity** (optional): "critical", "error", "warning" (default: all)
   * **time_window** (optional): Time window in hours (default: 24)
   * **category** (optional): Filter by category

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "errors": {
          "time_window_hours": 24,
          "total_errors": 59,
          "by_severity": {
            "critical": 0,
            "error": 12,
            "warning": 47
          },
          "by_category": {
            "network": {
              "total": 8,
              "error": 5,
              "warning": 3,
              "most_common": "Connection timeout"
            },
            "modbus": {
              "total": 25,
              "error": 7,
              "warning": 18,
              "most_common": "CRC check failed"
            },
            "system": {
              "total": 15,
              "error": 0,
              "warning": 15,
              "most_common": "High memory usage"
            }
          },
          "trend": {
            "last_hour": 3,
            "last_6_hours": 18,
            "last_12_hours": 35,
            "last_24_hours": 59
          },
          "recent_errors": [
            {
              "timestamp": "2025-03-12T14:58:00Z",
              "category": "modbus",
              "severity": "error",
              "message": "CRC check failed for device 3",
              "count": 7
            }
          ]
        }
      }

Log Operations API
------------------

Test Logging Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/test

   Test logging configuration by generating test log entries.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/test HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "categories": ["system", "network", "modbus"],
        "levels": ["INFO", "ERROR", "WARNING"],
        "test_messages_count": 10,
        "verify_destinations": true,
        "cleanup_after_test": true
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
        "recommendations": [
          "Increase buffer size for network logs to improve performance",
          "Consider compressing cloud uploads to reduce bandwidth usage"
        ]
      }

Restart Logging Service
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/restart

   Restart the logging service to apply configuration changes.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/restart HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "graceful": true,
        "timeout_seconds": 30,
        "backup_config": true,
        "confirmation": {
          "user": "admin",
          "reason": "Applying new logging configuration"
        }
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
        "restart_time": "2025-03-12T16:20:05Z"
      }

Clear Log Files
~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/clear

   Clear log files (with confirmation).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/clear HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "categories": ["all"], // or specific categories
        "retain_last_days": 1,
        "exclude_critical": true,
        "compress_before_delete": true,
        "confirmation": {
          "user": "admin",
          "password": "encrypted_password",
          "reason": "Storage cleanup before system update"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "clear_id": "clear_20250312_163000",
        "status": "completed",
        "files_deleted": 8,
        "files_compressed": 12,
        "storage_freed_mb": 85,
        "remaining_logs_mb": 71,
        "retained_logs_days": 1,
        "excluded_categories": ["security", "system"]
      }

Export Log Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/export-config

   Export logging configuration for backup or migration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/export-config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "format": "json", // "json", "yaml"
        "include_passwords": false,
        "encrypt": true,
        "password": "optional_encryption_password"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "config_export_20250312_163500",
        "status": "ready",
        "format": "json",
        "size_kb": 45,
        "download_url": "/api/v1/logging/operations/export-config/download/config_export_20250312_163500",
        "checksum": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
      }

Import Log Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/operations/import-config

   Import logging configuration from backup.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/operations/import-config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "config_data": "base64_encoded_config_file_or_url",
        "validate_only": false,
        "backup_current": true,
        "apply_after_import": true,
        "confirmation": {
          "user": "admin",
          "password": "encrypted_password"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "import_id": "config_import_20250312_164000",
        "status": "validating",
        "backup_created": true,
        "backup_path": "/var/log/config_backup_pre_import_20250312_164000.json",
        "validation_results": {
          "format_valid": true,
          "version_compatible": true,
          "permissions_ok": true,
          "storage_sufficient": true
        },
        "estimated_apply_time": "2025-03-12T16:40:30Z"
      }

Real-Time Monitoring API
------------------------

WebSocket Log Stream
~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/logging/ws?token=<jwt_token>

Connection Parameters
^^^^^^^^^^^^^^^^^^^^^

* **categories**: Comma-separated categories to subscribe to
* **levels**: Comma-separated log levels
* **format**: "json" or "text"

Client Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "action": "subscribe",
     "categories": ["system", "network", "security"],
     "levels": ["ERROR", "WARN", "INFO"],
     "filter": "connection"
   }

Server Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "event": "log_entry",
     "timestamp": "2025-03-12T16:00:00Z",
     "category": "system",
     "level": "INFO",
     "message": "Logging service started successfully",
     "source": "logging-daemon",
     "process_id": 1234,
     "tags": ["startup", "service"]
   }

Get Active Log Streams
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/streams/active

   Get information about active log streams.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "streams": [
          {
            "stream_id": "ws_stream_20250312_150000",
            "client_ip": "192.168.1.100",
            "user": "admin",
            "categories": ["system", "network"],
            "levels": ["ERROR", "WARN", "INFO"],
            "connected_since": "2025-03-12T15:00:00Z",
            "messages_sent": 2450,
            "bytes_sent_mb": 12.5
          },
          {
            "stream_id": "syslog_stream_1",
            "destination": "192.168.1.100:514",
            "protocol": "udp",
            "categories": ["all"],
            "connected_since": "2025-03-12T08:00:00Z",
            "messages_sent": 125000,
            "bytes_sent_mb": 650
          }
        ],
        "total_streams": 8,
        "total_throughput_mbps": 1.2
      }

Health & Diagnostics API
------------------------

Get Logging Health Status
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /logging/health

   Get comprehensive health status of logging system.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "health": {
          "overall": "healthy",
          "components": {
            "local_storage": {
              "status": "healthy",
              "available_mb": 144,
              "usage_percentage": 52,
              "iops": 125
            },
            "rotation_service": {
              "status": "healthy",
              "last_success": "2025-03-12T02:00:00Z",
              "next_scheduled": "2025-03-13T02:00:00Z"
            },
            "compression": {
              "status": "healthy",
              "algorithm": "gzip",
              "compression_ratio": 0.35
            },
            "destinations": {
              "local": "healthy",
              "cloud": "connected",
              "syslog": "connected",
              "mqtt": "connected",
              "websocket": "active"
            }
          },
          "performance": {
            "lines_per_second": 8.5,
            "avg_latency_ms": 45,
            "buffer_usage_percentage": 65,
            "queue_depth": 120
          },
          "alerts": [
            {
              "level": "warning",
              "component": "local_storage",
              "message": "Storage usage at 52%, monitor regularly",
              "timestamp": "2025-03-12T15:00:00Z"
            }
          ],
          "recommendations": [
            "Consider increasing storage allocation if logging volume increases",
            "Review retention periods for optimal storage usage"
          ]
        }
      }

Run Logging Diagnostics
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /logging/diagnostics/run

   Run comprehensive diagnostics on logging system.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /logging/diagnostics/run HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "tests": [
          "storage_health",
          "destination_connectivity",
          "performance",
          "configuration_validation"
        ],
        "verbose": true,
        "generate_report": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "diagnostic_id": "log_diag_20250312_170000",
        "status": "running",
        "tests": [
          {
            "name": "storage_health",
            "status": "in_progress",
            "progress": 45
          },
          {
            "name": "destination_connectivity",
            "status": "pending",
            "progress": 0
          }
        ],
        "estimated_completion": "2025-03-12T17:02:00Z",
        "results_url": "/api/v1/logging/diagnostics/results/log_diag_20250312_170000"
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
     "timestamp": "2025-03-12T16:00:00Z",
     "request_id": "req_abc123def456"
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
   * - **COMPRESSION_FAILED**
     - Log compression failed
   * - **ENCRYPTION_FAILED**
     - Log encryption failed
   * - **RATE_LIMIT_EXCEEDED**
     - Log rate limit exceeded
   * - **CATEGORY_PROTECTED**
     - Category cannot be modified (e.g., security)

Examples
--------

Python - Logging Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Get logging status
   headers = {"Authorization": "Bearer your_token"}
   
   status_response = requests.get(
       "https://univa-gateway/api/v1/logging/status",
       headers=headers
   )
   
   status = status_response.json()
   print(f"Logging Enabled: {status['status']['enabled']}")
   print(f"Storage Used: {status['status']['storage']['used_mb']}MB")
   print(f"Available: {status['status']['storage']['available_mb']}MB")
   
   # List categories
   categories_response = requests.get(
       "https://univa-gateway/api/v1/logging/categories",
       headers=headers,
       params={"enabled_only": True}
   )
   
   categories = categories_response.json()
   for category in categories['categories']:
       print(f"{category['name']}: {category['log_level']} (Lines Today: {category['lines_today']})")
   
   # Configure system logs
   system_config = {
       "enabled": True,
       "log_level": "DEBUG",
       "interval_ms": 500,
       "max_file_size_mb": 20,
       "retention_days": 14,
       "include_process_id": True,
       "include_thread_id": True
   }
   
   config_response = requests.put(
       "https://univa-gateway/api/v1/logging/categories/system",
       json=system_config,
       headers=headers
   )
   
   print(f"Configuration updated: {config_response.json()['message']}")
   
   # Get real-time statistics
   stats_response = requests.get(
       "https://univa-gateway/api/v1/logging/statistics/realtime",
       headers=headers,
       params={"time_window": 10}
   )
   
   stats = stats_response.json()
   print(f"Lines per second: {stats['statistics']['overall']['lines_per_second']}")
   print(f"Active categories: {stats['statistics']['overall']['active_categories']}")

Python - Log Operations
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Test logging configuration
   test_request = {
       "categories": ["system", "network", "modbus"],
       "levels": ["INFO", "ERROR", "WARNING"],
       "test_messages_count": 5,
       "verify_destinations": True,
       "cleanup_after_test": True
   }
   
   test_response = requests.post(
       "https://univa-gateway/api/v1/logging/operations/test",
       json=test_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   test_result = test_response.json()
   print(f"Test ID: {test_result['test_id']}")
   print(f"Messages Generated: {test_result['messages_generated']}")
   print(f"Destinations Tested: {test_result['destinations_tested']}")
   
   # Run log rotation
   rotation_request = {
       "reason": "Monthly log rotation",
       "categories": ["all"],
       "compress_immediately": True,
       "archive_name": "logs_monthly_20250312"
   }
   
   rotation_response = requests.post(
       "https://univa-gateway/api/v1/logging/rotation/manual",
       json=rotation_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   rotation_result = rotation_response.json()
   rotation_id = rotation_result['rotation_id']
   print(f"Rotation started: {rotation_id}")
   
   # Monitor rotation progress
   import time
   while True:
       progress_response = requests.get(
           f"https://univa-gateway/api/v1/logging/rotation/progress/{rotation_id}",
           headers={"Authorization": "Bearer your_token"}
       )
       
       progress = progress_response.json()
       print(f"Rotation progress: {progress.get('progress', 0)}%")
       
       if progress['status'] == 'completed':
           print(f"Rotation completed: {progress['files_rotated']} files")
           print(f"Compressed size: {progress['compressed_size_mb']}MB")
           break
       
       time.sleep(2)

JavaScript - Real-time Log Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket connection for real-time log monitoring
   const ws = new WebSocket('wss://univa-gateway/api/v1/logging/ws?token=' + token);
   
   ws.onopen = function() {
       console.log('Connected to Logging WebSocket');
       
       // Subscribe to log events
       ws.send(JSON.stringify({
           action: 'subscribe',
           categories: ['system', 'network', 'security'],
           levels: ['ERROR', 'WARN', 'INFO'],
           filter: ''
       }));
   };
   
   ws.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       if (message.event === 'log_entry') {
           displayLogEntry(message);
           
           // Update statistics
           updateLogStatistics(message.category, message.level);
           
           // Check for critical errors
           if (message.level === 'ERROR' || message.level === 'CRITICAL') {
               showErrorAlert(message);
           }
       }
   };
   
   ws.onerror = function(error) {
       console.error('WebSocket error:', error);
   };
   
   ws.onclose = function() {
       console.log('WebSocket connection closed');
   };
   
   // UI update functions
   function displayLogEntry(log) {
       const logEntry = document.createElement('div');
       logEntry.className = `log-entry category-${log.category} level-${log.level.toLowerCase()}`;
       logEntry.innerHTML = `
           <span class="timestamp">${new Date(log.timestamp).toLocaleTimeString()}</span>
           <span class="category">${log.category}</span>
           <span class="level">${log.level}</span>
           <span class="message">${log.message}</span>
       `;
       
       const container = document.getElementById('log-container');
       container.appendChild(logEntry);
       
       // Auto-scroll to bottom
       container.scrollTop = container.scrollHeight;
   }
   
   function updateLogStatistics(category, level) {
       // Update category counters
       const catCounter = document.getElementById(`counter-${category}`);
       if (catCounter) {
           catCounter.textContent = parseInt(catCounter.textContent) + 1;
       }
       
       // Update level counters
       const levelCounter = document.getElementById(`counter-${level.toLowerCase()}`);
       if (levelCounter) {
           levelCounter.textContent = parseInt(levelCounter.textContent) + 1;
       }
   }

JavaScript - Logging Dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Get logging statistics
   async function loadLoggingStatistics() {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch('/api/v1/logging/statistics/realtime?time_window=10', {
               headers: {
                   'Authorization': `Bearer ${token}`
               }
           });
           
           const stats = await response.json();
           renderStatisticsCharts(stats.statistics);
           updateStorageUsage(stats.statistics.overall.storage_used_mb);
           updatePerformanceMetrics(stats.statistics.overall);
       } catch (error) {
           console.error('Error loading statistics:', error);
       }
   }
   
   // Update log category configuration
   async function updateLogCategory(categoryId, config) {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch(`/api/v1/logging/categories/${categoryId}`, {
               method: 'PUT',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(config)
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification(`Log category ${categoryId} updated`);
               return result.category;
           } else {
               throw new Error(result.message || 'Category update failed');
           }
       } catch (error) {
           console.error('Error updating log category:', error);
           showError('Failed to update log category configuration');
       }
   }
   
   // Configure log rotation
   async function configureLogRotation(settings) {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch('/api/v1/logging/rotation', {
               method: 'PUT',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(settings)
           });
           
           const result = await response.json();
           
           if (result.success) {
               showSuccessNotification('Log rotation configuration updated');
               updateRotationSchedule(result.rotation.next_rotation);
               return result;
           } else {
               throw new Error(result.message || 'Rotation configuration failed');
           }
       } catch (error) {
           console.error('Error configuring log rotation:', error);
           showError('Failed to update log rotation settings');
       }
   }
   
   // Run logging diagnostics
   async function runLoggingDiagnostics() {
       const token = localStorage.getItem('token');
       
       const diagnosticConfig = {
           tests: [
               'storage_health',
               'destination_connectivity',
               'performance',
               'configuration_validation'
           ],
           verbose: true,
           generate_report: true
       };
       
       try {
           const response = await fetch('/api/v1/logging/diagnostics/run', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(diagnosticConfig)
           });
           
           const result = await response.json();
           
           if (result.success) {
               startDiagnosticMonitoring(result.diagnostic_id);
               return result;
           } else {
               throw new Error(result.message || 'Diagnostic failed to start');
           }
       } catch (error) {
           console.error('Error running diagnostics:', error);
           showError('Failed to run logging diagnostics');
       }
   }

Best Practices
--------------

Category Configuration
~~~~~~~~~~~~~~~~~~~~~~

1. **Selective Logging**
   - Enable only necessary categories
   - Set appropriate log levels per category
   - Consider performance impact

2. **Performance Optimization**
   - Adjust intervals based on importance
   - Configure appropriate buffer sizes
   - Use compression for network destinations

3. **Storage Management**
   - Set appropriate retention periods
   - Configure rotation schedules
   - Monitor storage usage regularly

Destination Management
~~~~~~~~~~~~~~~~~~~~~~

1. **Multiple Destinations**
   - Use local storage for reliability
   - Configure cloud for backup/analysis
   - Use syslog for centralized logging

2. **Network Optimization**
   - Use compression for remote destinations
   - Configure retry policies
   - Monitor connection health

3. **Security Considerations**
   - Encrypt sensitive log data
   - Secure destination credentials
   - Monitor access patterns

Rotation & Retention
~~~~~~~~~~~~~~~~~~~~

1. **Rotation Strategy**
   - Balance frequency vs. performance
   - Use compression to save space
   - Maintain appropriate retention

2. **Storage Optimization**
   - Delete old logs automatically
   - Archive important logs
   - Monitor storage growth

3. **Compliance Requirements**
   - Retain security logs per policy
   - Maintain audit trails
   - Document rotation policies

Security Considerations
-----------------------

Authentication & Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Access Control**
   - Admin-level for configuration changes
   - Operator-level for monitoring
   - Viewer-level for read-only access

2. **Audit Trail**
   - Log all configuration changes
   - Track access to log files
   - Monitor for unauthorized access

3. **Security Logs**
   - Special protection for security logs
   - Cannot be disabled without approval
   - Extended retention periods

Data Protection
~~~~~~~~~~~~~~~

1. **Encryption**
   - Encrypt logs in transit
   - Encrypt sensitive log data
   - Secure credential storage

2. **Access Control**
   - Restrict log file access
   - Control destination access
   - Monitor log exports

3. **Compliance**
   - Follow data retention policies
   - Secure log storage
   - Regular security audits

Audit & Compliance
~~~~~~~~~~~~~~~~~~

1. **Logging Requirements**
   - Configuration change logs: 90 days
   - Security event logs: 180 days
   - System operation logs: 30 days

2. **Reporting**
   - Regular compliance reports
   - Security audit logs
   - Storage usage reports

3. **Documentation**
   - Logging policy documentation
   - Configuration documentation
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
     - 300MB minimum for logs
   * - **Memory**
     - 256MB for logging buffers
   * - **CPU**
     - Multi-core for compression
   * - **Network**
     - Stable for remote destinations
   * - **Filesystem**
     - Support for log rotation

Performance Considerations
--------------------------

1. **Logging Performance**
   - Maximum throughput: 1000 lines/sec
   - Buffer memory: 256MB-1GB
   - Compression overhead: 5-20% CPU

2. **Network Impact**
   - Local logging: <1% network
   - Remote logging: 10-100 KB/s
   - Compression savings: 60-80%

3. **Storage Impact**
   - Daily growth: 10-100 MB/day
   - Compression ratio: 0.3-0.5
   - Rotation overhead: <5% CPU

Support & Troubleshooting
-------------------------

Getting Help
~~~~~~~~~~~~

1. **Documentation**
   - API reference
   - Configuration guides
   - Troubleshooting guides

2. **Support Channels**
   - Email: logging-support@univagateway.com
   - Phone: +1-800-LOGGING
   - Online portal

3. **Debug Information**
   - Generate diagnostic reports
   - Share configuration files
   - Provide error codes

Common Solutions
~~~~~~~~~~~~~~~~

1. **Storage Full**
   - Increase storage allocation
   - Adjust retention periods
   - Run manual rotation

2. **Performance Issues**
   - Adjust logging intervals
   - Reduce log verbosity
   - Increase buffer sizes

3. **Connectivity Issues**
   - Check network connectivity
   - Verify destination settings
   - Review firewall rules

Notes
-----

.. warning::

   **Security Logs**: Security logs have special protection and cannot be disabled without admin approval

.. warning::

   **Storage Management**: Monitor log storage regularly to prevent system issues

.. important::

   **Performance Impact**: High-frequency logging can impact system performance

.. note::

   **Configuration Backup**: Always backup logging configuration before changes

.. note::

   **Testing**: Test logging configuration in staging before production deployment

.. note::

   **Monitoring**: Set up alerts for critical logging issues

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
   * - **Log queries**
     - 100
     - GET endpoints
   * - **WebSocket connections**
     - 5 per user
     - Concurrent connections
   * - **Log streaming**
     - 1000 lines/sec
     - Per client limit
   * - **Statistics queries**
     - 30
     - Historical data

Compliance Notes
----------------

1. **Security Regulations**
   - Security logs require extended retention
   - Configuration changes must be audited
   - Access to logs must be controlled

2. **Data Protection**
   - Encrypt logs containing sensitive data
   - Secure log transmission
   - Control log exports

3. **Audit Requirements**
   - Maintain comprehensive audit trails
   - Regular security reviews
   - Compliance reporting

Versioning
----------

API version is included in the URL path (``/api/v1/``). Breaking changes will result in a new version number.

Support
-------

For API support, contact:

- **Email**: logging-support@univagateway.com
- **Documentation**: https://docs.univagateway.com/api/logging
- **Emergency**: +1-800-LOGGING (564-4644)
- **API Status**: https://status.univagateway.com/api
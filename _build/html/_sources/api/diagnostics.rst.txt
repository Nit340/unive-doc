Diagnostics & Terminal API
==========================

APIs for system monitoring, real-time terminal access, diagnostic tools, and packet capture capabilities.

Overview
--------

The Diagnostics & Terminal API provides comprehensive system monitoring, real-time terminal access, diagnostic tools, and packet capture capabilities for the Univa Gateway platform. This system enables administrators to monitor system health, execute commands, analyze network traffic, and troubleshoot issues in real-time.

Base URL
--------

::

   https://univa-gateway/api/v1/diagnostics

Authentication
--------------

All endpoints require JWT authentication via the ``Authorization`` header:

.. code-block:: http

   Authorization: Bearer <jwt_token>

.. note::

   Terminal access and diagnostic tools require admin-level privileges.

System Logs API
---------------

Get Real-Time Logs
~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/logs/stream

   Stream real-time system logs with filtering capabilities.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **levels** (optional): Comma-separated log levels (ERROR, WARN, INFO, DEBUG, TRACE)
   * **categories** (optional): Comma-separated categories (System, Network, ModbusEngine, ACS, MQTT, CraneIQ, RF_Radio)
   * **search** (optional): Search term to filter logs
   * **limit** (optional): Maximum number of log entries (default: 1000)
   * **tail** (optional): Return only recent N lines (default: 100)
   * **since** (optional): ISO timestamp to get logs from specific time

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "logs": [
          {
            "timestamp": "2025-03-12T09:44:10Z",
            "level": "INFO",
            "category": "System",
            "message": "Boot sequence completed successfully.",
            "source": "kernel",
            "tags": ["boot", "startup"]
          },
          {
            "timestamp": "2025-03-12T09:44:11Z",
            "level": "INFO",
            "category": "NetManager",
            "message": "eth0 link up, IP 192.168.1.105",
            "tags": ["network", "interface"]
          },
          {
            "timestamp": "2025-03-12T09:44:12Z",
            "level": "INFO",
            "category": "ModbusEngine",
            "message": "Polling device ID 3, address 40001",
            "tags": ["modbus", "polling"]
          }
        ],
        "total": 1250,
        "filtered": 3,
        "has_more": true
      }

Export Logs
~~~~~~~~~~~

.. http:post:: /diagnostics/logs/export

   Export logs to downloadable format.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/logs/export HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "format": "json", // "json", "csv", "text", "archive"
        "compression": "gzip", // "none", "gzip", "zip"
        "date_from": "2025-03-10T00:00:00Z",
        "date_to": "2025-03-12T23:59:59Z",
        "filters": {
          "levels": ["ERROR", "WARN"],
          "categories": ["CraneIQ", "ACS"]
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "logs_export_20250312_143000",
        "download_url": "/api/v1/diagnostics/logs/export/download/logs_export_20250312_143000",
        "estimated_size_mb": 45
      }

Terminal API
------------

Establish SSH Session
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/terminal/sessions

   Create a new SSH terminal session.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/terminal/sessions HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "session_type": "ssh", // "ssh", "serial", "telnet"
        "host": "localhost", // or gateway IP for remote
        "port": 22,
        "username": "root",
        "timeout_seconds": 300,
        "pty_config": {
          "rows": 40,
          "cols": 120,
          "term": "xterm-256color"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "session_id": "term_session_20250312_150000",
        "status": "connected",
        "gateway_info": {
          "hostname": "univa-gw-01",
          "os": "Linux 5.10.0-21-arm64",
          "uptime": "2 days, 14:12"
        },
        "websocket_url": "wss://univa-gateway/api/v1/diagnostics/terminal/ws/term_session_20250312_150000",
        "session_expires": "2025-03-12T15:30:00Z"
      }

Execute Command Directly
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/terminal/execute

   Execute a single command without interactive session.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/terminal/execute HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "command": "rf-test-tool --scan",
        "working_directory": "/root",
        "timeout_seconds": 30,
        "capture_output": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "execution_id": "exec_20250312_150230",
        "command": "rf-test-tool --scan",
        "exit_code": 0,
        "output": "Scanning 433MHz band...\n[+] Channel 1 (433.100): RSSI -92 dBm (Clear)\n[+] Channel 2 (433.200): RSSI -88 dBm (Clear)\n[!] Channel 3 (433.300): RSSI -45 dBm (BUSY)\nScan complete.",
        "duration_ms": 2450,
        "start_time": "2025-03-12T15:02:30Z",
        "end_time": "2025-03-12T15:02:32Z"
      }

Command History
~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/terminal/history

   Get command execution history.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **limit** (optional): Number of commands to return (default: 50)
   * **user** (optional): Filter by user
   * **date_from** (optional): Filter commands after this date
   * **status** (optional): Filter by exit code (0=success, other=error)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "history": [
          {
            "command_id": "cmd_20250312_145500",
            "command": "uname -a",
            "user": "admin",
            "exit_code": 0,
            "duration_ms": 120,
            "timestamp": "2025-03-12T14:55:00Z",
            "output_summary": "Linux univa-gw 5.10.0-21-arm64..."
          },
          {
            "command_id": "cmd_20250312_145200",
            "command": "top -b -n 1 | head -n 5",
            "user": "admin",
            "exit_code": 0,
            "duration_ms": 250,
            "timestamp": "2025-03-12T14:52:00Z",
            "output_summary": "top - 09:45:02 up 2 days..."
          }
        ],
        "total_commands": 2450,
        "success_rate": 98.5
      }

System Monitoring API
---------------------

Get System Metrics
~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/metrics/system

   Retrieve real-time system resource metrics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **interval** (optional): Time interval in seconds for data points (default: 1)
   * **duration** (optional): Duration to fetch in seconds (default: 60)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "metrics": {
          "cpu": {
            "usage_percent": 34.2,
            "cores": 4,
            "load_average": [0.45, 0.32, 0.28],
            "process_count": 112
          },
          "memory": {
            "total_mb": 3920,
            "used_mb": 824.5,
            "free_mb": 1452.2,
            "usage_percent": 68.5
          },
          "storage": {
            "root": {
              "total_mb": 29000,
              "used_mb": 12000,
              "free_mb": 16000,
              "usage_percent": 44
            },
            "boot": {
              "total_mb": 253,
              "used_mb": 52,
              "free_mb": 201,
              "usage_percent": 21
            }
          },
          "network": {
            "interfaces": {
              "eth0": {
                "rx_bytes_sec": 245000,
                "tx_bytes_sec": 123000,
                "rx_packets_sec": 450,
                "tx_packets_sec": 320,
                "ip_address": "192.168.1.105"
              }
            }
          }
        },
        "timestamp": "2025-03-12T15:00:02Z"
      }

RF Signal Monitoring
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/metrics/rf

   Monitor RF signal quality and spectrum.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **frequency_range** (optional): Comma-separated min,max in MHz (default: "433,434")
   * **scan** (optional): Perform fresh scan (true/false)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rf_metrics": {
          "current_frequency_mhz": 433.3,
          "signal_strength_dbm": -45,
          "signal_quality": "BUSY",
          "noise_floor_dbm": -92,
          "scan_results": [
            {
              "frequency_mhz": 433.1,
              "rssi_dbm": -92,
              "status": "Clear",
              "bandwidth_khz": 200
            },
            {
              "frequency_mhz": 433.2,
              "rssi_dbm": -88,
              "status": "Clear",
              "bandwidth_khz": 200
            },
            {
              "frequency_mhz": 433.3,
              "rssi_dbm": -45,
              "status": "BUSY",
              "bandwidth_khz": 200,
              "occupied": true
            }
          ],
          "last_scan": "2025-03-12T15:00:00Z"
        },
        "health": {
          "status": "healthy",
          "message": "RF system operating normally"
        }
      }

Diagnostic Tools API
--------------------

Packet Capture
~~~~~~~~~~~~~~

.. http:post:: /diagnostics/tools/packet-capture/start

   Start network packet capture.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/tools/packet-capture/start HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "interface": "eth0", // "eth0", "wlan0", "can0", "rf0"
        "duration_seconds": 30,
        "buffer_size_mb": 50,
        "filters": {
          "protocols": ["modbus", "mqtt"],
          "ports": [502, 1883],
          "promiscuous": true
        },
        "output_format": "pcap" // "pcap", "json", "text"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "capture_id": "pcap_20250312_151500",
        "status": "running",
        "interface": "eth0",
        "estimated_size_mb": 120,
        "stop_url": "/api/v1/diagnostics/tools/packet-capture/stop/pcap_20250312_151500",
        "download_url": "/api/v1/diagnostics/tools/packet-capture/download/pcap_20250312_151500"
      }

Stop Packet Capture
~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/tools/packet-capture/stop/{capture_id}

   Stop ongoing packet capture.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Path Parameters**:

   * **capture_id** (required): ID of the capture operation

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "capture_id": "pcap_20250312_151500",
        "status": "stopped",
        "duration_seconds": 28,
        "packets_captured": 12450,
        "file_size_mb": 12.5,
        "download_url": "/api/v1/diagnostics/tools/packet-capture/download/pcap_20250312_151500"
      }

Serial Port Monitor
~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/tools/serial-monitor

   Monitor and interact with serial ports.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/tools/serial-monitor HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "port": "/dev/ttyUSB0",
        "baud_rate": 115200,
        "data_bits": 8,
        "parity": "none", // "none", "odd", "even"
        "stop_bits": 1,
        "flow_control": "none" // "none", "rts_cts", "xon_xoff"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "session_id": "serial_20250312_152000",
        "status": "connected",
        "port": "/dev/ttyUSB0",
        "baud_rate": 115200,
        "websocket_url": "wss://univa-gateway/api/v1/diagnostics/tools/serial/ws/serial_20250312_152000",
        "log_file": "/var/log/serial_monitor_20250312_152000.log"
      }

RF Spectrum Scanner
~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/tools/rf-scanner/scan

   Perform RF spectrum analysis.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/tools/rf-scanner/scan HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "frequency_range_mhz": [433.0, 434.0],
        "step_size_mhz": 0.2,
        "sensitivity_dbm": -80,
        "duration_seconds": 10,
        "output_format": "json" // "json", "csv", "text"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "scan_id": "rfscan_20250312_153000",
        "status": "scanning",
        "parameters": {
          "range": "433.0-434.0 MHz",
          "steps": 10,
          "duration": "10 seconds"
        },
        "results_url": "/api/v1/diagnostics/tools/rf-scanner/results/rfscan_20250312_153000"
      }

Get RF Scan Results
~~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/tools/rf-scanner/results/{scan_id}

   Retrieve completed RF scan results.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **scan_id** (required): ID of the scan operation

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "scan_id": "rfscan_20250312_153000",
        "status": "completed",
        "scan_time": "2025-03-12T15:30:00Z",
        "results": [
          {
            "frequency_mhz": 433.1,
            "rssi_dbm": -92.5,
            "noise_floor_dbm": -95.2,
            "snr_db": 2.7,
            "status": "Clear",
            "bandwidth_khz": 200
          },
          {
            "frequency_mhz": 433.3,
            "rssi_dbm": -45.2,
            "noise_floor_dbm": -95.1,
            "snr_db": 49.9,
            "status": "BUSY",
            "bandwidth_khz": 200,
            "occupied": true
          }
        ],
        "summary": {
          "total_channels": 10,
          "occupied_channels": 1,
          "max_signal_dbm": -45.2,
          "min_signal_dbm": -95.1,
          "average_noise_dbm": -95.2
        }
      }

Quick Diagnostics API
---------------------

Run System Diagnostics
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/run

   Execute comprehensive system diagnostics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/run HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "checks": [
          "hardware",
          "network",
          "rf_system"
        ],
        "verbose": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "diagnostic_id": "diag_full_20250312_154500",
        "status": "running",
        "estimated_duration_seconds": 45,
        "checks": [
          {
            "name": "hardware_diagnostic",
            "status": "in_progress",
            "progress": 30
          },
          {
            "name": "network_diagnostic",
            "status": "pending",
            "progress": 0
          }
        ],
        "results_url": "/api/v1/diagnostics/results/diag_full_20250312_154500"
      }

Get Diagnostic Results
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/results/{diagnostic_id}

   Retrieve diagnostic check results.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Path Parameters**:

   * **diagnostic_id** (required): ID of the diagnostic operation

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "diagnostic_id": "diag_full_20250312_154500",
        "status": "completed",
        "timestamp": "2025-03-12T15:45:30Z",
        "duration_seconds": 42,
        "results": {
          "hardware": {
            "status": "healthy",
            "checks": [
              {"name": "cpu_temperature", "status": "pass", "value": "45°C", "threshold": "80°C"},
              {"name": "memory_integrity", "status": "pass", "value": "OK"},
              {"name": "storage_health", "status": "warning", "value": "85% used", "threshold": "90%"}
            ]
          },
          "network": {
            "status": "healthy",
            "checks": [
              {"name": "internet_connectivity", "status": "pass", "value": "Connected", "latency_ms": 45},
              {"name": "dns_resolution", "status": "pass", "value": "OK", "response_time_ms": 12}
            ]
          }
        },
        "summary": {
          "total_checks": 24,
          "passed": 22,
          "warnings": 2,
          "failed": 0,
          "overall_status": "healthy"
        }
      }

Configuration Management API
----------------------------

Get Diagnostics Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /diagnostics/config

   Retrieve current diagnostics configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "config": {
          "log_settings": {
            "retention_days": 30,
            "max_size_mb": 1024,
            "levels_enabled": ["ERROR", "WARN", "INFO", "DEBUG"],
            "categories_enabled": ["System", "Network", "ModbusEngine", "ACS", "CraneIQ"]
          },
          "terminal_settings": {
            "session_timeout_minutes": 30,
            "max_sessions": 3,
            "history_size": 1000
          },
          "monitoring_settings": {
            "metrics_interval_seconds": 1,
            "rf_scan_interval_minutes": 5
          },
          "capture_settings": {
            "max_capture_size_mb": 100,
            "default_duration_seconds": 30,
            "allowed_interfaces": ["eth0", "wlan0"]
          }
        }
      }

Update Diagnostics Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /diagnostics/config

   Update diagnostics system configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /diagnostics/config HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "log_settings": {
          "retention_days": 45,
          "levels_enabled": ["ERROR", "WARN", "INFO"]
        },
        "terminal_settings": {
          "session_timeout_minutes": 45,
          "max_sessions": 5
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Diagnostics configuration updated successfully",
        "requires_restart": false
      }

Export & Backup API
-------------------

Export All Diagnostics Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/export/all

   Export comprehensive diagnostics data package.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/export/all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "include_logs": true,
        "include_metrics": true,
        "include_captures": true,
        "include_diagnostics": true,
        "date_from": "2025-03-10T00:00:00Z",
        "date_to": "2025-03-12T23:59:59Z",
        "compression": "zip"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "export_full_20250312_160000",
        "status": "processing",
        "estimated_size_mb": 320,
        "contents": {
          "logs_mb": 45,
          "metrics_mb": 120,
          "captures_mb": 150,
          "diagnostics_mb": 5
        },
        "download_url": "/api/v1/diagnostics/export/download/export_full_20250312_160000"
      }

Clear Diagnostic Data
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/clear

   Clear diagnostic data and logs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/clear HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "clear_logs": true,
        "clear_captures": true,
        "older_than_days": 7,
        "reason": "Storage cleanup"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "operation_id": "clear_20250312_163000",
        "status": "completed",
        "cleared_data": {
          "logs_mb": 125,
          "captures_mb": 320,
          "files_deleted": 45
        },
        "remaining_data_mb": 45,
        "freed_space_mb": 445
      }

System Control API
------------------

Reboot Gateway
~~~~~~~~~~~~~~

.. http:post:: /diagnostics/system/reboot

   Reboot the Univa Gateway.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/system/reboot HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "delay_seconds": 10,
        "reason": "Diagnostics maintenance"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "System reboot scheduled",
        "reboot_time": "2025-03-12T16:35:10Z",
        "estimated_downtime": "00:01:30"
      }

Reset to Factory Defaults
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /diagnostics/system/reset-defaults

   Reset diagnostics configuration to factory defaults.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /diagnostics/system/reset-defaults HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "preserve_logs": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Diagnostics system reset to factory defaults",
        "reset_components": [
          "terminal_settings",
          "monitoring_settings",
          "capture_settings"
        ],
        "preserved_components": [
          "logs",
          "diagnostic_history"
        ]
      }

WebSocket API
-------------

Log Stream WebSocket
~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/logs/ws?token=<jwt_token>

Subscribe Message
^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "action": "subscribe",
     "filters": {
       "levels": ["ERROR", "WARN"],
       "categories": ["CraneIQ", "ACS"]
     }
   }

Log Message
^^^^^^^^^^^

.. code-block:: json

   {
     "event": "log_entry",
     "data": {
       "timestamp": "2025-03-12T15:00:00Z",
       "level": "ERROR",
       "category": "CraneIQ",
       "message": "Brake slip detected"
     }
   }

Terminal WebSocket
~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/terminal/ws/{session_id}

Send Command
^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "command",
     "command": "uname -a",
     "working_directory": "/root"
   }

Terminal Output
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "output",
     "data": "Linux univa-gw 5.10.0-21-arm64...",
     "timestamp": "2025-03-12T15:01:00Z"
   }

Exit Notification
^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "exit",
     "exit_code": 0,
     "duration_ms": 150
   }

Metrics WebSocket
~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/metrics/ws?token=<jwt_token>

Subscribe Message
^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "action": "subscribe",
     "metrics": ["cpu", "memory", "network"],
     "interval_ms": 1000
   }

Metrics Update
^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "metrics_update",
     "timestamp": "2025-03-12T15:01:00Z",
     "data": {
       "cpu_usage": 35.2,
       "memory_usage": 68.7,
       "network_rx": 245120,
       "network_tx": 123450
     }
   }

General WebSocket Events
~~~~~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/ws?token=<jwt_token>

Event Types
^^^^^^^^^^^

**Capture Progress**:

.. code-block:: json

   {
     "event": "capture_progress",
     "data": {
       "capture_id": "pcap_20250312_151500",
       "packets_captured": 12450,
       "size_mb": 12.5,
       "progress_percent": 65
     }
   }

**Diagnostic Progress**:

.. code-block:: json

   {
     "event": "diagnostic_progress",
     "data": {
       "diagnostic_id": "diag_full_20250312_154500",
       "current_check": "network_diagnostic",
       "progress_percent": 45,
       "estimated_remaining_seconds": 25
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
     "timestamp": "2025-03-12T16:00:00Z",
     "request_id": "req_abc123def456"
   }

Common Error Codes
~~~~~~~~~~~~~~~~~~

.. list-table:: Diagnostics Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **TERMINAL_SESSION_LIMIT**
     - Maximum terminal sessions reached
   * - **INVALID_COMMAND**
     - Command not allowed or invalid
   * - **CAPTURE_ACTIVE**
     - Packet capture already in progress
   * - **INSUFFICIENT_STORAGE**
     - Not enough storage for operation
   * - **PERMISSION_DENIED**
     - User lacks required permissions
   * - **SESSION_EXPIRED**
     - Terminal session has expired
   * - **INTERFACE_UNAVAILABLE**
     - Network interface not available
   * - **DIAGNOSTIC_RUNNING**
     - Diagnostic already in progress
   * - **RF_SCANNER_BUSY**
     - RF spectrum scanner already in use
   * - **SERIAL_PORT_BUSY**
     - Serial port already in use
   * - **EXPORT_TOO_LARGE**
     - Export exceeds maximum allowed size
   * - **REBOOT_SCHEDULED**
     - System reboot already scheduled

Examples
--------

Python - System Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   import json
   
   # Configuration
   BASE_URL = "https://univa-gateway/api/v1/diagnostics"
   TOKEN = "your_jwt_token_here"
   
   headers = {
       "Authorization": f"Bearer {TOKEN}",
       "Content-Type": "application/json"
   }
   
   # Get system metrics
   response = requests.get(
       f"{BASE_URL}/metrics/system",
       headers=headers,
       params={"interval": 5, "duration": 60}
   )
   
   if response.status_code == 200:
       metrics = response.json()
       print(f"CPU Usage: {metrics['metrics']['cpu']['usage_percent']}%")
       print(f"Memory Usage: {metrics['metrics']['memory']['usage_percent']}%")
   
   # Execute terminal command
   command_data = {
       "command": "rf-test-tool --scan",
       "working_directory": "/opt/univa/rf-tools",
       "timeout_seconds": 30,
       "capture_output": True
   }
   
   response = requests.post(
       f"{BASE_URL}/terminal/execute",
       json=command_data,
       headers=headers
   )
   
   if response.status_code == 200:
       result = response.json()
       print(f"Command Output:\n{result['output']}")
   
   # Start packet capture
   capture_data = {
       "interface": "eth0",
       "duration_seconds": 30,
       "buffer_size_mb": 50,
       "filters": {
           "protocols": ["modbus"],
           "ports": [502],
           "promiscuous": True
       },
       "output_format": "pcap"
   }
   
   response = requests.post(
       f"{BASE_URL}/tools/packet-capture/start",
       json=capture_data,
       headers=headers
   )
   
   if response.status_code == 200:
       capture = response.json()
       print(f"Capture started: {capture['capture_id']}")

JavaScript - Real-time Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket connection for real-time logs
   const logSocket = new WebSocket(
       `wss://univa-gateway/api/v1/diagnostics/logs/ws?token=${token}`
   );
   
   logSocket.onopen = function() {
       console.log('Connected to log stream');
       
       // Subscribe to error and warning logs
       logSocket.send(JSON.stringify({
           action: 'subscribe',
           filters: {
               levels: ['ERROR', 'WARN'],
               categories: ['CraneIQ', 'ACS', 'RF_Radio']
           }
       }));
   };
   
   logSocket.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       if (message.event === 'log_entry') {
           displayLog(message.data);
       }
   };
   
   // Real-time metrics
   const metricsSocket = new WebSocket(
       `wss://univa-gateway/api/v1/diagnostics/metrics/ws?token=${token}`
   );
   
   metricsSocket.onopen = function() {
       metricsSocket.send(JSON.stringify({
           action: 'subscribe',
           metrics: ['cpu', 'memory', 'network'],
           interval_ms: 1000
       }));
   };
   
   metricsSocket.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       if (message.type === 'metrics_update') {
           updateDashboard(message.data);
       }
   };

Python - Complete Diagnostics Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   import time
   
   class DiagnosticsClient:
       def __init__(self, base_url, token):
           self.base_url = base_url
           self.headers = {
               "Authorization": f"Bearer {token}",
               "Content-Type": "application/json"
           }
       
       def run_system_diagnostics(self):
           """Run comprehensive system diagnostics"""
           data = {
               "checks": ["hardware", "network", "rf_system"],
               "verbose": True
           }
           
           response = requests.post(
               f"{self.base_url}/run",
               json=data,
               headers=self.headers
           )
           
           if response.status_code == 200:
               result = response.json()
               diagnostic_id = result['diagnostic_id']
               
               # Wait for completion
               return self.wait_for_diagnostics(diagnostic_id)
           
           return None
       
       def wait_for_diagnostics(self, diagnostic_id):
           """Poll for diagnostic results"""
           max_attempts = 60  # 30 seconds max
           
           for attempt in range(max_attempts):
               response = requests.get(
                   f"{self.base_url}/results/{diagnostic_id}",
                   headers=self.headers
               )
               
               if response.status_code == 200:
                   result = response.json()
                   
                   if result['status'] == 'completed':
                       return result
                   
               time.sleep(0.5)  # Wait 500ms between checks
           
           return None
       
       def export_diagnostics_data(self, days=7):
           """Export recent diagnostics data"""
           import datetime
           
           end_date = datetime.datetime.utcnow()
           start_date = end_date - datetime.timedelta(days=days)
           
           data = {
               "include_logs": True,
               "include_metrics": True,
               "include_diagnostics": True,
               "date_from": start_date.isoformat() + "Z",
               "date_to": end_date.isoformat() + "Z",
               "compression": "zip"
           }
           
           response = requests.post(
               f"{self.base_url}/export/all",
               json=data,
               headers=self.headers
           )
           
           return response.json()
   
   # Usage
   client = DiagnosticsClient(
       base_url="https://univa-gateway/api/v1/diagnostics",
       token="your_token_here"
   )
   
   # Run diagnostics
   results = client.run_system_diagnostics()
   
   if results:
       print(f"Overall Status: {results['summary']['overall_status']}")
       print(f"Checks Passed: {results['summary']['passed']}/{results['summary']['total_checks']}")
   
   # Export data
   export_result = client.export_diagnostics_data(days=30)
   print(f"Export ID: {export_result['export_id']}")
   print(f"Download URL: {export_result['download_url']}")

JavaScript - Dashboard Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   class DiagnosticsDashboard {
       constructor(baseUrl, token) {
           this.baseUrl = baseUrl;
           this.token = token;
           this.headers = {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           };
       }
       
       async getSystemMetrics() {
           const response = await fetch(`${this.baseUrl}/metrics/system`, {
               headers: this.headers
           });
           
           if (!response.ok) {
               throw new Error(`Failed to fetch metrics: ${response.status}`);
           }
           
           return await response.json();
       }
       
       async executeCommand(command, options = {}) {
           const data = {
               command: command,
               working_directory: options.workingDirectory || '/root',
               timeout_seconds: options.timeout || 30,
               capture_output: true
           };
           
           const response = await fetch(`${this.baseUrl}/terminal/execute`, {
               method: 'POST',
               headers: this.headers,
               body: JSON.stringify(data)
           });
           
           if (!response.ok) {
               throw new Error(`Command execution failed: ${response.status}`);
           }
           
           return await response.json();
       }
       
       async startPacketCapture(interface, options = {}) {
           const data = {
               interface: interface,
               duration_seconds: options.duration || 30,
               buffer_size_mb: options.bufferSize || 50,
               filters: {
                   protocols: options.protocols || ['modbus', 'mqtt'],
                   ports: options.ports || [502, 1883],
                   promiscuous: options.promiscuous || true
               },
               output_format: options.format || 'pcap'
           };
           
           const response = await fetch(`${this.baseUrl}/tools/packet-capture/start`, {
               method: 'POST',
               headers: this.headers,
               body: JSON.stringify(data)
           });
           
           if (!response.ok) {
               throw new Error(`Capture failed to start: ${response.status}`);
           }
           
           return await response.json();
       }
       
       async getLogs(filterOptions = {}) {
           const params = new URLSearchParams();
           
           if (filterOptions.levels) {
               params.append('levels', filterOptions.levels.join(','));
           }
           
           if (filterOptions.categories) {
               params.append('categories', filterOptions.categories.join(','));
           }
           
           if (filterOptions.search) {
               params.append('search', filterOptions.search);
           }
           
           if (filterOptions.limit) {
               params.append('limit', filterOptions.limit);
           }
           
           const response = await fetch(
               `${this.baseUrl}/logs/stream?${params.toString()}`,
               { headers: this.headers }
           );
           
           if (!response.ok) {
               throw new Error(`Failed to fetch logs: ${response.status}`);
           }
           
           return await response.json();
       }
   }
   
   // Usage in dashboard
   const dashboard = new DiagnosticsDashboard(
       'https://univa-gateway/api/v1/diagnostics',
       localStorage.getItem('token')
   );
   
   // Update metrics every 5 seconds
   setInterval(async () => {
       try {
           const metrics = await dashboard.getSystemMetrics();
           updateMetricsUI(metrics);
       } catch (error) {
           console.error('Failed to update metrics:', error);
       }
   }, 5000);

Best Practices
--------------

System Monitoring
~~~~~~~~~~~~~~~~~

1. **Real-time Monitoring**
   - Use WebSockets for live metrics updates
   - Set appropriate update intervals (1-5 seconds)
   - Implement connection recovery

2. **Log Management**
   - Apply filters to reduce network traffic
   - Use search parameters for specific issues
   - Export logs regularly for archiving

3. **Terminal Operations**
   - Always specify timeout for commands
   - Use working directory appropriately
   - Review command history regularly

Diagnostic Tools
~~~~~~~~~~~~~~~~

1. **Packet Capture**
   - Start with short durations (10-30 seconds)
   - Use protocol filters to reduce capture size
   - Stop captures when troubleshooting is complete

2. **RF Scanning**
   - Scan during maintenance windows
   - Document interference sources
   - Compare scans over time for trends

3. **Serial Monitoring**
   - Verify port settings before connecting
   - Monitor for communication errors
   - Log serial traffic for debugging

Security Considerations
-----------------------

Authentication
~~~~~~~~~~~~~~

- **Terminal Access**: Requires admin privileges
- **Command Execution**: Commands are logged and audited
- **Data Export**: Export data is secured and encrypted

Data Protection
~~~~~~~~~~~~~~~

- **Packet Capture**: May contain sensitive data - handle with care
- **Log Export**: Password protect archives containing sensitive information
- **Session Management**: Terminal sessions expire automatically

Rate Limiting
-------------

.. list-table:: Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Endpoint Type
     - Requests/Minute
     - Notes
   * - **Log streaming**
     - 1000
     - Real-time log access
   * - **Terminal commands**
     - 60
     - Per session limit
   * - **Packet capture**
     - 1 concurrent
     - One capture at a time
   * - **Diagnostic runs**
     - 1 concurrent
     - One diagnostic at a time
   * - **WebSocket connections**
     - 5 per user
     - Concurrent connections

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
     - 2GB for diagnostics data
   * - **Memory**
     - 1GB for monitoring tools
   * - **CPU**
     - Multi-core for parallel diagnostics
   * - **Network**
     - Stable for WebSocket connections

Performance Impact
~~~~~~~~~~~~~~~~~~

- **Metrics Collection**: <5% CPU usage
- **Log Streaming**: <10% network bandwidth
- **Packet Capture**: 10-100 MB/min depending on traffic
- **RF Scanning**: <15% CPU during active scans

Troubleshooting
---------------

Common Issues
~~~~~~~~~~~~~

1. **Terminal Connection Failed**
   - Verify user has admin privileges
   - Check SSH service is running
   - Confirm network connectivity

2. **Logs Not Streaming**
   - Check WebSocket connection
   - Verify filters are not too restrictive
   - Check log service status

3. **Metrics Not Updating**
   - Verify monitoring service is running
   - Check system resources
   - Review error logs

Getting Help
~~~~~~~~~~~~

- **API Documentation**: https://docs.univagateway.com/api/diagnostics
- **Support Email**: diagnostics-support@univagateway.com
- **Emergency Contact**: +1-800-DIAGNOSTIC (342-4667)
- **Status Page**: https://status.univagateway.com

Versioning
----------

API version is included in the URL path (``/api/v1/``). Breaking changes will result in a new version number.

Changelog
---------

**v1.0.0** (2025-03-12)
~~~~~~~~~~~~~~~~~~~~~~~

- Initial release of Diagnostics & Terminal API
- Complete system monitoring capabilities
- Real-time terminal access
- Comprehensive diagnostic tools
- WebSocket support for live updates
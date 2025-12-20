Diagnostics & Terminal API
========================================

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

   Terminal access requires admin-level privileges.

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
            "process_id": 1234,
            "tags": ["boot", "startup"]
          },
          {
            "timestamp": "2025-03-12T09:44:11Z",
            "level": "INFO",
            "category": "NetManager",
            "message": "eth0 link up, IP 192.168.1.105",
            "source": "networkd",
            "process_id": 2345,
            "tags": ["network", "interface"]
          },
          {
            "timestamp": "2025-03-12T09:44:12Z",
            "level": "INFO",
            "category": "ModbusEngine",
            "message": "Polling device ID 3, address 40001",
            "source": "modbus-daemon",
            "process_id": 3456,
            "tags": ["modbus", "polling"]
          }
        ],
        "total": 1250,
        "filtered": 3,
        "has_more": true
      }

WebSocket Log Stream
~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/logs/ws?token=<jwt_token>

Client Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "action": "subscribe",
     "filters": {
       "levels": ["ERROR", "WARN"],
       "categories": ["CraneIQ", "ACS"]
     }
   }

Server Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "log_entry",
     "data": {
       "timestamp": "2025-03-12T09:44:14Z",
       "level": "ERROR",
       "category": "CraneIQ",
       "message": "Brake slip detected on Hoist Motor",
       "source": "crane-safety",
       "process_id": 4567
     }
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
        },
        "include_metadata": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_id": "logs_export_20250312_143000",
        "status": "processing",
        "estimated_size_mb": 45,
        "estimated_records": 125000,
        "download_url": "/api/v1/diagnostics/logs/export/download/logs_export_20250312_143000",
        "estimated_completion": "2025-03-12T14:35:00Z"
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

WebSocket Terminal Access
~~~~~~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/terminal/ws/{session_id}

Client Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "command",
     "command": "uname -a",
     "working_directory": "/root"
   }

Server Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "type": "output",
     "data": "Linux univa-gw 5.10.0-21-arm64 #1 SMP Debian 5.10.162-1 (2023-01-21) aarch64 GNU/Linux\n",
     "timestamp": "2025-03-12T15:01:00Z"
   }

.. code-block:: json

   {
     "type": "exit",
     "exit_code": 0,
     "duration_ms": 150
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
        "capture_output": true,
        "environment": {
          "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        }
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
            "process_count": 112,
            "history": [
              {"timestamp": "2025-03-12T15:00:00Z", "value": 32.1},
              {"timestamp": "2025-03-12T15:00:01Z", "value": 34.5},
              {"timestamp": "2025-03-12T15:00:02Z", "value": 31.8}
            ]
          },
          "memory": {
            "total_mb": 3920,
            "used_mb": 824.5,
            "free_mb": 1452.2,
            "buffer_cache_mb": 1643.3,
            "usage_percent": 68.5,
            "swap_total_mb": 0,
            "swap_used_mb": 0
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

WebSocket Metrics Stream
~~~~~~~~~~~~~~~~~~~~~~~~

Connection Endpoint
^^^^^^^^^^^^^^^^^^^

::

   wss://univa-gateway/api/v1/diagnostics/metrics/ws

Client Messages
^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "action": "subscribe",
     "metrics": ["cpu", "memory", "network"],
     "interval_ms": 1000
   }

Server Messages
^^^^^^^^^^^^^^^

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
          "transmitter_power_dbm": 20,
          "receiver_sensitivity_dbm": -120,
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
          "ip_addresses": [],
          "promiscuous": true
        },
        "output_format": "pcap", // "pcap", "json", "text"
        "store_to_file": true
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
        "websocket_url": "wss://univa-gateway/api/v1/diagnostics/tools/packet-capture/ws/pcap_20250312_151500",
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
        "flow_control": "none", // "none", "rts_cts", "xon_xoff"
        "mode": "monitor", // "monitor", "interactive", "sniffer"
        "log_to_file": true
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
        "step_size_mhz": 0.1,
        "sensitivity_dbm": -80,
        "duration_seconds": 10,
        "output_format": "json", // "json", "csv", "text"
        "visualization": true,
        "detect_modulation": true
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
        "results_url": "/api/v1/diagnostics/tools/rf-scanner/results/rfscan_20250312_153000",
        "estimated_completion": "2025-03-12T15:30:10Z"
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
            "modulation": "unknown",
            "bandwidth_khz": 200,
            "peak_power_dbm": -92.5
          },
          {
            "frequency_mhz": 433.3,
            "rssi_dbm": -45.2,
            "noise_floor_dbm": -95.1,
            "snr_db": 49.9,
            "status": "BUSY",
            "modulation": "FSK",
            "bandwidth_khz": 200,
            "peak_power_dbm": -45.2,
            "occupied": true,
            "signal_source": "known_transmitter_01"
          }
        ],
        "summary": {
          "total_channels": 10,
          "occupied_channels": 1,
          "max_signal_dbm": -45.2,
          "min_signal_dbm": -95.1,
          "average_noise_dbm": -95.2
        },
        "visualization_data": {
          "frequencies": [433.1, 433.2, 433.3, 433.4, 433.5],
          "rssi_values": [-92.5, -88.3, -45.2, -78.1, -85.7]
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
          "rf_system",
          "modbus",
          "storage",
          "security"
        ],
        "verbose": true,
        "fix_issues": false,
        "generate_report": true
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
        "results_url": "/api/v1/diagnostics/results/diag_full_20250312_154500",
        "websocket_url": "wss://univa-gateway/api/v1/diagnostics/ws/diag_full_20250312_154500"
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
              {"name": "memory_integrity", "status": "pass", "value": "OK", "threshold": "N/A"},
              {"name": "storage_health", "status": "warning", "value": "85% used", "threshold": "90%", "recommendation": "Consider cleanup"}
            ]
          },
          "network": {
            "status": "healthy",
            "checks": [
              {"name": "internet_connectivity", "status": "pass", "value": "Connected", "latency_ms": 45},
              {"name": "dns_resolution", "status": "pass", "value": "OK", "response_time_ms": 12},
              {"name": "firewall_status", "status": "pass", "value": "Active", "rules": 24}
            ]
          },
          "rf_system": {
            "status": "warning",
            "checks": [
              {"name": "rf_radio_health", "status": "pass", "value": "Operational"},
              {"name": "signal_strength", "status": "warning", "value": "-45 dBm", "threshold": "-65 dBm", "recommendation": "High noise on 433.3MHz"},
              {"name": "antenna_connection", "status": "pass", "value": "OK", "vswr": 1.2}
            ]
          }
        },
        "summary": {
          "total_checks": 24,
          "passed": 22,
          "warnings": 2,
          "failed": 0,
          "overall_status": "healthy",
          "recommendations": [
            "Monitor storage usage (currently 85%)",
            "Investigate RF noise on 433.3MHz"
          ]
        },
        "report_url": "/api/v1/diagnostics/report/diag_full_20250312_154500.pdf"
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
            "compression": "daily",
            "levels_enabled": ["ERROR", "WARN", "INFO", "DEBUG"],
            "categories_enabled": ["System", "Network", "ModbusEngine", "ACS", "CraneIQ"]
          },
          "terminal_settings": {
            "session_timeout_minutes": 30,
            "max_sessions": 3,
            "allowed_commands": ["*"],
            "history_size": 1000
          },
          "monitoring_settings": {
            "metrics_interval_seconds": 1,
            "rf_scan_interval_minutes": 5,
            "alert_thresholds": {
              "cpu_percent": 90,
              "memory_percent": 85,
              "storage_percent": 90,
              "rssi_dbm": -65
            }
          },
          "capture_settings": {
            "max_capture_size_mb": 100,
            "default_duration_seconds": 30,
            "allowed_interfaces": ["eth0", "wlan0"],
            "auto_delete_days": 7
          }
        },
        "last_modified": "2025-03-12T14:30:00Z",
        "modified_by": "admin"
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
        },
        "apply_immediately": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Diagnostics configuration updated successfully",
        "applied_changes": [
          "log_settings.retention_days: 30 → 45",
          "log_settings.levels_enabled: Added DEBUG",
          "terminal_settings.session_timeout_minutes: 30 → 45",
          "terminal_settings.max_sessions: 3 → 5"
        ],
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
        "compression": "zip",
        "password_protect": true,
        "password": "optional_encryption_password"
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
        "download_url": "/api/v1/diagnostics/export/download/export_full_20250312_160000",
        "estimated_completion": "2025-03-12T16:15:00Z"
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
        "clear_metrics": false,
        "clear_captures": true,
        "clear_terminal_history": false,
        "older_than_days": 7,
        "confirmation": {
          "user": "admin",
          "timestamp": "2025-03-12T16:30:00Z",
          "reason": "Storage cleanup before maintenance"
        }
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
        "reason": "Diagnostics maintenance",
        "confirmation": {
          "user": "admin",
          "password": "encrypted_admin_password"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "System reboot scheduled",
        "reboot_time": "2025-03-12T16:35:10Z",
        "estimated_downtime": "00:01:30",
        "warning": "All active connections will be terminated"
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
        "preserve_logs": true,
        "preserve_captures": false,
        "confirmation": {
          "user": "admin",
          "emergency_code": "RESET_DIAGNOSTICS_2025"
        }
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
        ],
        "requires_restart": true,
        "restart_scheduled": "2025-03-12T16:40:00Z"
      }

Audit & Security API
--------------------

Get Audit Log
~~~~~~~~~~~~~

.. http:get:: /diagnostics/audit

   Retrieve diagnostics system audit log.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **action** (optional): Filter by action type
   * **user** (optional): Filter by user
   * **date_from** (optional): ISO timestamp
   * **limit** (optional): Number of records (default: 100)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "audit_log": [
          {
            "timestamp": "2025-03-12T15:30:00Z",
            "user": "admin",
            "action": "terminal_command",
            "details": {
              "command": "rf-test-tool --scan",
              "session_id": "term_session_20250312_150000",
              "duration_ms": 2450
            },
            "ip_address": "192.168.1.100"
          },
          {
            "timestamp": "2025-03-12T15:15:00Z",
            "user": "admin",
            "action": "packet_capture_started",
            "details": {
              "capture_id": "pcap_20250312_151500",
              "interface": "eth0",
              "duration": "30 seconds"
            },
            "ip_address": "192.168.1.100"
          },
          {
            "timestamp": "2025-03-12T14:45:00Z",
            "user": "system",
            "action": "diagnostics_completed",
            "details": {
              "diagnostic_id": "diag_full_20250312_144500",
              "status": "completed",
              "checks_passed": 22,
              "checks_failed": 0
            },
            "ip_address": "127.0.0.1"
          }
        ],
        "total_records": 2450,
        "pagination": {
          "limit": 100,
          "offset": 0,
          "has_more": true
        }
      }

WebSocket Events
----------------

Base Connection Endpoint
~~~~~~~~~~~~~~~~~~~~~~~~

::

   wss://univa-gateway/api/v1/diagnostics/ws?token=<jwt_token>

Event Types
~~~~~~~~~~~

**Log Entry**:

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

**Metrics Update**:

.. code-block:: json

   {
     "event": "metrics_update",
     "data": {
       "cpu_usage": 34.2,
       "memory_usage": 68.5,
       "network_rx_kbps": 245,
       "network_tx_kbps": 123
     }
   }

**Terminal Output**:

.. code-block:: json

   {
     "event": "terminal_output",
     "data": {
       "session_id": "term_session_20250312_150000",
       "output": "Linux univa-gw 5.10.0-21-arm64...",
       "timestamp": "2025-03-12T15:01:00Z"
     }
   }

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
   import time
   
   # Get system metrics
   headers = {"Authorization": "Bearer your_token"}
   
   metrics_response = requests.get(
       "https://univa-gateway/api/v1/diagnostics/metrics/system",
       headers=headers,
       params={"interval": 5, "duration": 60}
   )
   
   metrics = metrics_response.json()
   print(f"CPU Usage: {metrics['metrics']['cpu']['usage_percent']}%")
   print(f"Memory Usage: {metrics['metrics']['memory']['usage_percent']}%")
   print(f"Storage Usage: {metrics['metrics']['storage']['root']['usage_percent']}%")
   
   # Monitor RF signals
   rf_response = requests.get(
       "https://univa-gateway/api/v1/diagnostics/metrics/rf",
       headers=headers,
       params={"scan": True, "frequency_range": "433,434"}
   )
   
   rf_metrics = rf_response.json()
   for channel in rf_metrics['rf_metrics']['scan_results']:
       print(f"{channel['frequency_mhz']}MHz: {channel['rssi_dbm']}dBm ({channel['status']})")
   
   # Get real-time logs
   logs_response = requests.get(
       "https://univa-gateway/api/v1/diagnostics/logs/stream",
       headers=headers,
       params={"levels": "ERROR,WARN", "tail": 10}
   )
   
   logs = logs_response.json()
   for log in logs['logs']:
       print(f"{log['timestamp']} [{log['level']}] {log['category']}: {log['message']}")

Python - Terminal Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Execute command directly
   import requests
   
   execute_request = {
       "command": "rf-test-tool --scan --detailed",
       "working_directory": "/opt/univa/rf-tools",
       "timeout_seconds": 30,
       "capture_output": True
   }
   
   execute_response = requests.post(
       "https://univa-gateway/api/v1/diagnostics/terminal/execute",
       json=execute_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   result = execute_response.json()
   print(f"Command: {result['command']}")
   print(f"Exit Code: {result['exit_code']}")
   print(f"Output:\n{result['output']}")
   print(f"Duration: {result['duration_ms']}ms")
   
   # Get command history
   history_response = requests.get(
       "https://univa-gateway/api/v1/diagnostics/terminal/history",
       headers={"Authorization": "Bearer your_token"},
       params={"limit": 10, "user": "admin"}
   )
   
   history = history_response.json()
   for cmd in history['history']:
       print(f"{cmd['timestamp']}: {cmd['command']} (Exit: {cmd['exit_code']})")

Python - Diagnostic Tools
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Start packet capture
   capture_request = {
       "interface": "eth0",
       "duration_seconds": 30,
       "buffer_size_mb": 50,
       "filters": {
           "protocols": ["modbus"],
           "ports": [502],
           "promiscuous": True
       },
       "output_format": "pcap",
       "store_to_file": True
   }
   
   capture_response = requests.post(
       "https://univa-gateway/api/v1/diagnostics/tools/packet-capture/start",
       json=capture_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   capture_result = capture_response.json()
   capture_id = capture_result['capture_id']
   print(f"Capture started: {capture_id}")
   
   # Run system diagnostics
   diagnostic_request = {
       "checks": ["hardware", "network", "rf_system"],
       "verbose": True,
       "generate_report": True
   }
   
   diagnostic_response = requests.post(
       "https://univa-gateway/api/v1/diagnostics/run",
       json=diagnostic_request,
       headers={"Authorization": "Bearer your_token"}
   )
   
   diagnostic_result = diagnostic_response.json()
   diagnostic_id = diagnostic_result['diagnostic_id']
   print(f"Diagnostic started: {diagnostic_id}")
   
   # Wait and get results
   import time
   time.sleep(45)  # Wait for completion
   
   results_response = requests.get(
       f"https://univa-gateway/api/v1/diagnostics/results/{diagnostic_id}",
       headers={"Authorization": "Bearer your_token"}
   )
   
   results = results_response.json()
   print(f"Overall Status: {results['summary']['overall_status']}")
   print(f"Passed: {results['summary']['passed']}/{results['summary']['total_checks']}")

JavaScript - Real-time Diagnostics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket connection for real-time diagnostics
   const ws = new WebSocket('wss://univa-gateway/api/v1/diagnostics/ws?token=' + token);
   
   ws.onopen = function() {
       console.log('Connected to Diagnostics WebSocket');
       
       // Subscribe to log events
       ws.send(JSON.stringify({
           action: 'subscribe',
           events: ['log_entry', 'metrics_update']
       }));
   };
   
   ws.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       switch(message.event) {
           case 'log_entry':
               displayLogEntry(message.data);
               break;
               
           case 'metrics_update':
               updateMetricsDashboard(message.data);
               break;
               
           case 'terminal_output':
               displayTerminalOutput(message.data);
               break;
               
           case 'capture_progress':
               updateCaptureProgress(message.data);
               break;
               
           case 'diagnostic_progress':
               updateDiagnosticProgress(message.data);
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
   function displayLogEntry(log) {
       const logEntry = document.createElement('div');
       logEntry.className = `log-entry level-${log.level.toLowerCase()}`;
       logEntry.innerHTML = `
           <span class="timestamp">${new Date(log.timestamp).toLocaleTimeString()}</span>
           <span class="level">${log.level}</span>
           <span class="category">${log.category}</span>
           <span class="message">${log.message}</span>
       `;
       document.getElementById('log-container').appendChild(logEntry);
   }
   
   function updateMetricsDashboard(metrics) {
       document.getElementById('cpu-usage').textContent = metrics.cpu_usage.toFixed(1) + '%';
       document.getElementById('memory-usage').textContent = metrics.memory_usage.toFixed(1) + '%';
       document.getElementById('network-rx').textContent = formatBytes(metrics.network_rx);
       document.getElementById('network-tx').textContent = formatBytes(metrics.network_tx);
       
       // Update progress bars
       updateProgressBar('cpu-bar', metrics.cpu_usage);
       updateProgressBar('memory-bar', metrics.memory_usage);
   }

JavaScript - Diagnostic Dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Run system diagnostics
   async function runSystemDiagnostics() {
       const token = localStorage.getItem('token');
       
       const diagnosticConfig = {
           checks: getSelectedDiagnosticChecks(),
           verbose: true,
           generate_report: true
       };
       
       try {
           const response = await fetch('/api/v1/diagnostics/run', {
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
           showError('Failed to start diagnostic checks');
       }
   }
   
   // Start packet capture
   async function startPacketCapture(interface, filters) {
       const token = localStorage.getItem('token');
       
       const captureConfig = {
           interface: interface,
           duration_seconds: 30,
           buffer_size_mb: 50,
           filters: filters,
           output_format: 'pcap',
           store_to_file: true
       };
       
       try {
           const response = await fetch('/api/v1/diagnostics/tools/packet-capture/start', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(captureConfig)
           });
           
           const result = await response.json();
           
           if (result.success) {
               startCaptureMonitoring(result.capture_id);
               return result;
           } else {
               throw new Error(result.message || 'Capture failed to start');
           }
       } catch (error) {
           console.error('Error starting packet capture:', error);
           showError('Failed to start packet capture');
       }
   }
   
   // Export diagnostics data
   async function exportDiagnosticsData(options) {
       const token = localStorage.getItem('token');
       
       const exportConfig = {
           include_logs: options.includeLogs || true,
           include_metrics: options.includeMetrics || true,
           include_captures: options.includeCaptures || false,
           include_diagnostics: options.includeDiagnostics || true,
           date_from: options.dateFrom || null,
           date_to: options.dateTo || null,
           compression: 'zip',
           password_protect: options.passwordProtect || false,
           password: options.password || null
       };
       
       try {
           const response = await fetch('/api/v1/diagnostics/export/all', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(exportConfig)
           });
           
           const result = await response.json();
           
           if (result.success) {
               monitorExportProgress(result.export_id);
               return result;
           } else {
               throw new Error(result.message || 'Export failed to start');
           }
       } catch (error) {
           console.error('Error exporting data:', error);
           showError('Failed to export diagnostics data');
       }
   }

Best Practices
--------------

System Monitoring
~~~~~~~~~~~~~~~~~

1. **Regular Monitoring**
   - Monitor system metrics daily
   - Set up threshold alerts
   - Review logs for critical errors

2. **Performance Baseline**
   - Establish normal performance baselines
   - Track metrics over time
   - Identify trends and patterns

3. **Proactive Diagnostics**
   - Run diagnostics before issues occur
   - Schedule regular system checks
   - Maintain diagnostic history

Terminal Operations
~~~~~~~~~~~~~~~~~~~

1. **Command Safety**
   - Test commands in non-production first
   - Use dry-run options when available
   - Keep command history for audit

2. **Session Management**
   - Close unused terminal sessions
   - Set appropriate timeouts
   - Monitor active sessions

3. **Security Considerations**
   - Restrict command execution
   - Log all terminal activities
   - Use role-based access control

Diagnostic Tools
~~~~~~~~~~~~~~~~

1. **Packet Capture**
   - Use appropriate filters
   - Limit capture duration
   - Secure captured data

2. **RF Analysis**
   - Schedule scans during off-peak
   - Document interference sources
   - Maintain spectrum usage logs

3. **Serial Monitoring**
   - Configure correct port settings
   - Log serial communications
   - Monitor for anomalies

Security Considerations
-----------------------

Authentication & Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Terminal Access Control**
   - Admin-level permissions required
   - Session timeout enforcement
   - Command whitelisting

2. **Data Protection**
   - Encrypt diagnostic exports
   - Secure packet capture files
   - Protect audit logs

3. **Access Monitoring**
   - Log all diagnostic operations
   - Track user actions
   - Monitor for unauthorized access

Data Privacy
~~~~~~~~~~~~

1. **Sensitive Data Handling**
   - Anonymize logs when needed
   - Secure packet capture data
   - Protect user credentials

2. **Export Security**
   - Password protect exports
   - Encrypt sensitive data
   - Secure download links

3. **Compliance**
   - Follow data retention policies
   - Maintain audit trails
   - Secure diagnostic data

Audit & Compliance
~~~~~~~~~~~~~~~~~~

1. **Logging Requirements**
   - Terminal command logs: 90 days
   - Diagnostic operation logs: 180 days
   - Security audit logs: 365 days

2. **Reporting**
   - Generate compliance reports
   - Track security incidents
   - Document diagnostic findings

3. **Documentation**
   - Document diagnostic procedures
   - Maintain security policies
   - Update compliance documentation

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
   * - **RF Hardware**
     - Supported RF modules for scanning

Performance Considerations
--------------------------

1. **Monitoring Impact**
   - Metrics collection: <5% CPU
   - Log streaming: <10% network
   - RF scanning: <15% CPU

2. **Diagnostic Operations**
   - Full diagnostics: 30-60 seconds
   - Packet capture: 10-100 MB/min
   - RF spectrum scan: 10-30 seconds

3. **Resource Usage**
   - Terminal sessions: 50MB each
   - WebSocket connections: 5-10MB each
   - Diagnostic tools: 100-500MB

Support & Troubleshooting
-------------------------

Getting Help
~~~~~~~~~~~~

1. **Documentation**
   - API reference
   - User guides
   - Troubleshooting guides

2. **Support Channels**
   - Email: diagnostics-support@univagateway.com
   - Phone: +1-800-DIAGNOSTIC
   - Online portal

3. **Debug Information**
   - Generate diagnostic reports
   - Share system logs
   - Provide error codes

Common Solutions
~~~~~~~~~~~~~~~~

1. **Terminal Connection Issues**
   - Check user permissions
   - Verify network connectivity
   - Restart terminal service

2. **Diagnostic Failures**
   - Check system resources
   - Verify hardware connectivity
   - Review error logs

3. **Performance Issues**
   - Reduce monitoring frequency
   - Limit concurrent operations
   - Optimize capture settings

Notes
-----

.. warning::

   **Terminal Access**: Terminal operations can modify system configuration. Use with caution.

.. warning::

   **Packet Capture**: May capture sensitive network data. Follow security policies.

.. important::

   **RF Operations**: RF scanning may interfere with other RF devices. Schedule appropriately.

.. note::

   **Monitoring**: Set up alerts for critical system metrics.

.. note::

   **Export Security**: Always password protect diagnostic exports containing sensitive data.

.. note::

   **Audit Trail**: All diagnostic operations are logged for compliance and troubleshooting.

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

Compliance Notes
----------------

1. **Security Regulations**
   - Terminal access requires multi-factor authentication
   - All commands are logged for audit
   - Export data must be encrypted

2. **Data Protection**
   - Follow data retention policies
   - Secure diagnostic data storage
   - Protect user privacy

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

- **Email**: diagnostics-support@univagateway.com
- **Documentation**: https://docs.univagateway.com/api/diagnostics
- **Emergency**: +1-800-DIAGNOSTIC (342-4667)
- **API Status**: https://status.univagateway.com/api
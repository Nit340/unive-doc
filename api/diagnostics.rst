Diagnostics & Terminal API
===========================

Page Route (Frontend)
---------------------

.. http:get:: /diagnostics

   **Description**: Renders the complete diagnostics and terminal management page with all system metrics, logs, terminal access, and diagnostic tools.

   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9

   **Response**::

      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Diagnostics & Terminal - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-metrics='{...}' data-logs='[...]' data-terminal-status='{...}'>
          <!-- Diagnostics page with:
               SECTION 1: System Metrics Dashboard
               SECTION 2: Real-time Log Viewer
               SECTION 3: Terminal Access
               SECTION 4: Diagnostic Tools
               SECTION 5: Packet Capture
               SECTION 6: RF Spectrum Analysis
               FOOTER: Export, Clear, System Controls
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded system metrics and logs
   - All diagnostics information, active sessions, and tool status are embedded
   - JavaScript reads this data and renders the complete diagnostics interface
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**::

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all diagnostics operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

System Metrics (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/diagnostics/metrics

   **Description**: Get current system metrics and health status.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cpu_usage": 34.2,
        "memory_usage": 68.5,
        "disk_usage": 44.0,
        "network_traffic": {
          "rx_bytes_sec": 245000,
          "tx_bytes_sec": 123000
        },
        "system_uptime": "2 days, 14:12:05"
      }

.. http:get:: /api/diagnostics/metrics/history

   **Description**: Get historical system metrics.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Query Parameters**:
   
   * **interval** (optional): Time interval (1m, 5m, 1h, 1d)
   * **duration** (optional): Duration to fetch (1h, 6h, 24h, 7d)
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "metrics": [
          {
            "timestamp": "2024-03-20T14:00:00Z",
            "cpu": 32.1,
            "memory": 67.8,
            "network_rx": 240000,
            "network_tx": 120000
          }
        ]
      }

Real-time Logs (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/diagnostics/logs

   **Description**: Get system logs with filtering.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Query Parameters**:
   
   * **level** (optional): Log level (error, warn, info, debug)
   * **category** (optional): Log category
   * **search** (optional): Search term
   * **limit** (optional): Maximum entries (default: 1000)
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "logs": [
          {
            "timestamp": "2024-03-20T14:30:22Z",
            "level": "INFO",
            "category": "System",
            "message": "Boot sequence completed",
            "source": "kernel"
          }
        ],
        "total": 1250
      }

.. http:post:: /api/diagnostics/logs/stream

   **Description**: Start streaming real-time logs via WebSocket.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/logs/stream HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "levels": ["ERROR", "WARN", "INFO"],
        "categories": ["System", "Network"],
        "websocket_url": "wss://gateway/diagnostics/logs/ws"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "stream_started": true,
        "session_id": "log_stream_20240320_001",
        "websocket_url": "wss://gateway/diagnostics/logs/ws/log_stream_20240320_001"
      }

Terminal Access (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/diagnostics/terminal/session

   **Description**: Create a new terminal session.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/terminal/session HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "session_type": "ssh",
        "host": "localhost",
        "username": "admin",
        "timeout": 300
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "session_created": true,
        "session_id": "term_20240320_001",
        "websocket_url": "wss://gateway/diagnostics/terminal/ws/term_20240320_001",
        "expires_at": "2024-03-20T15:30:00Z"
      }

.. http:post:: /api/diagnostics/terminal/execute

   **Description**: Execute a command directly.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/terminal/execute HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "command": "df -h",
        "timeout": 30,
        "capture_output": true
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "executed": true,
        "exit_code": 0,
        "output": "Filesystem      Size  Used Avail Use% Mounted on\n/dev/root        29G   12G   16G  44% /",
        "duration_ms": 120
      }

.. http:get:: /api/diagnostics/terminal/sessions

   **Description**: Get active terminal sessions.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "sessions": [
          {
            "session_id": "term_20240320_001",
            "user": "admin",
            "started_at": "2024-03-20T14:30:00Z",
            "active": true
          }
        ]
      }

.. http:delete:: /api/diagnostics/terminal/session/{session_id}

   **Description**: Terminate a terminal session.
   
   **Path Parameters**:
   
   * **session_id** (string): Terminal session identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "terminated": true,
        "session_id": "term_20240320_001",
        "message": "Terminal session terminated"
      }

Diagnostic Tools (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/diagnostics/tools/ping

   **Description**: Perform network ping test.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/tools/ping HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "target": "8.8.8.8",
        "count": 4,
        "timeout": 5
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "completed": true,
        "target": "8.8.8.8",
        "packets_sent": 4,
        "packets_received": 4,
        "average_latency": 45.2,
        "result": "Success"
      }

.. http:post:: /api/diagnostics/tools/traceroute

   **Description**: Perform traceroute.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/tools/traceroute HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "target": "google.com",
        "max_hops": 30,
        "timeout": 10
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "completed": true,
        "hops": [
          {"hop": 1, "ip": "192.168.1.1", "latency": 1.2},
          {"hop": 2, "ip": "10.0.0.1", "latency": 5.4}
        ]
      }

.. http:post:: /api/diagnostics/tools/speedtest

   **Description**: Perform network speed test.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/tools/speedtest HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "server": "auto",
        "duration": 10
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "completed": true,
        "download_mbps": 95.4,
        "upload_mbps": 42.1,
        "latency": 24,
        "server": "New York, US"
      }

Packet Capture (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/diagnostics/packet-capture/start

   **Description**: Start packet capture on network interface.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/packet-capture/start HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "interface": "eth0",
        "duration": 30,
        "filter": "port 502 or port 1883",
        "max_size": 50
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "started": true,
        "capture_id": "pcap_20240320_001",
        "interface": "eth0",
        "estimated_size": 25,
        "stop_url": "/api/diagnostics/packet-capture/stop/pcap_20240320_001"
      }

.. http:post:: /api/diagnostics/packet-capture/stop/{capture_id}

   **Description**: Stop packet capture.
   
   **Path Parameters**:
   
   * **capture_id** (string): Capture session identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "stopped": true,
        "capture_id": "pcap_20240320_001",
        "duration": 28,
        "packets_captured": 12450,
        "file_size": 12.5,
        "download_url": "/api/diagnostics/packet-capture/download/pcap_20240320_001"
      }

.. http:get:: /api/diagnostics/packet-capture/list

   **Description**: List available packet captures.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "captures": [
          {
            "capture_id": "pcap_20240320_001",
            "interface": "eth0",
            "started": "2024-03-20T14:30:00Z",
            "duration": 30,
            "size": 12.5,
            "status": "completed"
          }
        ]
      }

.. http:get:: /api/diagnostics/packet-capture/download/{capture_id}

   **Description**: Download packet capture file.
   
   **Path Parameters**:
   
   * **capture_id** (string): Capture session identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/octet-stream
      Content-Disposition: attachment; filename="capture_20240320_001.pcap"
      
      [Binary PCAP data]

RF Spectrum Analysis (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/diagnostics/rf/scan

   **Description**: Start RF spectrum scan.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/rf/scan HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "frequency_range": [433.0, 434.0],
        "step_size": 0.1,
        "duration": 10
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "started": true,
        "scan_id": "rfscan_20240320_001",
        "frequency_range": "433.0-434.0 MHz",
        "estimated_duration": 10,
        "results_url": "/api/diagnostics/rf/results/rfscan_20240320_001"
      }

.. http:get:: /api/diagnostics/rf/results/{scan_id}

   **Description**: Get RF scan results.
   
   **Path Parameters**:
   
   * **scan_id** (string): RF scan identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "results": [
          {
            "frequency": 433.1,
            "signal_strength": -92,
            "noise_floor": -95,
            "status": "Clear"
          },
          {
            "frequency": 433.3,
            "signal_strength": -45,
            "noise_floor": -95,
            "status": "Busy"
          }
        ],
        "summary": {
          "channels_scanned": 10,
          "busy_channels": 1,
          "max_signal": -45,
          "min_signal": -95
        }
      }

.. http:get:: /api/diagnostics/rf/status

   **Description**: Get current RF system status.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "rf_system": "operational",
        "current_frequency": 433.3,
        "signal_strength": -45,
        "noise_floor": -92,
        "transmit_power": 20,
        "temperature": 42.5
      }

Footer Actions
~~~~~~~~~~~~~~

.. http:post:: /api/diagnostics/export

   **Description**: Export diagnostics data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/export HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "include_logs": true,
        "include_metrics": true,
        "include_captures": false,
        "date_range": "last_7_days",
        "format": "zip"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "exported": true,
        "export_id": "export_20240320_001",
        "download_url": "/api/diagnostics/export/download/export_20240320_001.zip",
        "size": 45.2
      }

.. http:post:: /api/diagnostics/clear

   **Description**: Clear diagnostics data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/clear HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "clear_logs": true,
        "clear_captures": true,
        "older_than": 30,
        "reason": "Storage cleanup"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cleared": true,
        "logs_cleared": 125,
        "captures_cleared": 320,
        "space_freed": 445
      }

.. http:post:: /api/diagnostics/system/reboot

   **Description**: Reboot the gateway system.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/system/reboot HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "delay": 10,
        "reason": "System maintenance"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reboot_scheduled": true,
        "reboot_time": "2024-03-20T15:30:10Z",
        "estimated_downtime": "90 seconds"
      }

.. http:post:: /api/diagnostics/system/reset

   **Description**: Reset diagnostics configuration.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/diagnostics/system/reset HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "preserve_logs": true,
        "preserve_captures": false
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "preserved_logs": true,
        "preserved_captures": false,
        "config_reset": true
      }

Route Summary
-------------

.. list-table:: Diagnostics Management Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/diagnostics`` - Main diagnostics page
     - Yes
   * - Metrics
     - GET
     - ``/api/diagnostics/metrics`` - Current system metrics
     - Yes
   * - Metrics
     - GET
     - ``/api/diagnostics/metrics/history`` - Historical metrics
     - Yes
   * - Logs
     - GET
     - ``/api/diagnostics/logs`` - Get system logs
     - Yes
   * - Logs
     - POST
     - ``/api/diagnostics/logs/stream`` - Start log streaming
     - Yes
   * - Terminal
     - POST
     - ``/api/diagnostics/terminal/session`` - Create terminal session
     - Yes
   * - Terminal
     - POST
     - ``/api/diagnostics/terminal/execute`` - Execute command
     - Yes
   * - Terminal
     - GET
     - ``/api/diagnostics/terminal/sessions`` - List sessions
     - Yes
   * - Terminal
     - DELETE
     - ``/api/diagnostics/terminal/session/{id}`` - Terminate session
     - Yes
   * - Tools
     - POST
     - ``/api/diagnostics/tools/ping`` - Ping test
     - Yes
   * - Tools
     - POST
     - ``/api/diagnostics/tools/traceroute`` - Traceroute
     - Yes
   * - Tools
     - POST
     - ``/api/diagnostics/tools/speedtest`` - Speed test
     - Yes
   * - Packet
     - POST
     - ``/api/diagnostics/packet-capture/start`` - Start capture
     - Yes
   * - Packet
     - POST
     - ``/api/diagnostics/packet-capture/stop/{id}`` - Stop capture
     - Yes
   * - Packet
     - GET
     - ``/api/diagnostics/packet-capture/list`` - List captures
     - Yes
   * - Packet
     - GET
     - ``/api/diagnostics/packet-capture/download/{id}`` - Download capture
     - Yes
   * - RF
     - POST
     - ``/api/diagnostics/rf/scan`` - Start RF scan
     - Yes
   * - RF
     - GET
     - ``/api/diagnostics/rf/results/{id}`` - Get RF results
     - Yes
   * - RF
     - GET
     - ``/api/diagnostics/rf/status`` - RF system status
     - Yes
   * - Footer
     - POST
     - ``/api/diagnostics/export`` - Export data
     - Yes
   * - Footer
     - POST
     - ``/api/diagnostics/clear`` - Clear data
     - Yes
   * - Footer
     - POST
     - ``/api/diagnostics/system/reboot`` - Reboot system
     - Yes
   * - Footer
     - POST
     - ``/api/diagnostics/system/reset`` - Reset config
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /diagnostics``
   - Server renders HTML with embedded system data
   - All 6 sections populated with current diagnostics data

2. **User views system metrics** (SECTION 1):
   - Real-time CPU, memory, disk, network usage
   - Historical graphs via ``GET /api/diagnostics/metrics/history``
   - Auto-refresh every 5 seconds

3. **User monitors logs** (SECTION 2):
   - Real-time log viewer with filtering
   - "Start Streaming" → ``POST /api/diagnostics/logs/stream``
   - Search and filter capabilities

4. **User accesses terminal** (SECTION 3):
   - "New Terminal" → ``POST /api/diagnostics/terminal/session``
   - "Execute Command" → ``POST /api/diagnostics/terminal/execute``
   - "View Active Sessions" → ``GET /api/diagnostics/terminal/sessions``
   - "Terminate Session" → ``DELETE /api/diagnostics/terminal/session/{id}``

5. **User runs diagnostic tools** (SECTION 4):
   - "Ping Test" → ``POST /api/diagnostics/tools/ping``
   - "Traceroute" → ``POST /api/diagnostics/tools/traceroute``
   - "Speed Test" → ``POST /api/diagnostics/tools/speedtest``

6. **User captures packets** (SECTION 5):
   - "Start Capture" → ``POST /api/diagnostics/packet-capture/start``
   - "Stop Capture" → ``POST /api/diagnostics/packet-capture/stop/{id}``
   - "Download Capture" → ``GET /api/diagnostics/packet-capture/download/{id}``

7. **User analyzes RF spectrum** (SECTION 6):
   - "Start RF Scan" → ``POST /api/diagnostics/rf/scan``
   - "View Results" → ``GET /api/diagnostics/rf/results/{id}``
   - "RF Status" → ``GET /api/diagnostics/rf/status``

8. **User uses footer actions**:
   - "Export Data" → ``POST /api/diagnostics/export``
   - "Clear Data" → ``POST /api/diagnostics/clear``
   - "Reboot System" → ``POST /api/diagnostics/system/reboot``
   - "Reset Config" → ``POST /api/diagnostics/system/reset``

Error Codes
-----------

.. list-table:: Diagnostics Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - TERMINAL_SESSION_LIMIT
     - Maximum terminal sessions reached
   * - INVALID_COMMAND
     - Command not allowed or invalid
   * - CAPTURE_ACTIVE
     - Packet capture already in progress
   * - INSUFFICIENT_STORAGE
     - Not enough storage for operation
   * - PERMISSION_DENIED
     - User lacks required permissions
   * - SESSION_EXPIRED
     - Terminal session has expired
   * - INTERFACE_UNAVAILABLE
     - Network interface not available
   * - RF_SCANNER_BUSY
     - RF spectrum scanner already in use
   * - EXPORT_TOO_LARGE
     - Export exceeds maximum allowed size
   * - SYSTEM_BUSY
     - System operation cannot be performed now

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Cookie: session_token=<token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages diagnostics for the current gateway specified in `X-Gateway-ID` header. Each gateway has its own independent diagnostics data.

System Requirements
-------------------

### Storage Requirements
- **Log retention**: Minimum 1GB for 30 days of logs
- **Packet captures**: 100MB per capture session
- **Metrics storage**: 500MB for 1 year of metrics
- **Total minimum**: 2GB free space recommended

### Performance Impact
- **Log streaming**: <5% CPU usage
- **Packet capture**: 10-20% CPU during active capture
- **RF scanning**: 15-25% CPU during active scan
- **Terminal sessions**: <2% CPU per active session

### Network Requirements
- **WebSocket connections**: Stable connection required
- **Data export**: High bandwidth for large exports
- **Remote access**: VPN recommended for WAN access

Rate Limiting
-------------

.. list-table:: Diagnostics Rate Limits
   :widths: 40 30 30
   :header-rows: 1

   * - Endpoint Type
     - Requests/Minute
     - Notes
   * - **Log access**
     - 120
     - Per user
   * - **Terminal commands**
     - 60
     - Per session
   * - **Packet capture**
     - 1 concurrent
     - System-wide
   * - **RF scanning**
     - 1 concurrent
     - System-wide
   * - **System control**
     - 10
     - Critical operations

Data Retention
--------------

### Default Retention Periods
- **System logs**: 30 days
- **Packet captures**: 7 days
- **Metrics data**: 1 year (aggregated)
- **Terminal history**: 90 days
- **Diagnostic results**: 30 days

### Automatic Cleanup
- Old data automatically deleted based on retention policy
- Manual cleanup via ``POST /api/diagnostics/clear``
- Configurable retention in settings

Troubleshooting
---------------

### Common Issues

1. **Terminal connection fails**
   - Verify SSH service is running
   - Check user has admin privileges
   - Confirm network connectivity

2. **Logs not updating**
   - Check log service status
   - Verify disk space availability
   - Check log retention settings

3. **Packet capture fails**
   - Verify interface exists and is up
   - Check available storage space
   - Confirm user has capture permissions

4. **RF scanner not responding**
   - Check RF hardware status
   - Verify driver is loaded
   - Confirm frequency range is valid

### Debug Information

When contacting support, provide:
- Gateway ID
- Error message and code
- Timestamp of issue
- Steps to reproduce
- Relevant logs or captures


Glossary
--------

.. glossary::

   System Metrics
      Real-time measurements of CPU, memory, disk, and network usage.

   Log Streaming
      Real-time delivery of system log entries via WebSocket.

   Terminal Session
      Interactive command-line access to the gateway system.

   Packet Capture
      Recording of network traffic for analysis and debugging.

   RF Spectrum Analysis
      Scanning and analysis of radio frequency signals.

   Diagnostic Tools
      Utilities for network testing and system diagnostics.

   Export Package
      Compressed archive containing diagnostics data for analysis.

   Retention Policy
      Rules governing how long diagnostics data is stored.

   Rate Limiting
      Restrictions on API request frequency to prevent abuse.

   WebSocket
      Protocol for full-duplex communication over a single TCP connection.


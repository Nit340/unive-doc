Gateway OTA & System API
=======================================

This document describes the OTA update management page and its related API endpoints for managing Over-the-Air (OTA) updates, system recovery, and maintenance operations.

Page Route (Frontend)
---------------------

.. http:get:: /ota-system-updates

   **Description**: Renders the complete OTA update management page with all system information, available updates, settings, and recovery options embedded in the HTML.

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
        <title>OTA & System Updates - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-system='{...}' data-updates='[...]' data-settings='{...}' data-safety='{...}'>
          <!-- OTA update page with:
               SECTION 1: System Information & Partition Status
               SECTION 2: Available Updates
               SECTION 3: Manual Upload Management
               SECTION 4: Update Settings & Schedule
               SECTION 5: Recovery Operations
               SECTION 6: Safety Constraints
               SECTION 7: Live Update Monitor & Logs
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded system and update data
   - All system information, available updates, settings, and safety status are embedded
   - JavaScript reads this data and renders the complete OTA update interface
   - No separate API calls needed on initial page load
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all OTA update operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

System Information (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/ota/status

   **Description**: Get current gateway system information, partitions, and auto-rollback status.
   
   **UI Element**: SECTION 1: System Information cards and Partition Status
   
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
          "gateway": {
            "model": "Univa-GW-Pro",
            "os_version": "v5.0.4",
            "kernel": "6.1.33-LTS",
            "uptime": "19d 3h 12m",
            "bootloader": "U-Boot 2024.01",
            "last_ota": "2025-03-10T14:32:11Z"
          },
          "partitions": {
            "active": {
              "id": "A",
              "version": "v5.0.4",
              "status": "healthy",
              "size": "8GB",
              "used": "3.2GB",
              "booted": true
            },
            "inactive": {
              "id": "B",
              "version": "v5.0.3",
              "status": "ready",
              "next_boot": true,
              "size": "8GB",
              "used": "2.8GB"
            },
            "auto_rollback": true,
            "rollback_retries": 3
          }
        },
        "success": true,
        "message": "Gateway status retrieved successfully"
      }

.. http:post:: /api/v1/ota/snapshot

   **Description**: Create a system snapshot.
   
   **UI Element**: SECTION 1: "Snapshot" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "name": "pre-update-backup",
        "description": "Snapshot before security update"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "snapshot_id": "snap-006",
          "status": "creating",
          "estimated_duration": "00:10:00",
          "progress_url": "/api/v1/ota/progress?snapshot_id=snap-006"
        },
        "success": true,
        "message": "Snapshot creation started"
      }

.. http:post:: /api/v1/ota/debug-bundle

   **Description**: Generate debug bundle for troubleshooting.
   
   **UI Element**: SECTION 1: "Debug Bundle" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "include_logs": true,
        "include_configs": true,
        "include_metrics": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "bundle_id": "debug-20250315-001",
          "status": "creating",
          "estimated_size": "150 MB",
          "download_url": "/api/v1/ota/debug/download/debug-20250315-001.tar.gz"
        },
        "success": true,
        "message": "Debug bundle creation started"
      }

Available Updates (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/ota/updates

   **Description**: Get list of available updates for the gateway.
   
   **UI Element**: SECTION 2: Available Updates table
   
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
          "updates": [
            {
              "id": "update-001",
              "version": "v5.0.5",
              "type": "security",
              "severity": "critical",
              "size": "452 MB",
              "released": "2025-03-15",
              "description": "Security patches for kernel vulnerabilities...",
              "changelog": [
                "Fixed CVE-2025-1234",
                "Fixed CVE-2025-5678",
                "Improved system stability"
              ],
              "signature": "verified",
              "compatible": true
            }
          ],
          "total_available": 2
        },
        "success": true,
        "message": "Available updates retrieved successfully"
      }

.. http:post:: /api/v1/ota/check-updates

   **Description**: Manually check for new updates.
   
   **UI Element**: SECTION 2: "Check for Updates" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "force": true,
        "background": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "check_id": "check-20250315-001",
          "status": "checking",
          "server": "https://ota.univa.com",
          "estimated_duration": "00:00:30"
        },
        "success": true,
        "message": "Update check initiated"
      }

.. http:post:: /api/v1/ota/auto-update

   **Description**: Toggle auto-update setting.
   
   **UI Element**: SECTION 2: "Auto-Update: On/Off" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enabled": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "auto_update": true,
          "auto_check_enabled": true
        },
        "success": true,
        "message": "Auto-update enabled"
      }

.. http:post:: /api/v1/ota/download

   **Description**: Start downloading an available update.
   
   **UI Element**: SECTION 2: "Install Now" buttons
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "update_id": "update-001",
        "version": "v5.0.5",
        "target_partition": "B",
        "background": false,
        "verify_signature": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "OTA-JOB-8821X",
          "status": "downloading",
          "update": {
            "id": "update-001",
            "version": "v5.0.5",
            "size": "452 MB"
          },
          "target_partition": "B",
          "progress_url": "/api/v1/ota/progress?job_id=OTA-JOB-8821X"
        },
        "success": true,
        "message": "Update download started"
      }

Manual Upload Management (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/ota/upload/verify

   **Description**: Verify manually uploaded firmware image.
   
   **UI Element**: SECTION 3: "Verify & Prepare Deployment" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **file** (required): Firmware image file
   * **version** (optional): Custom version string

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "verified": true,
          "file": {
            "name": "custom-v5.1.0.img",
            "size": "950 MB",
            "checksum": "sha256:1234abcd...",
            "version": "v5.1.0-custom"
          },
          "metadata": {
            "kernel": "6.2.0-LTS",
            "signature": "verified"
          },
          "compatibility": {
            "hardware": "compatible"
          }
        },
        "success": true,
        "message": "Image verified successfully"
      }

.. http:post:: /api/v1/ota/upload/deploy

   **Description**: Deploy manually uploaded firmware image.
   
   **UI Element**: SECTION 3: "Deploy to Partition B" button (appears after verification)
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "filename": "custom-v5.1.0.img",
        "version": "v5.1.0-custom",
        "target_partition": "B",
        "checksum": "sha256:1234abcd..."
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "MANUAL-DEPLOY-001",
          "status": "deploying",
          "image": {
            "name": "custom-v5.1.0.img",
            "version": "v5.1.0-custom"
          },
          "target_partition": "B"
        },
        "success": true,
        "message": "Manual image deployment started"
      }

Update Settings Management (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/ota/settings

   **Description**: Get current update settings and configuration.
   
   **UI Element**: SECTION 4: Update Settings & Schedule cards
   
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
          "auto_update": {
            "enabled": true,
            "auto_download": true,
            "auto_security": true,
            "auto_reboot": false
          },
          "maintenance_window": {
            "enabled": true,
            "schedule": "daily",
            "time": "02:00"
          },
          "update_strategy": "seamless"
        },
        "success": true,
        "message": "Update settings retrieved successfully"
      }

.. http:post:: /api/v1/ota/settings

   **Description**: Update system settings.
   
   **UI Element**: SECTION 4: All settings checkboxes and dropdowns
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "auto_update": {
          "auto_download": true,
          "auto_security": true,
          "auto_reboot": false
        },
        "maintenance_window": {
          "enabled": true,
          "schedule": "daily",
          "time": "02:00"
        },
        "update_strategy": "seamless"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "updated": true,
          "next_maintenance_window": "2025-03-16T02:00:00Z"
        },
        "success": true,
        "message": "Settings updated successfully"
      }

Recovery Operations (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/ota/recovery/rollback

   **Description**: Execute rollback to previous partition.
   
   **UI Element**: SECTION 5: "Execute Rollback" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "action": "rollback_to_a",
        "reason": "Update issues detected"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "action": "rollback_to_a",
          "status": "completed",
          "from_partition": "B",
          "to_partition": "A",
          "from_version": "v5.0.5",
          "to_version": "v5.0.4",
          "reboot_required": true
        },
        "success": true,
        "message": "Rollback executed successfully"
      }

.. http:post:: /api/v1/ota/recovery/restore

   **Description**: Restore from a snapshot.
   
   **UI Element**: SECTION 5: "Start Restore" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "version": "v5.0.3",
        "target_partition": "B"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "RESTORE-001",
          "status": "restoring",
          "version": "v5.0.3",
          "target_partition": "B"
        },
        "success": true,
        "message": "OS restoration started"
      }

.. http:post:: /api/v1/ota/recovery/factory

   **Description**: Perform factory reset.
   
   **UI Element**: SECTION 5: "Factory Reset" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "pin": "0000",
        "confirm": true,
        "wipe_data": true,
        "reason": "System decommissioning"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "started",
          "operation": "factory_reset",
          "estimated_duration": "00:30:00"
        },
        "success": true,
        "message": "Factory reset started"
      }

.. http:post:: /api/v1/ota/recovery/upload

   **Description**: Upload recovery image.
   
   **UI Element**: SECTION 5: "Upload" recovery image button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **file** (required): Recovery image file

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "uploaded": true,
          "filename": "recovery_v1.2.img",
          "size": "2.5 GB"
        },
        "success": true,
        "message": "Recovery image uploaded successfully"
      }

Safety Management (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/ota/safety

   **Description**: Get current safety constraints status.
   
   **UI Element**: SECTION 6: Safety Constraints checkboxes
   
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
          "constraints": {
            "crane_motion": {
              "enabled": true,
              "status": "idle",
              "blocking": false
            },
            "acs_collision": {
              "enabled": true,
              "status": "clear",
              "blocking": false
            },
            "ups_battery": {
              "enabled": true,
              "level": 85,
              "blocking": false
            }
          },
          "force_override": false,
          "update_allowed": true
        },
        "success": true,
        "message": "Safety status retrieved successfully"
      }

.. http:post:: /api/v1/ota/safety/override

   **Description**: Override safety constraints.
   
   **UI Element**: SECTION 6: "Force Override" toggle
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enable": true,
        "pin": "7392",
        "reason": "Emergency update required",
        "constraints": ["crane_motion", "acs_collision"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "override_enabled": true,
          "expires_at": "2025-03-15T12:05:00Z",
          "overridden_constraints": ["crane_motion", "acs_collision"],
          "update_allowed": true
        },
        "success": true,
        "message": "Safety override enabled"
      }

Monitor & Live Updates (SECTION 7)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/ota/progress

   **Description**: Get progress of ongoing update download or installation.
   
   **UI Element**: SECTION 7: Live Update Monitor progress bar
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **job_id** (string): ID of the update job *(required)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "downloading",
          "job_id": "OTA-JOB-8821X",
          "progress": {
            "percentage": 65.5,
            "current_operation": "downloading",
            "speed": "8.2 MB/s",
            "downloaded": "295 MB",
            "total": "452 MB",
            "eta": "00:30"
          },
          "update": {
            "id": "update-001",
            "version": "v5.0.5"
          },
          "elapsed_time": "00:05:30"
        },
        "success": true,
        "message": "Update progress retrieved successfully"
      }

.. http:post:: /api/v1/ota/install

   **Description**: Start installing a downloaded update.
   
   **UI Element**: SECTION 7: Install button after download completes
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "job_id": "OTA-JOB-8821X",
        "strategy": "seamless",
        "reboot_after": true,
        "create_snapshot": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "OTA-JOB-8821X",
          "status": "installing",
          "strategy": "seamless",
          "target_partition": "B",
          "reboot_after": true,
          "next_boot_partition": "B"
        },
        "success": true,
        "message": "Update installation started"
      }

.. http:get:: /api/v1/ota/logs

   **Description**: Get system logs for monitoring.
   
   **UI Element**: SECTION 7: Update Activity Log
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **limit** (integer): Maximum number of logs to return *(optional, default: 50)*
   * **level** (string): Filter by log level *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "logs": [
            {
              "timestamp": "2025-03-15T10:25:30Z",
              "level": "info",
              "source": "ota",
              "message": "Update verification completed"
            }
          ],
          "total": 125
        },
        "success": true,
        "message": "System logs retrieved successfully"
      }

.. http:get:: /api/v1/ota/monitor

   **Description**: Get real-time system monitoring status.
   
   **UI Element**: SECTION 7: Live Update Monitor status
   
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
          "system_status": "idle",
          "current_operation": "none",
          "last_update_check": "2025-03-15T10:30:00Z",
          "next_check": "2025-03-15T18:00:00Z",
          "resources": {
            "cpu_usage": 15.2,
            "memory_usage": "52.5%"
          }
        },
        "success": true,
        "message": "Monitor status retrieved successfully"
      }

.. http:post:: /api/v1/ota/cancel

   **Description**: Cancel ongoing update operation.
   
   **UI Element**: SECTION 7: Cancel button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "job_id": "OTA-JOB-8821X",
        "reason": "User requested cancellation"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "job_id": "OTA-JOB-8821X",
          "status": "cancelled",
          "progress_at_cancel": 65.5
        },
        "success": true,
        "message": "Update operation cancelled"
      }

WebSocket Real-time Updates
---------------------------

Connection Endpoint
~~~~~~~~~~~~~~~~~~~
::

   ws://{gateway-ip}/api/v1/ota/ws
   wss://{gateway-ip}/api/v1/ota/ws (secure)

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

Update Progress
^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "update_progress",
     "job_id": "OTA-JOB-8821X",
     "status": "downloading",
     "progress": 85.5,
     "current_operation": "verifying",
     "speed": "12.5 MB/s",
     "eta": "00:02:15",
     "timestamp": "2025-03-15T11:55:00Z"
   }

System Status Update
^^^^^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "system_status",
     "status": "installing",
     "current_operation": "flashing_partition_b",
     "progress": 45,
     "resources": {
       "cpu_usage": 68.5,
       "memory_usage": 75.2
     },
     "timestamp": "2025-03-15T11:57:00Z"
   }

Log Entry
^^^^^^^^^
.. code-block:: json

   {
     "type": "log_entry",
     "timestamp": "2025-03-15T11:58:00Z",
     "level": "info",
     "source": "ota",
     "message": "Update installation completed successfully"
   }

Safety Update
^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "safety_update",
     "constraint": "crane_motion",
     "status": "moving",
     "blocking": true,
     "estimated_clear": "00:01:30"
   }

Route Summary
-------------

.. list-table:: OTA Update Management Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/ota-system-updates`` - Main OTA update page
     - Yes
   * - System Info
     - GET
     - ``/api/v1/ota/status`` - System information
     - Yes
   * - System Info
     - POST
     - ``/api/v1/ota/snapshot`` - Create snapshot
     - Yes
   * - System Info
     - POST
     - ``/api/v1/ota/debug-bundle`` - Generate debug bundle
     - Yes
   * - Available Updates
     - GET
     - ``/api/v1/ota/updates`` - Available updates
     - Yes
   * - Available Updates
     - POST
     - ``/api/v1/ota/check-updates`` - Check for updates
     - Yes
   * - Available Updates
     - POST
     - ``/api/v1/ota/auto-update`` - Toggle auto-update
     - Yes
   * - Available Updates
     - POST
     - ``/api/v1/ota/download`` - Download update
     - Yes
   * - Manual Upload
     - POST
     - ``/api/v1/ota/upload/verify`` - Verify uploaded image
     - Yes
   * - Manual Upload
     - POST
     - ``/api/v1/ota/upload/deploy`` - Deploy manual image
     - Yes
   * - Settings
     - GET
     - ``/api/v1/ota/settings`` - Get update settings
     - Yes
   * - Settings
     - POST
     - ``/api/v1/ota/settings`` - Update settings
     - Yes
   * - Recovery
     - POST
     - ``/api/v1/ota/recovery/rollback`` - Execute rollback
     - Yes
   * - Recovery
     - POST
     - ``/api/v1/ota/recovery/restore`` - Restore OS
     - Yes
   * - Recovery
     - POST
     - ``/api/v1/ota/recovery/factory`` - Factory reset
     - Yes
   * - Recovery
     - POST
     - ``/api/v1/ota/recovery/upload`` - Upload recovery image
     - Yes
   * - Safety
     - GET
     - ``/api/v1/ota/safety`` - Safety status
     - Yes
   * - Safety
     - POST
     - ``/api/v1/ota/safety/override`` - Safety override
     - Yes
   * - Monitor
     - GET
     - ``/api/v1/ota/progress`` - Update progress
     - Yes
   * - Monitor
     - POST
     - ``/api/v1/ota/install`` - Install update
     - Yes
   * - Monitor
     - GET
     - ``/api/v1/ota/logs`` - System logs
     - Yes
   * - Monitor
     - GET
     - ``/api/v1/ota/monitor`` - Monitor status
     - Yes
   * - Monitor
     - POST
     - ``/api/v1/ota/cancel`` - Cancel operation
     - Yes
   * - Real-time
     - WS
     - ``/api/v1/ota/ws`` - WebSocket for real-time updates
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /ota-system-updates``
   - Server renders HTML with embedded system and update data
   - All 7 sections populated with current data

2. **User views system information** (SECTION 1):
   - Gateway model, OS version, kernel, uptime
   - Partition status (A/B partitions)
   - "Snapshot" button → ``POST /api/v1/ota/snapshot``
   - "Debug Bundle" button → ``POST /api/v1/ota/debug-bundle``

3. **User checks available updates** (SECTION 2):
   - Available updates table with versions and types
   - "Check for Updates" button → ``POST /api/v1/ota/check-updates``
   - "Auto-Update: On/Off" toggle → ``POST /api/v1/ota/auto-update``
   - "Install Now" buttons → ``POST /api/v1/ota/download``

4. **User manages manual uploads** (SECTION 3):
   - File upload area for custom firmware
   - "Verify & Prepare Deployment" → ``POST /api/v1/ota/upload/verify``
   - "Deploy to Partition B" → ``POST /api/v1/ota/upload/deploy``

5. **User configures update settings** (SECTION 4):
   - Auto-update settings checkboxes
   - Maintenance window schedule
   - Update strategy selection
   - "Save Settings" → ``POST /api/v1/ota/settings``

6. **User performs recovery operations** (SECTION 5):
   - "Execute Rollback" → ``POST /api/v1/ota/recovery/rollback``
   - "Start Restore" → ``POST /api/v1/ota/recovery/restore``
   - "Factory Reset" → ``POST /api/v1/ota/recovery/factory``
   - "Upload" recovery image → ``POST /api/v1/ota/recovery/upload``

7. **User monitors safety constraints** (SECTION 6):
   - Safety constraint status (crane motion, ACS, UPS battery)
   - "Force Override" toggle → ``POST /api/v1/ota/safety/override``

8. **User monitors live updates** (SECTION 7):
   - Progress bar for ongoing operations → ``GET /api/v1/ota/progress``
   - "Install" button after download → ``POST /api/v1/ota/install``
   - Update activity logs → ``GET /api/v1/ota/logs``
   - Real-time monitoring → ``GET /api/v1/ota/monitor``
   - "Cancel" button → ``POST /api/v1/ota/cancel``
   - WebSocket connection for real-time updates → ``/api/v1/ota/ws``

Error Codes
-----------

.. list-table:: OTA Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - OTA_INVALID_IMAGE
     - Invalid firmware image
   * - OTA_SIGNATURE_FAILED
     - Signature verification failed
   * - OTA_INSUFFICIENT_SPACE
     - Insufficient disk space for update
   * - OTA_UPDATE_BLOCKED
     - Update blocked by safety constraints
   * - OTA_DOWNLOAD_FAILED
     - Download failed
   * - OTA_INSTALL_FAILED
     - Installation failed
   * - OTA_ROLLBACK_FAILED
     - Rollback operation failed
   * - OTA_RECOVERY_FAILED
     - Recovery operation failed
   * - OTA_INVALID_PIN
     - Invalid PIN for operation
   * - OTA_OPERATION_IN_PROGRESS
     - Another operation is in progress

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Cookie: session_token=<token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages OTA updates for the current gateway specified in `X-Gateway-ID` header.

Update Types
------------

### Update Severity Levels
- **Critical**: Security patches, critical bug fixes
- **High**: Important feature updates
- **Medium**: Minor improvements and optimizations
- **Low**: Optional feature enhancements

### Update Strategies
1. **Seamless Update**: Download to inactive partition, switch on reboot
2. **In-place Update**: Update current partition directly
3. **Rolling Update**: Update one component at a time

Safety Constraints
------------------

### Constraint Types
1. **Crane Motion**: Blocks updates if crane is in motion
2. **ACS (Anti-Collision System)**: Blocks updates if collision risk detected
3. **UPS Battery**: Blocks updates if battery below threshold (typically 30%)
4. **Network Connectivity**: Requires stable network connection
5. **System Load**: Blocks updates during high system load

### Override Process
1. User provides PIN for verification
2. Specify reason for override
3. Select constraints to override
4. Override expires after 15 minutes or operation completion

Best Practices
--------------

1. **Create Snapshots**: Always create snapshot before major updates
2. **Check Safety Constraints**: Verify all safety constraints are clear
3. **Schedule During Maintenance**: Use maintenance windows for updates
4. **Monitor Progress**: Watch update progress in real-time
5. **Test After Update**: Verify system functionality post-update
6. **Enable Auto-rollback**: Keep auto-rollback enabled for safety
7. **Keep Recovery Images**: Maintain current recovery images
8. **Verify Signatures**: Always verify update signatures


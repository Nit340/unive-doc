Gateway OTA & System Update API Documentation
===============================================

API for managing Over-the-Air (OTA) updates, system recovery, and maintenance operations for Univa IoT Gateway.

Base Path: /api/v1/ota

.. contents:: Table of Contents
   :depth: 3
   :local:

Overview
--------

This API handles all OTA update operations for the current gateway only. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

System Information
~~~~~~~~~~~~~~~~~~

Get Gateway Status
^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/status

   Get current gateway system information, partitions, and auto-rollback status.
   
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

Create System Snapshot
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/snapshot

   Create a system snapshot.
   
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

Available Updates
~~~~~~~~~~~~~~~~~

Check Available Updates
^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/updates

   Get list of available updates for the gateway.
   
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

Toggle Auto-Update
^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/auto-update

   Toggle auto-update setting.
   
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

Download & Installation
~~~~~~~~~~~~~~~~~~~~~~~

Start Update Download
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/download

   Start downloading an available update.
   
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

Get Update Progress
^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/progress

   Get progress of ongoing update download or installation.
   
   **UI Element**: SECTION 5: Live Update Monitor progress bar
   
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

Install Downloaded Update
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/install

   Start installing a downloaded update.
   
   **UI Element**: SECTION 5: Progress completion leading to installation
   
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

Manual Upload Management
~~~~~~~~~~~~~~~~~~~~~~~~

Verify Uploaded Image
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/upload/verify

   Verify manually uploaded firmware image.
   
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

Deploy Manual Image
^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/upload/deploy

   Deploy manually uploaded firmware image.
   
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

Update Settings Management
~~~~~~~~~~~~~~~~~~~~~~~~~~

Get Update Settings
^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/settings

   Get current update settings and configuration.
   
   **UI Element**: SECTION 3: Update Settings & Schedule cards
   
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

Update Settings
^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/settings

   Update system settings.
   
   **UI Element**: SECTION 3: All settings checkboxes and dropdowns
   
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

Recovery Operations
~~~~~~~~~~~~~~~~~~~

Execute Rollback
^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/recovery/rollback

   Execute rollback to previous partition.
   
   **UI Element**: SECTION 4: "Execute Rollback" button
   
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

Restore Previous OS
^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/recovery/restore

   Restore from a snapshot.
   
   **UI Element**: SECTION 4: "Start Restore" button
   
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

Factory Reset
^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/recovery/factory

   Perform factory reset.
   
   **UI Element**: SECTION 4: "Factory Reset" button
   
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

Upload Recovery Image
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/recovery/upload

   Upload recovery image.
   
   **UI Element**: SECTION 4: "Upload" recovery image button
   
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

Safety Management
~~~~~~~~~~~~~~~~~

Get Safety Status
^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/safety

   Get current safety constraints status.
   
   **UI Element**: SECTION 4: Safety Constraints checkboxes
   
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

Safety Override
^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/safety/override

   Override safety constraints.
   
   **UI Element**: SECTION 4: "Force Override" toggle
   
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

Monitor & Logs
~~~~~~~~~~~~~~

Get System Logs
^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/logs

   Get system logs for monitoring.
   
   **UI Element**: SECTION 5: Update Activity Log
   
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

Get Monitor Status
^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/ota/monitor

   Get real-time system monitoring status.
   
   **UI Element**: SECTION 5: Live Update Monitor
   
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

Cancel Operation
~~~~~~~~~~~~~~~~

Cancel Update Operation
^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/ota/cancel

   Cancel ongoing update operation.
   
   **UI Element**: SECTION 5: Cancel button (implied in UI)
   
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

Error Codes
-----------

.. list-table:: OTA Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **OTA_INVALID_IMAGE**
     - Invalid firmware image
   * - **OTA_SIGNATURE_FAILED**
     - Signature verification failed
   * - **OTA_INSUFFICIENT_SPACE**
     - Insufficient disk space for update
   * - **OTA_UPDATE_BLOCKED**
     - Update blocked by safety constraints
   * - **OTA_DOWNLOAD_FAILED**
     - Download failed
   * - **OTA_INSTALL_FAILED**
     - Installation failed
   * - **OTA_ROLLBACK_FAILED**
     - Rollback operation failed
   * - **OTA_RECOVERY_FAILED**
     - Recovery operation failed
   * - **OTA_INVALID_PIN**
     - Invalid PIN for operation
   * - **OTA_OPERATION_IN_PROGRESS**
     - Another operation is in progress

Authentication
--------------

All endpoints require:

.. code-block:: http

   Authorization: Bearer <jwt_token>
   X-Gateway-ID: GW-3920A9

Rate Limiting
-------------

* 60 requests per minute per user
* Rate limit headers included in responses

Response Format
---------------

All responses:

.. code-block:: json

   {
     "data": { ... },
     "success": true,
     "message": "Operation successful"
   }

Error responses:

.. code-block:: json

   {
     "data": null,
     "success": false,
     "error": {
       "code": "OTA_UPDATE_BLOCKED",
       "message": "Update blocked by safety constraints"
     }
   }

UI-API Mapping Summary
----------------------

.. list-table:: OTA UI to API Mapping
   :widths: 25 35 40
   :header-rows: 1

   * - UI Section
     - GET Endpoints
     - POST Endpoints
   * - **System Info**
     - ``/ota/status``
     - ``/ota/snapshot``
   * - **Available Updates**
     - ``/ota/updates``
     - ``/ota/auto-update``, ``/ota/download``
   * - **Update Progress**
     - ``/ota/progress``
     - ``/ota/install``
   * - **Manual Upload**
     - -
     - ``/ota/upload/verify``, ``/ota/upload/deploy``
   * - **Update Settings**
     - ``/ota/settings``
     - ``/ota/settings`` (POST)
   * - **Recovery**
     - -
     - ``/ota/recovery/rollback``, ``/ota/recovery/restore``, ``/ota/recovery/factory``, ``/ota/recovery/upload``
   * - **Safety**
     - ``/ota/safety``
     - ``/ota/safety/override``
   * - **Monitor**
     - ``/ota/logs``, ``/ota/monitor``
     - ``/ota/cancel``
   * - **Real-time**
     - WebSocket ``/ota/ws``
     - -
System OTA & Recovery API
=========================

APIs for managing Over-The-Air (OTA) updates, system recovery, and maintenance operations.

Overview
--------

The System OTA & Recovery API provides comprehensive functionality for managing gateway firmware updates, system recovery operations, and maintenance tasks. Supports dual-boot partitions for seamless updates and rollbacks.

Endpoints
---------

Gateway Status
~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/gateway/status

   Get detailed gateway status and partition information.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "online",
        "gateway": {
          "id": "univa-gw-01",
          "name": "Univa-GW-01",
          "model": "Univa-GW-Pro",
          "os_version": "v5.0.4",
          "kernel": "6.1.33-LTS",
          "uptime": "19d 3h 12m",
          "bootloader": "U-Boot 2024.01",
          "last_ota": "2025-03-10 14:32:11",
          "serial_number": "GW2025-1190021",
          "hardware_revision": "RevB"
        },
        "partitions": {
          "active": {
            "id": "A",
            "version": "v5.0.4",
            "status": "healthy",
            "boot_count": 127,
            "size": "8GB",
            "used": "3.2GB",
            "health": "good"
          },
          "inactive": {
            "id": "B",
            "version": "v5.0.3",
            "status": "ready",
            "next_boot": true,
            "size": "8GB",
            "used": "2.8GB",
            "health": "good"
          },
          "auto_rollback": true,
          "rollback_retries": 3,
          "rollback_count": 2
        },
        "system": {
          "cpu_usage": 12.5,
          "memory_usage": 65.3,
          "disk_usage": 40.2,
          "temperature": 42.1,
          "network_status": "connected"
        }
      }

Updates Management
------------------

Check for Available Updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/updates/check

   Check for available firmware updates.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **channel** (optional): Update channel (stable, beta, nightly)
   * **force** (optional): Force check bypassing cache

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "updates": [
          {
            "id": "update-001",
            "version": "v5.0.5",
            "type": "security",
            "severity": "critical",
            "size": 452,
            "size_unit": "MB",
            "released": "2025-03-15",
            "description": "Security patches for kernel vulnerabilities and network stack improvements.",
            "changelog": [
              "Fixed CVE-2025-1234 (Kernel memory leak)",
              "Fixed CVE-2025-5678 (Network stack vulnerability)",
              "Improved Modbus driver stability",
              "Enhanced container runtime security"
            ],
            "signature": "verified",
            "compatible": true,
            "prerequisites": ["v5.0.0+"],
            "checksum": "sha256:abcd1234...",
            "download_url": "https://updates.example.com/v5.0.5/gateway.img"
          },
          {
            "id": "update-002",
            "version": "v5.1.0",
            "type": "feature",
            "severity": "normal",
            "size": 1200,
            "size_unit": "MB",
            "released": "2025-03-10",
            "description": "Major feature release with improved container runtime.",
            "changelog": [
              "New container runtime v2.0",
              "Enhanced monitoring dashboard",
              "Added support for CAN FD",
              "Improved OTA reliability"
            ],
            "signature": "verified",
            "compatible": true,
            "checksum": "sha256:efgh5678...",
            "download_url": "https://updates.example.com/v5.1.0/gateway.img"
          }
        ],
        "last_checked": "2025-03-15T10:30:00Z",
        "next_check": "2025-03-15T18:00:00Z"
      }

Start Update Download
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/updates/download

   Start downloading an update to the inactive partition.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/updates/download HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "update_id": "update-001",
        "version": "v5.0.5",
        "target_partition": "B",
        "background": false,
        "verify_signature": true,
        "verify_checksum": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "started",
        "job_id": "OTA-JOB-8821X",
        "update": {
          "id": "update-001",
          "version": "v5.0.5",
          "size": 452,
          "size_unit": "MB",
          "type": "security",
          "severity": "critical"
        },
        "target_partition": "B",
        "estimated_time": 120,
        "download_speed_limit": 0,
        "resumable": true,
        "cancelable": true
      }

Get Update Progress
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/updates/progress

   Get progress of current update operation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **job_id** (optional): Specific job ID to check

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "downloading",
        "job_id": "OTA-JOB-8821X",
        "progress": {
          "percentage": 65.5,
          "current_operation": "downloading",
          "speed": "8.2 MB/s",
          "downloaded": "295 MB",
          "total": "452 MB",
          "eta": "00:30",
          "elapsed": "00:35"
        },
        "update": {
          "version": "v5.0.5",
          "type": "security",
          "target_partition": "B"
        },
        "status_details": {
          "bytes_downloaded": 295000000,
          "bytes_total": 452000000,
          "connections": 1,
          "errors": 0,
          "retries": 0
        }
      }

Start Update Installation
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/updates/install

   Install downloaded update to inactive partition.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/updates/install HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "job_id": "OTA-JOB-8821X",
        "strategy": "seamless",
        "reboot_after": true,
        "reboot_delay": 60,
        "verify_installation": true,
        "create_snapshot": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "installing",
        "installation_id": "INST-8832Y",
        "progress": 25,
        "current_operation": "verifying_signature",
        "estimated_completion": "2025-03-15T10:25:00Z",
        "warning": "Gateway will be unavailable during installation",
        "next_steps": [
          "Verifying digital signature",
          "Checking partition integrity",
          "Writing image to partition",
          "Updating bootloader",
          "Rebooting (if enabled)"
        ]
      }

Manual Upload
-------------

Verify Uploaded Image
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/upload/verify

   Verify manually uploaded OS image.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/upload/verify HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="custom-v5.1.0.img"
      Content-Type: application/octet-stream
      
      --form-data; name="version"
      
      v5.1.0-custom

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "verified",
        "file": {
          "name": "custom-v5.1.0.img",
          "size": "950 MB",
          "checksum": "sha256:1234abcd...",
          "version": "v5.1.0-custom",
          "format": "raw",
          "compression": "gzip"
        },
        "metadata": {
          "kernel": "6.2.0-LTS",
          "build_id": "BUILD-20250315",
          "signature": "verified",
          "compatibility": "compatible",
          "architecture": "arm64",
          "bootloader_version": "2024.01",
          "minimum_hardware": "RevA"
        },
        "validation": {
          "signature_valid": true,
          "checksum_valid": true,
          "partition_layout_valid": true,
          "bootloader_compatible": true,
          "hardware_compatible": true,
          "size_adequate": true
        }
      }

Deploy Manual Image
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/upload/deploy

   Deploy verified manual image to inactive partition.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/upload/deploy HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "filename": "custom-v5.1.0.img",
        "version": "v5.1.0-custom",
        "target_partition": "B",
        "verify_before_deploy": true,
        "create_snapshot": true,
        "description": "Custom build with enhanced features"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "deploying",
        "deployment_id": "DEPLOY-9945Z",
        "version": "v5.1.0-custom",
        "target_partition": "B",
        "estimated_time": 300,
        "steps": [
          "Preparing partition",
          "Writing image",
          "Verifying write",
          "Updating partition table",
          "Creating recovery snapshot"
        ]
      }

Settings Management
-------------------

Get Update Settings
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/settings

   Get current OTA and update settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "auto_update": {
          "enabled": true,
          "auto_download": true,
          "auto_security": true,
          "auto_feature": false,
          "auto_reboot": false,
          "auto_rollback": true
        },
        "channels": {
          "current": "stable",
          "available": ["stable", "beta", "nightly"]
        },
        "maintenance_window": {
          "enabled": true,
          "schedule": "daily",
          "time": "02:00",
          "duration": 3600,
          "days": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
        },
        "update_strategy": "seamless",
        "verification": {
          "verify_signature": true,
          "verify_checksum": true,
          "verify_size": true
        },
        "safety_constraints": {
          "crane_motion_lock": true,
          "acs_collision_warning": true,
          "ups_battery_check": true,
          "minimum_battery_level": 30,
          "network_connectivity": true,
          "disk_space_threshold": 20
        },
        "notifications": {
          "available_updates": true,
          "download_complete": true,
          "installation_complete": true,
          "errors": true,
          "reboot_warning": true
        }
      }

Update Settings
~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/settings/update

   Update OTA and system settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/settings/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "auto_update": {
          "enabled": false,
          "auto_download": true,
          "auto_security": true,
          "auto_feature": false
        },
        "channels": {
          "current": "beta"
        },
        "update_strategy": "maintenance",
        "maintenance_window": {
          "schedule": "weekends",
          "time": "04:00",
          "duration": 7200
        },
        "verification": {
          "verify_signature": true,
          "verify_checksum": true
        },
        "notifications": {
          "available_updates": true,
          "download_complete": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "message": "Settings updated successfully",
        "restart_required": false,
        "changes": {
          "auto_update.enabled": {"old": true, "new": false},
          "channels.current": {"old": "stable", "new": "beta"},
          "maintenance_window.schedule": {"old": "daily", "new": "weekends"}
        }
      }

Recovery Operations
-------------------

Get Recovery Status
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/recovery/status

   Get recovery system status and available snapshots.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "recovery_image": {
          "present": true,
          "version": "recovery_v1.2",
          "size": "2.5 GB",
          "checksum": "sha256:recovery123...",
          "last_updated": "2025-03-10T14:30:00Z"
        },
        "snapshots": [
          {
            "id": "snap-001",
            "timestamp": "2025-03-10 14:32:11",
            "version": "v5.0.4",
            "type": "automatic",
            "size": "2.2 GB",
            "description": "Automatic snapshot before update",
            "partition": "A",
            "bootable": true
          },
          {
            "id": "snap-002",
            "timestamp": "2025-02-28 03:00:00",
            "version": "v5.0.3",
            "type": "manual",
            "size": "2.1 GB",
            "description": "Manual backup before feature deployment",
            "partition": "B",
            "bootable": true
          }
        ],
        "recovery_options": {
          "rollback_enabled": true,
          "snapshot_enabled": true,
          "factory_reset_enabled": true,
          "max_snapshots": 10,
          "snapshot_retention_days": 30
        }
      }

Execute Rollback
~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/recovery/rollback

   Rollback to previous working partition.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/recovery/rollback HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "action": "rollback_to_a",
        "reason": "Update issues detected - high memory usage",
        "verify_rollback": true,
        "create_diagnostic_report": true,
        "preserve_data": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "rolling_back",
        "action": "switch_to_partition_a",
        "rollback_id": "ROLLBACK-5567",
        "from_version": "v5.0.5",
        "to_version": "v5.0.4",
        "estimated_time": 120,
        "warning": "Gateway will reboot during rollback",
        "data_preserved": true,
        "diagnostic_report_id": "DIAG-8899"
      }

Restore Previous OS
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/recovery/restore

   Restore from a specific snapshot.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/recovery/restore HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "version": "v5.0.3",
        "snapshot_id": "snap-002",
        "target_partition": "A",
        "verify_restore": true,
        "description": "Restoring to stable version after failed update"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "restoring",
        "version": "v5.0.3",
        "snapshot_id": "snap-002",
        "restore_id": "RESTORE-6677",
        "estimated_time": 180,
        "steps": [
          "Verifying snapshot integrity",
          "Preparing target partition",
          "Restoring from snapshot",
          "Updating bootloader",
          "Verifying restored system"
        ]
      }

Factory Reset
~~~~~~~~~~~~~

.. http:post:: /cloud-integration/recovery/factory

   Perform factory reset (WARNING: All data will be lost).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/recovery/factory HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "pin": "0000",
        "confirm": true,
        "reset_type": "complete",
        "backup_config": true,
        "preserve_network": false,
        "reason": "Returning gateway to factory settings for troubleshooting"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "factory_reset_started",
        "reset_id": "FACTORY-7788",
        "warning": "ALL DATA WILL BE LOST. This action cannot be undone.",
        "estimated_time": 600,
        "steps": [
          "Creating emergency backup",
          "Wiping user data",
          "Restoring factory image",
          "Resetting configurations",
          "Rebooting system"
        ],
        "backup_created": true,
        "backup_location": "/recovery/backup_20250315.tar.gz"
      }

Safety Management
-----------------

Check Safety Status
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/safety/status

   Check safety constraints before allowing updates.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "constraints": {
          "crane_motion": {
            "enabled": true,
            "status": "idle",
            "blocking": false,
            "last_movement": "2025-03-15T09:30:00Z",
            "timeout": 300
          },
          "acs_collision": {
            "enabled": true,
            "status": "clear",
            "blocking": false,
            "distance": 15.2,
            "safe_threshold": 5.0
          },
          "ups_battery": {
            "enabled": true,
            "level": 85,
            "minimum_required": 30,
            "blocking": false,
            "estimated_runtime": "2h 30m"
          },
          "network_connectivity": {
            "enabled": true,
            "status": "connected",
            "blocking": false,
            "latency": 45,
            "stability": "good"
          },
          "disk_space": {
            "enabled": true,
            "available": "12GB",
            "minimum_required": "5GB",
            "blocking": false
          }
        },
        "force_override": false,
        "update_allowed": true,
        "safety_score": 95,
        "recommendations": [
          "System ready for updates",
          "Consider scheduling during maintenance window"
        ]
      }

Override Safety Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/safety/override

   Temporarily override safety constraints (requires PIN).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/safety/override HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enable": true,
        "pin": "7392",
        "reason": "Emergency security update required",
        "constraints": ["crane_motion", "ups_battery"],
        "duration": 3600,
        "acknowledge_risks": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "override_enabled",
        "override_id": "OVERRIDE-8899",
        "warning": "Safety constraints disabled. Proceed with extreme caution.",
        "duration": 3600,
        "expires_at": "2025-03-15T11:30:45Z",
        "constraints_overridden": ["crane_motion", "ups_battery"],
        "acknowledgments": {
          "risk_accepted": true,
          "operator": "admin_user",
          "timestamp": "2025-03-15T10:30:45Z"
        }
      }

Monitoring & Logging
--------------------

Get System Logs
~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/monitor/logs

   Get OTA and system logs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **limit** (optional): Number of log entries (default: 50, max: 1000)
   * **level** (optional): Filter by level (info, warning, error, debug)
   * **source** (optional): Filter by source (ota, recovery, safety, system)
   * **since** (optional): Filter logs since timestamp
   * **until** (optional): Filter logs until timestamp

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "logs": [
          {
            "timestamp": "2025-03-15 10:25:30",
            "level": "info",
            "source": "ota",
            "message": "Update verification completed",
            "details": {
              "update_id": "update-001",
              "version": "v5.0.5",
              "checksum": "sha256:abcd1234..."
            }
          },
          {
            "timestamp": "2025-03-15 10:20:00",
            "level": "info",
            "source": "ota",
            "message": "Download started: v5.0.5",
            "details": {
              "job_id": "OTA-JOB-8821X",
              "size": "452 MB",
              "target": "partition_B"
            }
          },
          {
            "timestamp": "2025-03-15 10:15:00",
            "level": "info",
            "source": "system",
            "message": "System ready for updates",
            "details": {
              "safety_check": "passed",
              "constraints": "all clear"
            }
          }
        ],
        "total": 150,
        "filtered": 50,
        "levels": {
          "info": 120,
          "warning": 20,
          "error": 10,
          "debug": 0
        }
      }

Get Current Status
~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/monitor/status

   Get current OTA system status.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "system_status": "idle",
        "current_operation": "none",
        "last_update_check": "2025-03-15 10:30:00",
        "next_check": "2025-03-15 18:00:00",
        "auto_update": "enabled",
        "active_jobs": 0,
        "queued_jobs": 1,
        "health": {
          "system": "healthy",
          "partitions": "healthy",
          "recovery": "ready",
          "safety": "enabled"
        },
        "resources": {
          "disk_free": "12GB",
          "memory_free": "2.1GB",
          "cpu_idle": 87.5,
          "network_available": true
        }
      }

Create System Snapshot
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/monitor/snapshot

   Create a system snapshot for backup.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/monitor/snapshot HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "pre-update-backup",
        "description": "Snapshot before security update v5.0.5",
        "include_data": true,
        "include_config": true,
        "include_logs": false,
        "compression": "gzip",
        "priority": "normal"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "snapshot_started",
        "snapshot_id": "snap-003",
        "name": "pre-update-backup",
        "estimated_size": "2.2 GB",
        "estimated_time": 300,
        "steps": [
          "Preparing filesystem",
          "Creating system image",
          "Backup user data",
          "Backup configurations",
          "Creating metadata",
          "Compressing archive"
        ]
      }

Get Debug Bundle
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/monitor/debug

   Generate and download debug bundle for troubleshooting.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **include_logs** (optional): Include system logs (default: true)
   * **include_config** (optional): Include configurations (default: true)
   * **include_system_info** (optional): Include system info (default: true)
   * **compression** (optional): Compression type (gzip, none)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "generating",
        "bundle_id": "debug-20250315",
        "estimated_time": 60,
        "size_estimate": "150 MB",
        "contents": [
          "System logs (last 7 days)",
          "Configuration files",
          "System information",
          "Hardware diagnostics",
          "Network configuration",
          "Update history"
        ],
        "download_url": "/cloud-integration/monitor/debug/download/debug-20250315.tar.gz",
        "expires_at": "2025-03-16T10:30:45Z"
      }

Job Management
--------------

List Active Jobs
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/jobs

   List active and recent OTA jobs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional): Filter by status (active, completed, failed, pending)
   * **type** (optional): Filter by type (download, install, rollback, snapshot)
   * **limit** (optional): Maximum results (default: 20, max: 100)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "jobs": [
          {
            "id": "OTA-JOB-8821X",
            "type": "update",
            "subtype": "download",
            "status": "downloading",
            "progress": 65,
            "started": "2025-03-15 10:15:00",
            "estimated_completion": "2025-03-15 10:20:00",
            "update": {
              "version": "v5.0.5",
              "size": "452 MB"
            },
            "user": "admin",
            "cancelable": true
          },
          {
            "id": "SNAP-003",
            "type": "snapshot",
            "subtype": "manual",
            "status": "pending",
            "progress": 0,
            "started": "2025-03-15 10:16:00",
            "estimated_completion": "2025-03-15 10:21:00",
            "details": {
              "name": "pre-update-backup",
              "size_estimate": "2.2 GB"
            },
            "user": "admin",
            "cancelable": true
          }
        ],
        "summary": {
          "active": 2,
          "pending": 1,
          "completed_today": 0,
          "failed_today": 0
        }
      }

Cancel Job
~~~~~~~~~~

.. http:post:: /cloud-integration/jobs/cancel

   Cancel an active OTA job.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/jobs/cancel HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "job_id": "OTA-JOB-8821X",
        "reason": "User requested cancellation",
        "cleanup": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "cancelled",
        "job_id": "OTA-JOB-8821X",
        "message": "Job cancelled successfully",
        "cleanup_performed": true,
        "partial_data_removed": true,
        "can_restart": false
      }

History & Reporting
-------------------

Get Update History
~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/history/updates

   Get history of past updates.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **limit** (optional): Number of entries (default: 20, max: 100)
   * **status** (optional): Filter by status (success, failed, cancelled)
   * **type** (optional): Filter by type (security, feature, hotfix)
   * **since** (optional): Filter since date
   * **until** (optional): Filter until date

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "history": [
          {
            "id": "update-001",
            "version": "v5.0.4",
            "type": "security",
            "status": "success",
            "date": "2025-03-10",
            "time": "02:00:00",
            "duration": "15m",
            "partition": "B",
            "size": "420 MB",
            "initiated_by": "auto",
            "rollback": false,
            "details": {
              "changelog": ["Security patches", "Bug fixes"],
              "safety_check": "passed",
              "verification": "success"
            }
          },
          {
            "id": "update-002",
            "version": "v5.0.3",
            "type": "hotfix",
            "status": "success",
            "date": "2025-02-28",
            "time": "03:00:00",
            "duration": "10m",
            "partition": "A",
            "size": "150 MB",
            "initiated_by": "manual",
            "rollback": false,
            "details": {
              "changelog": ["Fixed memory leak"],
              "safety_check": "passed"
            }
          },
          {
            "id": "update-003",
            "version": "v5.0.2",
            "type": "feature",
            "status": "failed",
            "date": "2025-02-15",
            "time": "02:30:00",
            "duration": "5m",
            "partition": "B",
            "size": "1.2 GB",
            "initiated_by": "manual",
            "rollback": true,
            "error": "Checksum mismatch",
            "error_details": {
              "expected": "sha256:abcd...",
              "actual": "sha256:efgh...",
              "resolution": "Redownload required"
            }
          }
        ],
        "statistics": {
          "total_updates": 24,
          "success_rate": "92%",
          "average_duration": "12m",
          "last_success": "2025-03-10",
          "last_failure": "2025-02-15"
        }
      }

WebSocket Real-time Updates
---------------------------

Connection Endpoint
~~~~~~~~~~~~~~~~~~~

::
   
   ws://gateway-ip/cloud-integration/ws/updates
   wss://gateway-ip/cloud-integration/ws/updates (secure)

Message Types
~~~~~~~~~~~~~

**Progress Updates**:

.. code-block:: json

   {
     "type": "progress_update",
     "job_id": "OTA-JOB-8821X",
     "progress": 85,
     "current_operation": "writing_partition",
     "eta": "00:05:00",
     "details": {
       "bytes_written": "385MB",
       "bytes_total": "452MB",
       "speed": "5.2MB/s"
     }
   }

**Status Changes**:

.. code-block:: json

   {
     "type": "status_change",
     "system_status": "updating",
     "message": "Update installation in progress",
     "severity": "info",
     "timestamp": "2025-03-15T10:25:30Z"
   }

**Safety Alerts**:

.. code-block:: json

   {
     "type": "safety_alert",
     "constraint": "crane_motion",
     "status": "violated",
     "message": "Crane movement detected during update",
     "severity": "warning",
     "action_required": true,
     "suggested_action": "Pause update or override safety"
   }

**Job Completion**:

.. code-block:: json

   {
     "type": "job_complete",
     "job_id": "OTA-JOB-8821X",
     "status": "success",
     "message": "Update downloaded successfully",
     "details": {
       "version": "v5.0.5",
       "size": "452MB",
       "duration": "5m 30s"
     }
   }

**System Events**:

.. code-block:: json

   {
     "type": "system_event",
     "event": "reboot_scheduled",
     "message": "System reboot scheduled in 60 seconds",
     "timestamp": "2025-03-15T10:26:00Z",
     "countdown": 60
   }

Error Handling
--------------

Error Response Format
~~~~~~~~~~~~~~~~~~~~~

.. sourcecode:: http

   HTTP/1.1 400 Bad Request
   Content-Type: application/json
   
   {
     "error": true,
     "code": "UPDATE_BLOCKED",
     "message": "Update blocked by safety constraints",
     "details": {
       "constraint": "crane_motion",
       "status": "moving",
       "detected_at": "2025-03-15T10:30:45Z",
       "resolution": "Wait for crane to stop or use force override with PIN",
       "estimated_wait": "00:02:30"
     },
     "suggested_actions": [
       "Wait for operations to complete",
       "Schedule update during maintenance window",
       "Use safety override with proper authorization"
     ],
     "request_id": "req_abc123",
     "timestamp": "2025-03-15T10:30:45Z"
   }

Common Error Codes
~~~~~~~~~~~~~~~~~~

.. list-table:: OTA Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **OTA_001**
     - Update already in progress
   * - **OTA_002**
     - Safety constraint violation
   * - **OTA_003**
     - Insufficient disk space
   * - **OTA_004**
     - Network connectivity issue
   * - **OTA_005**
     - Invalid update file
   * - **OTA_006**
     - Signature verification failed
   * - **OTA_007**
     - Checksum mismatch
   * - **OTA_008**
     - Invalid target partition
   * - **OTA_009**
     - Update server unreachable
   * - **OTA_010**
     - Invalid PIN for override
   * - **OTA_011**
     - Factory reset not allowed
   * - **OTA_012**
     - Rollback not available
   * - **OTA_013**
     - Job not found or completed
   * - **OTA_014**
     - System busy with other operation
   * - **OTA_015**
     - Maintenance window active

Examples
--------

Python - Complete Update Flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   import time
   
   # Check gateway status
   headers = {"Authorization": "Bearer your_token"}
   
   status_response = requests.get(
       "https://api.example.com/v1/cloud-integration/gateway/status",
       headers=headers
   )
   
   status = status_response.json()
   print(f"Gateway: {status['gateway']['name']}")
   print(f"Active Partition: {status['partitions']['active']['version']}")
   
   # Check for updates
   updates_response = requests.get(
       "https://api.example.com/v1/cloud-integration/updates/check",
       headers=headers
   )
   
   updates = updates_response.json()
   
   if updates['updates']:
       update = updates['updates'][0]  # Get first available update
       print(f"Available update: {update['version']} - {update['description']}")
       
       # Start download
       download_request = {
           "update_id": update['id'],
           "version": update['version'],
           "target_partition": "B"
       }
       
       download_response = requests.post(
           "https://api.example.com/v1/cloud-integration/updates/download",
           json=download_request,
           headers=headers
       )
       
       download_result = download_response.json()
       job_id = download_result['job_id']
       print(f"Download started: {job_id}")
       
       # Monitor progress
       while True:
           progress_response = requests.get(
               "https://api.example.com/v1/cloud-integration/updates/progress",
               params={"job_id": job_id},
               headers=headers
           )
           
           progress = progress_response.json()
           print(f"Progress: {progress['progress']['percentage']:.1f}%")
           
           if progress['status'] == 'downloaded':
               break
           
           time.sleep(5)
       
       # Install update
       install_request = {
           "job_id": job_id,
           "strategy": "seamless",
           "reboot_after": True
       }
       
       install_response = requests.post(
           "https://api.example.com/v1/cloud-integration/updates/install",
           json=install_request,
           headers=headers
       )
       
       print("Update installation started. System will reboot.")

Python - Manual Image Upload
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Upload custom image
   import requests
   
   # Upload and verify
   with open('custom-v5.1.0.img', 'rb') as f:
       files = {
           'file': ('custom-v5.1.0.img', f, 'application/octet-stream'),
           'version': (None, 'v5.1.0-custom')
       }
       
       verify_response = requests.post(
           "https://api.example.com/v1/cloud-integration/upload/verify",
           files=files,
           headers={"Authorization": "Bearer your_token"}
       )
   
   verify_result = verify_response.json()
   
   if verify_result['status'] == 'verified':
       print(f"Image verified: {verify_result['file']['version']}")
       
       # Deploy verified image
       deploy_request = {
           "filename": verify_result['file']['name'],
           "version": verify_result['file']['version'],
           "target_partition": "B"
       }
       
       deploy_response = requests.post(
           "https://api.example.com/v1/cloud-integration/upload/deploy",
           json=deploy_request,
           headers={"Authorization": "Bearer your_token"}
       )
       
       print(f"Deployment started: {deploy_response.json()['deployment_id']}")
   else:
       print(f"Verification failed: {verify_result.get('error', 'Unknown error')}")

Python - Safety Override and Update
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Check safety status
   safety_response = requests.get(
       "https://api.example.com/v1/cloud-integration/safety/status",
       headers=headers
   )
   
   safety = safety_response.json()
   
   if not safety['update_allowed']:
       print("Safety constraints blocking update")
       
       # Override safety (requires PIN)
       override_request = {
           "enable": True,
           "pin": "7392",
           "reason": "Emergency security update",
           "duration": 1800,
           "acknowledge_risks": True
       }
       
       override_response = requests.post(
           "https://api.example.com/v1/cloud-integration/safety/override",
           json=override_request,
           headers=headers
       )
       
       override_result = override_response.json()
       
       if override_result['status'] == 'override_enabled':
           print(f"Safety override enabled for {override_result['duration']} seconds")
           # Proceed with update...
       else:
           print("Safety override failed")
   else:
       print("System ready for updates")

JavaScript - Real-time Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket connection for real-time updates
   const ws = new WebSocket('ws://localhost/cloud-integration/ws/updates');
   
   ws.onopen = function() {
       console.log('Connected to OTA WebSocket');
   };
   
   ws.onmessage = function(event) {
       const message = JSON.parse(event.data);
       
       switch(message.type) {
           case 'progress_update':
               updateProgressBar(message.progress);
               updateStatusMessage(message.current_operation);
               updateETA(message.eta);
               break;
               
           case 'status_change':
               showNotification(message.message, message.severity);
               updateSystemStatus(message.system_status);
               break;
               
           case 'safety_alert':
               showSafetyAlert(message.constraint, message.message);
               if (message.action_required) {
                   showActionButtons(message.suggested_action);
               }
               break;
               
           case 'job_complete':
               showCompletionMessage(message.message, message.status);
               if (message.status === 'success') {
                   enableNextStep();
               }
               break;
               
           case 'system_event':
               if (message.event === 'reboot_scheduled') {
                   startRebootCountdown(message.countdown);
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
   
   // Update UI functions
   function updateProgressBar(percentage) {
       document.getElementById('progress-bar').style.width = percentage + '%';
       document.getElementById('progress-text').textContent = percentage.toFixed(1) + '%';
   }
   
   function updateStatusMessage(operation) {
       document.getElementById('current-operation').textContent = operation;
   }
   
   function showNotification(message, severity) {
       const notification = document.createElement('div');
       notification.className = `notification ${severity}`;
       notification.textContent = message;
       document.getElementById('notifications').appendChild(notification);
       
       setTimeout(() => notification.remove(), 5000);
   }

JavaScript - OTA Dashboard Control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Check for updates
   async function checkForUpdates() {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch('/cloud-integration/updates/check', {
               headers: {
                   'Authorization': `Bearer ${token}`
               }
           });
           
           const data = await response.json();
           renderUpdatesList(data.updates);
           return data.updates;
       } catch (error) {
           console.error('Error checking updates:', error);
           showError('Failed to check for updates');
       }
   }
   
   // Start download
   async function startDownload(updateId, version) {
       const token = localStorage.getItem('token');
       
       const requestBody = {
           update_id: updateId,
           version: version,
           target_partition: 'B'
       };
       
       try {
           const response = await fetch('/cloud-integration/updates/download', {
               method: 'POST',
               headers: {
                   'Authorization': `Bearer ${token}`,
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify(requestBody)
           });
           
           const result = await response.json();
           
           if (result.status === 'started') {
               startProgressMonitoring(result.job_id);
               return result.job_id;
           } else {
               throw new Error(result.message || 'Download failed to start');
           }
       } catch (error) {
           console.error('Error starting download:', error);
           showError('Failed to start download');
       }
   }
   
   // Monitor progress
   async function monitorProgress(jobId) {
       const token = localStorage.getItem('token');
       
       const interval = setInterval(async () => {
           try {
               const response = await fetch(`/cloud-integration/updates/progress?job_id=${jobId}`, {
                   headers: {
                       'Authorization': `Bearer ${token}`
                   }
               });
               
               const progress = await response.json();
               updateProgressUI(progress);
               
               if (progress.status === 'downloaded' || progress.status === 'failed') {
                   clearInterval(interval);
                   enableInstallButton(jobId);
               }
           } catch (error) {
               console.error('Error monitoring progress:', error);
           }
       }, 2000);
   }

Best Practices
--------------

Update Planning
~~~~~~~~~~~~~~~

1. **Schedule During Maintenance Windows**
   - Use maintenance window settings
   - Avoid peak operational hours
   - Coordinate with operations team

2. **Pre-update Checks**
   - Verify safety constraints
   - Check system resources
   - Create backups/snapshots

3. **Testing Strategy**
   - Test in staging first
   - Monitor post-update performance
   - Have rollback plan ready

Safety Considerations
~~~~~~~~~~~~~~~~~~~~~

1. **Never Override Without Authorization**
   - Always use PIN protection
   - Document override reasons
   - Limit override duration

2. **Monitor During Updates**
   - Watch for safety violations
   - Be ready to pause/stop
   - Have emergency procedures

3. **Post-update Verification**
   - Verify system functionality
   - Check critical operations
   - Monitor for issues

Troubleshooting
~~~~~~~~~~~~~~~

1. **Common Issues**
   - Network connectivity problems
   - Insufficient disk space
   - Signature verification failures
   - Safety constraint violations

2. **Resolution Steps**
   - Check logs for details
   - Verify network settings
   - Free up disk space
   - Review safety constraints

3. **Recovery Procedures**
   - Use rollback if available
   - Restore from snapshot
   - Contact support if needed

Security Considerations
-----------------------

Authentication & Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **PIN Protection**
   - Required for sensitive operations
   - Rate limiting on attempts
   - Audit logging

2. **Operation Limits**
   - Limit concurrent updates
   - Validate user permissions
   - Session timeouts

Data Protection
~~~~~~~~~~~~~~~

1. **Encryption**
   - Encrypt sensitive data
   - Secure credential storage
   - TLS for all communications

2. **Integrity Checking**
   - Verify update signatures
   - Check file checksums
   - Validate partition integrity

Audit & Compliance
~~~~~~~~~~~~~~~~~~

1. **Logging**
   - Log all OTA operations
   - Record safety overrides
   - Track user actions

2. **Reporting**
   - Generate compliance reports
   - Track update success rates
   - Monitor system health

System Requirements
-------------------

Minimum Requirements
~~~~~~~~~~~~~~~~~~~~

.. list-table:: System Requirements
   :widths: 40 60
   :header-rows: 1

   * - Component
     - Requirement
   * - **Disk Space**
     - 20GB free space
   * - **Memory**
     - 2GB RAM minimum
   * - **Network**
     - Stable internet connection
   * - **Partitions**
     - Dual-boot support required
   * - **Bootloader**
     - U-Boot 2024.01 or newer

Supported Formats
~~~~~~~~~~~~~~~~~

1. **Update Images**
   - Raw disk images (.img)
   - SquashFS images (.squashfs)
   - Compressed tar archives (.tar.gz)

2. **Signatures**
   - GPG signatures
   - X.509 certificates
   - SHA checksums

Performance Considerations
--------------------------

1. **Download Speed**
   - Average: 10-50 MB/s
   - Minimum: 1 MB/s required
   - Adjustable speed limits

2. **Installation Time**
   - Small updates: 2-5 minutes
   - Major updates: 10-30 minutes
   - Factory reset: 5-10 minutes

3. **System Impact**
   - CPU usage: 20-80% during install
   - Memory usage: 1-2GB
   - Network: 100% during download

Support & Troubleshooting
-------------------------

Getting Help
~~~~~~~~~~~~

1. **Documentation**
   - API reference
   - User guides
   - Troubleshooting guides

2. **Support Channels**
   - Email: ota-support@example.com
   - Phone: +1-800-OTA-HELP
   - Online portal

3. **Debug Information**
   - Generate debug bundles
   - Share system logs
   - Provide error codes

Common Solutions
~~~~~~~~~~~~~~~~

1. **Update Fails to Start**
   - Check safety constraints
   - Verify network connectivity
   - Ensure sufficient disk space

2. **Download Stalls**
   - Check network connection
   - Verify server accessibility
   - Try manual download

3. **Installation Fails**
   - Check system logs
   - Verify image integrity
   - Attempt rollback

Notes
-----

.. warning::

   **Critical Operations**: Some operations require system reboot

.. warning::

   **Data Loss Warning**: Factory reset erases all user data

.. important::

   **Safety First**: Always respect safety constraints

.. note::

   **Backup Regularly**: Create snapshots before major updates

.. note::

   **Test Updates**: Test in non-production first when possible

.. note::

   **Monitor Progress**: Use WebSocket for real-time updates
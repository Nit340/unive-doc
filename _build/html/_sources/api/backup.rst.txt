Backup & Recovery Management
============================

This document describes the backup and recovery management page and its related API endpoints for managing system backups, snapshots, recovery operations, and storage management.

Page Route (Frontend)
---------------------

.. http:get:: /backup-recovery

   **Description**: Renders the complete backup and recovery management page with all backup status, storage information, schedules, and recovery options embedded in the HTML.

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
        <title>Backup & Recovery - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-backup='{...}' data-storage='{...}' data-schedules='[...]' data-recovery='{...}'>
          <!-- Backup & Recovery page with:
               SECTION 1: Backup Management & Storage Status
               SECTION 2: Scheduled Backup Configuration
               SECTION 3: Snapshot Management
               SECTION 4: Disaster Recovery Operations
               SECTION 5: Import & Export Operations
               SECTION 6: Storage Management
               SECTION 7: Cleanup & Maintenance
               SECTION 8: Live Progress Monitor & Logs
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded backup and recovery data
   - All backup status, storage information, schedules, and recovery options are embedded
   - JavaScript reads this data and renders the complete backup management interface
   - No separate API calls needed on initial page load
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all backup and recovery operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Base Path: ``/api/v1/backup``

Backup System Status (Overview)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/status

   **Description**: Get overall backup system status, health, and configuration.
   
   **UI Element**: Security status badge at top of page
   
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
          "backup_system": "healthy",
          "security_score": 95,
          "last_successful_backup": "2025-03-15T02:00:00Z",
          "total_backups": 12,
          "storage_used": "1.4 GB",
          "components": {
            "scheduler": {"status": "enabled", "next_run": "2025-03-16T02:00:00Z"},
            "local_storage": {"status": "healthy", "used": "14%", "available": "8.6 GB"},
            "cloud_storage": {"status": "connected", "used": "320 MB"},
            "encryption": {"status": "enabled", "algorithm": "AES-256"}
          },
          "alerts": [
            {"level": "info", "message": "Scheduled backup in 15 hours"},
            {"level": "warning", "message": "MQTT certificate expiring in 14 days"}
          ]
        },
        "success": true,
        "message": "Backup system status retrieved successfully"
      }

Backup Management (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/list

   **Description**: Retrieve all backup files for table display.
   
   **UI Element**: SECTION 1 - Backup files table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **type** (string): Filter by type (full, config, pre-ota) *(optional)*
   * **date_from** (string): Filter from date (YYYY-MM-DD) *(optional)*
   * **date_to** (string): Filter to date (YYYY-MM-DD) *(optional)*
   * **limit** (integer): Number of results (default: 50) *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "backups": [
            {
              "id": 1,
              "name": "DailyBackup_20250612",
              "filename": "backup_2025-06-12_0200_full.tar.gz",
              "type": "Full System",
              "created_at": "2025-06-12T02:00:00Z",
              "size": 220000000,
              "size_display": "220 MB",
              "checksum": "a1b2c3d4e5f6g7h8",
              "location": "local",
              "encrypted": true,
              "integrity": "verified",
              "description": "Full System • Encrypted • Local",
              "components": ["system", "data", "config", "database"]
            }
          ],
          "total": 12,
          "storage_used": "1.4 GB",
          "storage_available": "8.6 GB"
        },
        "success": true,
        "message": "Backup list retrieved successfully"
      }

.. http:post:: /api/v1/backup/create

   **Description**: Create new manual backup from "Create Backup" modal.
   
   **UI Element**: SECTION 1 - "Create Backup" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "name": "pre_maintenance_backup_20250315",
        "type": "full",
        "components": ["system", "data", "config"],
        "destination": "local",
        "encrypt": true,
        "compression": "gzip",
        "verify_integrity": true,
        "description": "Manual backup before system maintenance"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "started",
          "backup_id": "backup_20250315_1430",
          "job_id": "JOB-001",
          "estimated_size": "220 MB",
          "estimated_duration": "5 minutes",
          "progress_url": "/api/v1/backup/progress/JOB-001"
        },
        "success": true,
        "message": "Backup creation started"
      }

.. http:get:: /api/v1/backup/download/{backup_id}

   **Description**: Download backup file.
   
   **UI Element**: SECTION 1 - "Download" button in backup table
   
   **Path Parameters**:

   * **backup_id** (string): Backup identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/octet-stream
      Content-Disposition: attachment; filename="backup_2025-06-12.tar.gz"
      
      [binary backup data]

.. http:post:: /api/v1/backup/delete

   **Description**: Delete backup file from table.
   
   **UI Element**: SECTION 1 - "Delete" button in backup table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "backup_id": 1,
        "reason": "Storage cleanup"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "deleted",
          "storage_freed": "220 MB"
        },
        "success": true,
        "message": "Backup deleted successfully"
      }

.. http:get:: /api/v1/backup/details/{backup_id}

   **Description**: Get detailed information about a specific backup.
   
   **UI Element**: SECTION 1 - "Restore Preview" button in backup table
   
   **Path Parameters**:

   * **backup_id** (string): Backup identifier

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
          "id": 1,
          "name": "DailyBackup_20250612",
          "filename": "backup_2025-06-12_0200_full.tar.gz",
          "type": "Full System",
          "created_at": "2025-06-12T02:00:00Z",
          "size": 220000000,
          "compressed_size": 150000000,
          "compression_ratio": 1.47,
          "checksum": "a1b2c3d4e5f6g7h8",
          "integrity": {
            "verified": true,
            "verification_time": "2025-06-12T03:00:00Z",
            "signature_valid": true
          },
          "encryption": {
            "enabled": true,
            "algorithm": "AES-256",
            "key_id": "key_001"
          },
          "components": {
            "system": {"included": true, "size": "120 MB"},
            "data": {"included": true, "size": "85 MB"},
            "config": {"included": true, "size": "15 MB"},
            "database": {"included": false, "size": "0 MB"}
          },
          "metadata": {
            "created_by": "scheduler",
            "gateway_id": "GW-001",
            "version": "v3.2.1",
            "description": "Scheduled nightly backup"
          },
          "storage_locations": [
            {
              "type": "local",
              "path": "/backups/2025-06-12/backup.tar.gz",
              "verified": true
            }
          ]
        },
        "success": true,
        "message": "Backup details retrieved successfully"
      }

Scheduled Backups (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/schedules

   **Description**: Get all backup schedules for form initialization.
   
   **UI Element**: SECTION 2 - Scheduled backups configuration form
   
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
          "schedules": [
            {
              "id": "schedule_nightly",
              "enabled": true,
              "name": "Nightly Full Backup",
              "type": "full",
              "frequency": "daily",
              "time": "02:00",
              "timezone": "UTC",
              "retention_count": 5,
              "encryption": true,
              "destination": "local",
              "auto_purge": true,
              "last_execution": "2025-06-12T02:00:00Z",
              "next_execution": "2025-06-13T02:00:00Z",
              "notifications": {
                "on_success": true,
                "on_failure": true
              }
            }
          ],
          "total_schedules": 1
        },
        "success": true,
        "message": "Backup schedules retrieved successfully"
      }

.. http:post:: /api/v1/backup/schedules/update

   **Description**: Save schedule configuration.
   
   **UI Element**: SECTION 2 - "Save Schedule" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "schedule_id": "schedule_nightly",
        "enabled": true,
        "frequency": "daily",
        "time": "02:00",
        "retention_count": 7,
        "type": "full",
        "destination": "local",
        "encryption": true,
        "auto_purge": true,
        "notifications": {
          "on_success": true,
          "on_failure": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "updated",
          "schedule_id": "schedule_nightly",
          "next_execution": "2025-06-13T02:00:00Z"
        },
        "success": true,
        "message": "Schedule updated successfully"
      }

.. http:post:: /api/v1/backup/schedules/toggle

   **Description**: Enable/disable a backup schedule.
   
   **UI Element**: SECTION 2 - Toggle switch in schedule header
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "schedule_id": "schedule_nightly",
        "enabled": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "updated",
          "schedule_id": "schedule_nightly",
          "enabled": false
        },
        "success": true,
        "message": "Schedule disabled"
      }

Snapshot Management (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/snapshots

   **Description**: Get all configuration snapshots.
   
   **UI Element**: SECTION 3 - Snapshots management table
   
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
          "snapshots": [
            {
              "id": 1,
              "name": "Pre_ACS_Update",
              "description": "Snapshot before ACS module update",
              "created_at": "2025-03-15T11:05:00Z",
              "size": "45 MB",
              "tags": ["ACS", "Zoning"],
              "components": ["acs_config", "zone_mappings"],
              "created_by": "admin",
              "status": "active"
            }
          ],
          "total": 3,
          "active": 3
        },
        "success": true,
        "message": "Snapshots retrieved successfully"
      }

.. http:post:: /api/v1/backup/snapshots/create

   **Description**: Create new configuration snapshot.
   
   **UI Element**: SECTION 3 - "Take Snapshot" modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "name": "Pre_ACS_Update_v2",
        "description": "Snapshot before ACS module v2.0 update",
        "tags": ["ACS", "Update", "Safety"],
        "components": ["acs_config", "zone_mappings", "user_permissions"],
        "include_settings": true,
        "include_data": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "created",
          "snapshot_id": 4,
          "name": "Pre_ACS_Update_v2",
          "size": "52 MB"
        },
        "success": true,
        "message": "Snapshot created successfully"
      }

.. http:post:: /api/v1/backup/snapshots/restore

   **Description**: Restore configuration from snapshot.
   
   **UI Element**: SECTION 3 - "Restore" button in snapshot table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "snapshot_id": 1,
        "components": ["acs_config", "zone_mappings"],
        "create_backup": true,
        "reason": "Rollback after failed update"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "restore_started",
          "restore_id": "RESTORE-001",
          "estimated_duration": "2 minutes",
          "pre_restore_backup_id": "backup_pre_restore_001",
          "progress_url": "/api/v1/backup/restore/progress/RESTORE-001"
        },
        "success": true,
        "message": "Snapshot restore started"
      }

Disaster Recovery (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/backup/recovery/initiate

   **Description**: Initiate disaster recovery from backup.
   
   **UI Element**: SECTION 4 - "Initiate Recovery" button in modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "backup_id": 1,
        "recovery_type": "full",
        "create_pre_recovery_snapshot": true,
        "module_filter": null,
        "reason": "System corruption detected",
        "confirmation": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "recovery_started",
          "recovery_id": "RECOVERY-001",
          "backup_id": 1,
          "estimated_duration": "15 minutes",
          "pre_recovery_snapshot_id": 5,
          "warning": "System will restart after recovery"
        },
        "success": true,
        "message": "Disaster recovery initiated"
      }

.. http:get:: /api/v1/backup/recovery/preview/{backup_id}

   **Description**: Get recovery preview and impact analysis.
   
   **UI Element**: SECTION 4 - Recovery preview in modal
   
   **Path Parameters**:

   * **backup_id** (string): Backup identifier

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
          "backup_id": 1,
          "backup_name": "DailyBackup_20250612",
          "recovery_type": "full",
          "impact_analysis": {
            "components_affected": ["system", "data", "config"],
            "estimated_downtime": "15 minutes",
            "restart_required": true,
            "data_loss_risk": "low",
            "pre_recovery_actions": ["stop_services", "create_snapshot"],
            "post_recovery_actions": ["verify_integrity", "restart_services"]
          },
          "warnings": [
            "All current data will be replaced",
            "Gateway will restart automatically"
          ],
          "recommendations": [
            "Ensure no users are connected",
            "Verify backup integrity before proceeding"
          ]
        },
        "success": true,
        "message": "Recovery preview generated"
      }

Import & Export Operations (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/backup/import

   **Description**: Import external backup file.
   
   **UI Element**: SECTION 5 - Import Backup modal file upload
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **file** (required): Backup file to import
   * **auto_restore** (boolean): Automatically restore after import *(optional, default: false)*
   * **create_backup_before** (boolean): Create backup before import *(optional, default: true)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "import_started",
          "import_id": "IMPORT-001",
          "filename": "external_backup.tar.gz",
          "validation_results": {
            "format": "valid",
            "signature": "verified",
            "version": "compatible",
            "checksum": "valid"
          },
          "backup_info": {
            "name": "ExternalBackup_20250610",
            "type": "full",
            "size": "210 MB",
            "created": "2025-06-10T14:30:00Z",
            "gateway_compatibility": "GW-001 v3.x"
          },
          "progress_url": "/api/v1/backup/import/progress/IMPORT-001"
        },
        "success": true,
        "message": "Backup import started"
      }

.. http:post:: /api/v1/backup/import/validate

   **Description**: Validate backup file before import.
   
   **UI Element**: SECTION 5 - File validation in Import modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **file** (required): Backup file to validate

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "validated",
          "validation": {
            "file_format": "valid",
            "signature": "verified",
            "checksum": "sha256:a1b2c3...",
            "version_compatibility": "compatible",
            "size": "210 MB",
            "type": "full",
            "created": "2025-06-10T14:30:00Z",
            "gateway_id": "GW-002",
            "components": ["system", "data", "config"]
          },
          "warnings": [],
          "recommendations": ["Create current backup before restore"]
        },
        "success": true,
        "message": "Backup validation completed"
      }

.. http:get:: /api/v1/backup/export/config

   **Description**: Export current configuration only.
   
   **UI Element**: SECTION 5 - "Export Configuration" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      Content-Disposition: attachment; filename="config_export_20250315.json"
      
      {
        "export_type": "configuration",
        "timestamp": "2025-03-15T14:30:00Z",
        "gateway": {
          "id": "GW-001",
          "version": "v3.2.1",
          "name": "Univa-GW-01"
        },
        "modules": {
          "acs": {},
          "zoning": {},
          "network": {}
        },
        "settings": {},
        "metadata": {
          "exported_by": "admin",
          "export_reason": "migration"
        }
      }

Storage Management (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/storage

   **Description**: Get storage configuration for all locations.
   
   **UI Element**: SECTION 6 - Storage Management modal
   
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
          "storage_locations": [
            {
              "id": "local",
              "type": "local",
              "name": "Local Storage",
              "enabled": true,
              "path": "/backups",
              "quota": "10 GB",
              "used": "1.4 GB",
              "available": "8.6 GB",
              "health": "good",
              "default": true
            },
            {
              "id": "aws",
              "type": "cloud",
              "name": "AWS S3 Bucket",
              "enabled": true,
              "provider": "AWS S3",
              "bucket": "company-backups",
              "region": "us-east-1",
              "used": "320 MB",
              "health": "connected",
              "last_sync": "2025-03-15T02:15:00Z"
            },
            {
              "id": "sftp",
              "type": "remote",
              "name": "SFTP Server",
              "enabled": false,
              "health": "offline",
              "configuration_required": true
            }
          ],
          "policies": {
            "default_location": "local",
            "replication_enabled": true,
            "encryption_default": true,
            "compression_default": "gzip"
          },
          "summary": {
            "total_backups": 12,
            "total_size": "1.4 GB",
            "largest_backup": "220 MB",
            "oldest_backup": "2025-06-01T02:00:00Z"
          }
        },
        "success": true,
        "message": "Storage configuration retrieved successfully"
      }

.. http:post:: /api/v1/backup/storage/update

   **Description**: Update storage configuration.
   
   **UI Element**: SECTION 6 - "Save Storage Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "storage_locations": {
          "local": {
            "quota": "20 GB",
            "cleanup_threshold": 80
          },
          "aws": {
            "enabled": true,
            "bucket": "new-company-backups"
          }
        },
        "policies": {
          "replication_enabled": true,
          "encryption_default": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "updated",
          "restart_required": false,
          "changes_applied": true
        },
        "success": true,
        "message": "Storage configuration updated successfully"
      }

.. http:post:: /api/v1/backup/storage/sync

   **Description**: Sync local backups to cloud storage.
   
   **UI Element**: SECTION 6 - "Sync to Cloud" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "source": "local",
        "destination": "aws",
        "backup_ids": ["all"],
        "verify_after_sync": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "sync_started",
          "sync_id": "SYNC-001",
          "source": "local",
          "destination": "aws",
          "backups_to_sync": 12,
          "estimated_size": "1.4 GB",
          "estimated_duration": "10 minutes",
          "progress_url": "/api/v1/backup/storage/sync/progress/SYNC-001"
        },
        "success": true,
        "message": "Storage sync started"
      }

Cleanup & Maintenance (SECTION 7)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/backup/cleanup

   **Description**: Run backup cleanup based on retention policies.
   
   **UI Element**: SECTION 7 - "Purge Old" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "dry_run": true,
        "storage_locations": ["local"],
        "retention_policy": "keep_last_5",
        "max_age_days": 30,
        "min_free_space": "5 GB",
        "reason": "Monthly cleanup"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "analyzed",
          "dry_run": true,
          "analysis": {
            "total_backups": 12,
            "eligible_for_removal": 7,
            "space_to_free": "850 MB",
            "backups_to_keep": 5,
            "protected_backups": 0
          },
          "action_required": true,
          "confirm_action": "cleanup_execute"
        },
        "success": true,
        "message": "Cleanup analysis completed"
      }

.. http:post:: /api/v1/backup/cleanup/execute

   **Description**: Execute cleanup after dry run analysis.
   
   **UI Element**: SECTION 7 - "Execute Cleanup" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "cleanup_plan_id": "ANALYSIS-001",
        "confirmation": true,
        "notify_on_completion": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "cleanup_started",
          "cleanup_id": "CLEANUP-001",
          "backups_to_remove": 7,
          "estimated_space": "850 MB",
          "estimated_duration": "2 minutes"
        },
        "success": true,
        "message": "Cleanup operation started"
      }

Live Progress Monitor & Logs (SECTION 8)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/backup/progress/{job_id}

   **Description**: Get progress of backup operation.
   
   **UI Element**: SECTION 8 - Progress modal
   
   **Path Parameters**:

   * **job_id** (string): Job identifier

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
          "job_id": "JOB-001",
          "operation": "backup_create",
          "status": "running",
          "progress": 65,
          "current_step": "compressing_data",
          "current_component": "database",
          "details": {
            "processed": "143 MB",
            "total": "220 MB",
            "speed": "15 MB/s",
            "eta": "00:01:30",
            "elapsed": "00:03:45"
          },
          "errors": 0,
          "warnings": 2
        },
        "success": true,
        "message": "Progress retrieved successfully"
      }

.. http:get:: /api/v1/backup/restore/progress/{restore_id}

   **Description**: Get progress of restore operation.
   
   **UI Element**: SECTION 8 - Restore progress display
   
   **Path Parameters**:

   * **restore_id** (string): Restore identifier

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
          "restore_id": "RESTORE-001",
          "operation": "recovery",
          "status": "running",
          "progress": 45,
          "current_step": "restoring_data",
          "current_component": "system",
          "details": {
            "files_processed": 4500,
            "files_total": 10000,
            "data_processed": "99 MB",
            "data_total": "220 MB",
            "eta": "00:05:30",
            "elapsed": "00:04:30"
          },
          "integrity_check": "pending",
          "restart_required": true
        },
        "success": true,
        "message": "Restore progress retrieved successfully"
      }

.. http:post:: /api/v1/backup/operation/cancel

   **Description**: Cancel running backup/restore operation.
   
   **UI Element**: SECTION 8 - "Cancel Operation" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "job_id": "JOB-001",
        "reason": "User requested cancellation"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "status": "cancelled",
          "job_id": "JOB-001",
          "partial_data_cleaned": true,
          "can_restart": true
        },
        "success": true,
        "message": "Operation cancelled successfully"
      }

.. http:get:: /api/v1/backup/statistics

   **Description**: Get backup system statistics.
   
   **UI Element**: SECTION 8 - Statistics dashboard
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Query Parameters**:

   * **period** (string): Time period (24h, 7d, 30d, 90d) *(optional)*

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "period": "30d",
          "summary": {
            "total_backups": 30,
            "successful": 29,
            "failed": 1,
            "success_rate": "96.7%",
            "total_size": "6.5 GB",
            "average_size": "216 MB",
            "storage_used": "1.4 GB",
            "storage_growth": "120 MB/week"
          },
          "by_type": {
            "full": {"count": 5, "size": "1.1 GB"},
            "config": {"count": 20, "size": "640 MB"},
            "snapshot": {"count": 5, "size": "85 MB"}
          },
          "timeline": [
            {"date": "2025-06-12", "successful": 1, "size": "220 MB"},
            {"date": "2025-06-11", "successful": 1, "size": "215 MB"}
          ],
          "storage_distribution": {
            "local": {"count": 30, "size": "1.4 GB"},
            "cloud": {"count": 12, "size": "320 MB"}
          }
        },
        "success": true,
        "message": "Statistics retrieved successfully"
      }

WebSocket Real-time Updates
---------------------------

Connection Endpoint
~~~~~~~~~~~~~~~~~~~
::

   ws://{gateway-ip}/api/v1/backup/ws
   wss://{gateway-ip}/api/v1/backup/ws (secure)

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

Backup Progress
^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "backup_progress",
     "job_id": "JOB-001",
     "backup_id": "backup_20250315_1430",
     "progress": 85,
     "current_step": "encrypting_data",
     "eta": "00:01:15",
     "timestamp": "2025-03-15T14:55:00Z"
   }

Restore Progress
^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "restore_progress",
     "restore_id": "RESTORE-001",
     "progress": 60,
     "current_step": "restoring_config",
     "eta": "00:03:45",
     "timestamp": "2025-03-15T15:30:00Z"
   }

Backup Completed
^^^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "backup_completed",
     "backup_id": "backup_20250315_1430",
     "name": "pre_maintenance_backup",
     "size": "220 MB",
     "duration": "00:05:15",
     "timestamp": "2025-03-15T14:35:15Z"
   }

Storage Alert
^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "storage_alert",
     "level": "warning",
     "storage": "local",
     "message": "Storage usage at 80%",
     "available": "2 GB",
     "used": "8 GB",
     "total": "10 GB",
     "timestamp": "2025-03-15T16:20:00Z"
   }

Schedule Event
^^^^^^^^^^^^^^
.. code-block:: json

   {
     "type": "schedule_event",
     "event": "execution_started",
     "schedule_id": "schedule_nightly",
     "timestamp": "2025-03-16T02:00:00Z"
   }

Route Summary
-------------

.. list-table:: Backup & Recovery Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/backup-recovery`` - Main backup page
     - Yes
   * - Status
     - GET
     - ``/api/v1/backup/status`` - System status
     - Yes
   * - Backup Mgmt
     - GET
     - ``/api/v1/backup/list`` - List backups
     - Yes
   * - Backup Mgmt
     - POST
     - ``/api/v1/backup/create`` - Create backup
     - Yes
   * - Backup Mgmt
     - GET
     - ``/api/v1/backup/download/{id}`` - Download backup
     - Yes
   * - Backup Mgmt
     - POST
     - ``/api/v1/backup/delete`` - Delete backup
     - Yes
   * - Backup Mgmt
     - GET
     - ``/api/v1/backup/details/{id}`` - Backup details
     - Yes
   * - Schedules
     - GET
     - ``/api/v1/backup/schedules`` - Get schedules
     - Yes
   * - Schedules
     - POST
     - ``/api/v1/backup/schedules/update`` - Update schedule
     - Yes
   * - Schedules
     - POST
     - ``/api/v1/backup/schedules/toggle`` - Toggle schedule
     - Yes
   * - Snapshots
     - GET
     - ``/api/v1/backup/snapshots`` - Get snapshots
     - Yes
   * - Snapshots
     - POST
     - ``/api/v1/backup/snapshots/create`` - Create snapshot
     - Yes
   * - Snapshots
     - POST
     - ``/api/v1/backup/snapshots/restore`` - Restore snapshot
     - Yes
   * - Recovery
     - POST
     - ``/api/v1/backup/recovery/initiate`` - Initiate recovery
     - Yes
   * - Recovery
     - GET
     - ``/api/v1/backup/recovery/preview/{id}`` - Recovery preview
     - Yes
   * - Import/Export
     - POST
     - ``/api/v1/backup/import`` - Import backup
     - Yes
   * - Import/Export
     - POST
     - ``/api/v1/backup/import/validate`` - Validate import
     - Yes
   * - Import/Export
     - GET
     - ``/api/v1/backup/export/config`` - Export config
     - Yes
   * - Storage
     - GET
     - ``/api/v1/backup/storage`` - Get storage config
     - Yes
   * - Storage
     - POST
     - ``/api/v1/backup/storage/update`` - Update storage
     - Yes
   * - Storage
     - POST
     - ``/api/v1/backup/storage/sync`` - Sync storage
     - Yes
   * - Cleanup
     - POST
     - ``/api/v1/backup/cleanup`` - Run cleanup
     - Yes
   * - Cleanup
     - POST
     - ``/api/v1/backup/cleanup/execute`` - Execute cleanup
     - Yes
   * - Progress
     - GET
     - ``/api/v1/backup/progress/{id}`` - Get progress
     - Yes
   * - Progress
     - GET
     - ``/api/v1/backup/restore/progress/{id}`` - Get restore progress
     - Yes
   * - Progress
     - POST
     - ``/api/v1/backup/operation/cancel`` - Cancel operation
     - Yes
   * - Statistics
     - GET
     - ``/api/v1/backup/statistics`` - Get statistics
     - Yes
   * - Real-time
     - WS
     - ``/api/v1/backup/ws`` - WebSocket for real-time updates
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /backup-recovery``
   - Server renders HTML with embedded backup and recovery data
   - All 8 sections populated with current data

2. **User views system status**:
   - Backup system health and security score
   - Last successful backup timestamp
   - Storage usage and availability

3. **User manages backups** (SECTION 1):
   - View all backup files in table
   - "Create Backup" button → ``POST /api/v1/backup/create``
   - "Download" button → ``GET /api/v1/backup/download/{id}``
   - "Delete" button → ``POST /api/v1/backup/delete``
   - "Restore Preview" → ``GET /api/v1/backup/details/{id}``

4. **User configures scheduled backups** (SECTION 2):
   - Schedule configuration form
   - "Save Schedule" → ``POST /api/v1/backup/schedules/update``
   - Toggle switch → ``POST /api/v1/backup/schedules/toggle``

5. **User manages snapshots** (SECTION 3):
   - Snapshots table with create/restore
   - "Take Snapshot" → ``POST /api/v1/backup/snapshots/create``
   - "Restore" button → ``POST /api/v1/backup/snapshots/restore``

6. **User performs disaster recovery** (SECTION 4):
   - "Initiate Recovery" → ``POST /api/v1/backup/recovery/initiate``
   - Recovery preview → ``GET /api/v1/backup/recovery/preview/{id}``

7. **User handles import/export** (SECTION 5):
   - Import backup file → ``POST /api/v1/backup/import``
   - Validate file → ``POST /api/v1/backup/import/validate``
   - Export configuration → ``GET /api/v1/backup/export/config``

8. **User manages storage** (SECTION 6):
   - Storage configuration → ``GET /api/v1/backup/storage``
   - Update settings → ``POST /api/v1/backup/storage/update``
   - Sync to cloud → ``POST /api/v1/backup/storage/sync``

9. **User runs cleanup** (SECTION 7):
   - Purge old backups → ``POST /api/v1/backup/cleanup``
   - Execute cleanup → ``POST /api/v1/backup/cleanup/execute``

10. **User monitors operations** (SECTION 8):
    - Progress monitoring → ``GET /api/v1/backup/progress/{id}``
    - Statistics dashboard → ``GET /api/v1/backup/statistics``
    - Cancel operations → ``POST /api/v1/backup/operation/cancel``
    - WebSocket connection → ``/api/v1/backup/ws``

Error Codes
-----------

.. list-table:: Backup & Recovery Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - BACKUP_FAILED
     - Backup operation failed
   * - RESTORE_FAILED
     - Restore operation failed
   * - STORAGE_FULL
     - Storage quota exceeded
   * - BACKUP_NOT_FOUND
     - Backup file not found
   * - VALIDATION_ERROR
     - Backup validation failed
   * - ENCRYPTION_ERROR
     - Encryption/decryption failed
   * - SCHEDULE_CONFLICT
     - Schedule configuration conflict
   * - IMPORT_FAILED
     - Backup import failed
   * - EXPORT_FAILED
     - Configuration export failed
   * - SNAPSHOT_FAILED
     - Snapshot operation failed
   * - CLEANUP_FAILED
     - Cleanup operation failed
   * - SYNC_FAILED
     - Storage sync failed
   * - OPERATION_IN_PROGRESS
     - Another operation is in progress

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Authorization: Bearer <token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages backups for the current gateway specified in `X-Gateway-ID` header.

Support Information
-------------------

- **Backup Support**: backup-support@univa.com
- **Disaster Recovery**: +1 (555) 789-0789
- **Support Hours**: 24/7 for recovery incidents
- **Documentation**: https://docs.univa.com/backup
- **Status Page**: https://status.univa.com/backup

---
*Document last updated: March 15, 2025*
*API Version: 1.0.0*
*Backup Module Version: 2.3.0*
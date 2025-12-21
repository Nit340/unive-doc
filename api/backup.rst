System Backup & Recovery API Documentation
==========================================

Overview
--------
Comprehensive REST APIs for managing system backups, recovery operations, scheduled snapshots, storage management, and disaster recovery. These APIs power the Backup & Recovery web interface.


1. Backup System Status
-----------------------

Get Backup Status
~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/status

   Get overall backup system status, health, and configuration.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Backup status retrieved successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Updates all status cards in the overview section.

2. Backup Management (SECTION 1)
---------------------------------

Get Backup List
~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/list

   Retrieve all backup files for table display.
   
   :query type: Filter by type (full, config, pre-ota) *(optional)*
   :query date_from: Filter from date (YYYY-MM-DD) *(optional)*
   :query date_to: Filter to date (YYYY-MM-DD) *(optional)*
   :query limit: Number of results (default: 50) *(optional)*
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Backup list retrieved successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Powers the backup files table in Section 1.

Create Backup
~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/create

   Create new manual backup from "Create Backup" modal.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
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
   
   :statuscode 201: Backup creation started
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "started",
        "backup_id": "backup_20250315_1430",
        "job_id": "JOB-001",
        "estimated_size": "220 MB",
        "estimated_duration": "5 minutes",
        "progress_url": "/cloud-integration/backup/progress/JOB-001"
      }

**UI Integration:** Initiates backup from modal and shows progress.

Download Backup
~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/download/{backup_id}

   Download backup file.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Backup file download
   :resheader Content-Type: application/octet-stream
   :resheader Content-Disposition: attachment; filename="backup_2025-06-12.tar.gz"
   
   **Response:**
   
   Binary backup file.

**UI Integration:** "Download" button in backup table actions.

Delete Backup
~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/delete

   Delete backup file from table.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "backup_id": 1,
        "reason": "Storage cleanup"
      }
   
   :statuscode 200: Backup deleted successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "deleted",
        "message": "Backup deleted successfully",
        "storage_freed": "220 MB"
      }

**UI Integration:** "Delete" button in backup table actions.

Get Backup Details
~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/details/{backup_id}

   Get detailed information about a specific backup.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Backup details retrieved successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** "Restore Preview" button in backup table.

3. Snapshot Management
----------------------

Get Snapshots
~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/snapshots

   Get all configuration snapshots.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Snapshots retrieved successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Updates snapshots card in overview.

Create Snapshot
~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/snapshots/create

   Create new configuration snapshot.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "name": "Pre_ACS_Update_v2",
        "description": "Snapshot before ACS module v2.0 update",
        "tags": ["ACS", "Update", "Safety"],
        "components": ["acs_config", "zone_mappings", "user_permissions"],
        "include_settings": true,
        "include_data": false
      }
   
   :statuscode 201: Snapshot created successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "created",
        "snapshot_id": 4,
        "name": "Pre_ACS_Update_v2",
        "size": "52 MB",
        "message": "Snapshot created successfully"
      }

**UI Integration:** "Take Snapshot" modal submission.

Restore Snapshot
~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/snapshots/restore

   Restore configuration from snapshot.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "snapshot_id": 1,
        "components": ["acs_config", "zone_mappings"],
        "create_backup": true,
        "reason": "Rollback after failed update"
      }
   
   :statuscode 200: Snapshot restore started
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "restore_started",
        "restore_id": "RESTORE-001",
        "estimated_duration": "2 minutes",
        "pre_restore_backup_id": "backup_pre_restore_001",
        "progress_url": "/cloud-integration/backup/restore/progress/RESTORE-001"
      }

4. Scheduled Backups (SECTION 2)
--------------------------------

Get Schedules
~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/schedules

   Get all backup schedules for form initialization.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Schedules retrieved successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Initializes all form fields in Section 2 accordion.

Update Schedule
~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/schedules/update

   Save schedule configuration.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
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
   
   :statuscode 200: Schedule updated successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "updated",
        "schedule_id": "schedule_nightly",
        "next_execution": "2025-06-13T02:00:00Z",
        "message": "Schedule updated successfully"
      }

**UI Integration:** "Save Schedule" button in Section 2.

Toggle Schedule Status
~~~~~~~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/schedules/toggle

   Enable/disable a backup schedule.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "schedule_id": "schedule_nightly",
        "enabled": false
      }
   
   :statuscode 200: Schedule status updated
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "updated",
        "schedule_id": "schedule_nightly",
        "enabled": false,
        "message": "Schedule disabled"
      }

**UI Integration:** Toggle switch in Section 2 header.

5. Disaster Recovery Operations
-------------------------------

Initiate Recovery
~~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/recovery/initiate

   Initiate disaster recovery from backup.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "backup_id": 1,
        "recovery_type": "full",
        "create_pre_recovery_snapshot": true,
        "module_filter": null,
        "reason": "System corruption detected",
        "confirmation": true
      }
   
   :statuscode 200: Recovery initiated
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "recovery_started",
        "recovery_id": "RECOVERY-001",
        "backup_id": 1,
        "estimated_duration": "15 minutes",
        "pre_recovery_snapshot_id": 5,
        "warning": "System will restart after recovery",
        "progress_url": "/cloud-integration/backup/recovery/progress/RECOVERY-001"
      }

**UI Integration:** "Initiate Recovery" button in Disaster Recovery modal.

Get Recovery Preview
~~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/recovery/preview/{backup_id}

   Get recovery preview and impact analysis.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Recovery preview generated
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Recovery preview in Disaster Recovery modal.

6. Import & Export Operations
------------------------------

Import Backup File
~~~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/import

   Import external backup file.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: multipart/form-data
   
   **Request:**
   
   .. sourcecode:: http
   
      POST /cloud-integration/backup/import HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="external_backup.tar.gz"
      Content-Type: application/gzip
      
      --form-data; name="auto_restore"
      
      false
      
      --form-data; name="create_backup_before"
      
      true
   
   :statuscode 200: Import started successfully
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
        "progress_url": "/cloud-integration/backup/import/progress/IMPORT-001"
      }

**UI Integration:** File upload in Import Backup modal.

Validate Import File
~~~~~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/import/validate

   Validate backup file before import.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: multipart/form-data
   
   **Request:**
   
   .. sourcecode:: http
   
      POST /cloud-integration/backup/import/validate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="backup.tar.gz"
      Content-Type: application/gzip
   
   :statuscode 200: Validation completed
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** File validation in Import Backup modal.

Export Configuration
~~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/export/config

   Export current configuration only.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Configuration exported
   :resheader Content-Type: application/json
   :resheader Content-Disposition: attachment; filename="config_export_20250315.json"
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "export_type": "configuration",
        "timestamp": "2025-03-15T14:30:00Z",
        "gateway": {
          "id": "GW-001",
          "version": "v3.2.1",
          "name": "Univa-GW-01"
        },
        "modules": {
          "acs": {...},
          "zoning": {...},
          "network": {...}
        },
        "settings": {...},
        "metadata": {
          "exported_by": "admin",
          "export_reason": "migration"
        }
      }

7. Storage Management
---------------------

Get Storage Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/storage

   Get storage configuration for all locations.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Storage configuration retrieved
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Storage Management modal content.

Update Storage Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/storage/update

   Update storage configuration.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
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
   
   :statuscode 200: Storage configuration updated
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "updated",
        "message": "Storage configuration updated",
        "restart_required": false,
        "changes_applied": true
      }

Sync to Cloud
~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/storage/sync

   Sync local backups to cloud storage.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "source": "local",
        "destination": "aws",
        "backup_ids": ["all"],
        "verify_after_sync": true
      }
   
   :statuscode 200: Sync started
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "sync_started",
        "sync_id": "SYNC-001",
        "source": "local",
        "destination": "aws",
        "backups_to_sync": 12,
        "estimated_size": "1.4 GB",
        "estimated_duration": "10 minutes",
        "progress_url": "/cloud-integration/backup/storage/sync/progress/SYNC-001"
      }

**UI Integration:** "Sync to Cloud" button in Storage Management modal.

8. Cleanup Operations
---------------------

Run Cleanup
~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/cleanup

   Run backup cleanup based on retention policies.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "dry_run": true,
        "storage_locations": ["local"],
        "retention_policy": "keep_last_5",
        "max_age_days": 30,
        "min_free_space": "5 GB",
        "reason": "Monthly cleanup"
      }
   
   :statuscode 200: Cleanup analysis completed
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** "Purge Old" button in Storage Management modal.

Execute Cleanup
~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/cleanup/execute

   Execute cleanup after dry run analysis.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "cleanup_plan_id": "ANALYSIS-001",
        "confirmation": true,
        "notify_on_completion": true
      }
   
   :statuscode 200: Cleanup executed
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "cleanup_started",
        "cleanup_id": "CLEANUP-001",
        "backups_to_remove": 7,
        "estimated_space": "850 MB",
        "estimated_duration": "2 minutes",
        "progress_url": "/cloud-integration/backup/cleanup/progress/CLEANUP-001"
      }

9. Progress Monitoring
----------------------

Get Backup Progress
~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/progress/{job_id}

   Get progress of backup operation.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Progress information retrieved
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** Progress modal updates.

Get Restore Progress
~~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/restore/progress/{restore_id}

   Get progress of restore operation.
   
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Restore progress retrieved
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

Cancel Operation
~~~~~~~~~~~~~~~~
.. http:post:: /cloud-integration/backup/operation/cancel

   Cancel running backup/restore operation.
   
   :reqheader Authorization: Bearer <token>
   :reqheader Content-Type: application/json
   
   **Request:**
   
   .. sourcecode:: json
   
      {
        "job_id": "JOB-001",
        "reason": "User requested cancellation"
      }
   
   :statuscode 200: Operation cancelled
   
   **Response:**
   
   .. sourcecode:: json
   
      {
        "status": "cancelled",
        "job_id": "JOB-001",
        "partial_data_cleaned": true,
        "can_restart": true
      }

10. Reporting & Statistics
--------------------------

Get Backup Statistics
~~~~~~~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/statistics

   Get backup system statistics.
   
   :query period: Time period (24h, 7d, 30d, 90d) *(optional)*
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Statistics retrieved
   
   **Response:**
   
   .. sourcecode:: json
   
      {
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
      }

**UI Integration:** "Generate Report" button in Storage Management modal.

Generate Report
~~~~~~~~~~~~~~~
.. http:get:: /cloud-integration/backup/report/generate

   Generate detailed backup report.
   
   :query format: Report format (pdf, html, csv) *(optional)*
   :query start_date: Start date (YYYY-MM-DD) *(optional)*
   :query end_date: End date (YYYY-MM-DD) *(optional)*
   :reqheader Authorization: Bearer <token>
   
   :statuscode 200: Report generated
   :resheader Content-Type: application/pdf
   :resheader Content-Disposition: attachment; filename="backup_report_20250315.pdf"
   
   **Response:**
   
   Report file in requested format.

11. WebSocket API
-----------------

Real-time Backup Updates
~~~~~~~~~~~~~~~~~~~~~~~~
.. websocket:: /ws/backup/updates

   Real-time updates for backup operations.
   
   **Message Types:**
   
   * **backup_progress** - Backup creation progress
   * **restore_progress** - Restore operation progress
   * **import_progress** - Import operation progress
   * **sync_progress** - Storage sync progress
   * **cleanup_progress** - Cleanup operation progress
   
   **Example Messages:**
   
   .. sourcecode:: json
   
      {
        "type": "backup_progress",
        "job_id": "JOB-001",
        "backup_id": "backup_20250315_1430",
        "progress": 85,
        "current_step": "encrypting_data",
        "eta": "00:01:15"
      }
      
      {
        "type": "restore_progress",
        "restore_id": "RESTORE-001",
        "progress": 60,
        "current_step": "restoring_config",
        "eta": "00:03:45"
      }
      
      {
        "type": "import_progress",
        "import_id": "IMPORT-001",
        "progress": 30,
        "current_step": "validating_data",
        "eta": "00:02:30"
      }

**UI Integration:** Auto-updates progress bars and status indicators.

Backup Notifications
~~~~~~~~~~~~~~~~~~~~
.. websocket:: /ws/backup/notifications

   Backup system notifications.
   
   **Message Types:**
   
   * **backup_completed** - Backup finished
   * **backup_failed** - Backup failed
   * **restore_completed** - Restore finished
   * **storage_alert** - Storage space warning
   * **schedule_event** - Schedule execution
   
   **Example Messages:**
   
   .. sourcecode:: json
   
      {
        "type": "backup_completed",
        "backup_id": "backup_20250315_1430",
        "name": "pre_maintenance_backup",
        "size": "220 MB",
        "duration": "00:05:15",
        "timestamp": "2025-03-15T14:35:15Z"
      }
      
      {
        "type": "storage_alert",
        "level": "warning",
        "storage": "local",
        "message": "Storage usage at 80%",
        "available": "2 GB",
        "used": "8 GB",
        "total": "10 GB"
      }
      
      {
        "type": "schedule_event",
        "event": "execution_started",
        "schedule_id": "schedule_nightly",
        "timestamp": "2025-03-16T02:00:00Z"
      }

**UI Integration:** Toast notifications and status badge updates.

System Health Updates
~~~~~~~~~~~~~~~~~~~~~
.. websocket:: /ws/backup/health

   Backup system health monitoring.
   
   **Message Types:**
   
   * **storage_health** - Storage health status
   * **service_status** - Backup service status
   * **connectivity_status** - Cloud connectivity
   
   **Example Messages:**
   
   .. sourcecode:: json
   
      {
        "type": "storage_health",
        "local": {
          "health": "good",
          "used": "1.4 GB",
          "available": "8.6 GB",
          "utilization": 14
        },
        "cloud": {
          "health": "connected",
          "last_sync": "2025-03-15T14:45:00Z"
        }
      }
      
      {
        "type": "service_status",
        "scheduler": "running",
        "encryption": "enabled",
        "compression": "enabled"
      }

**UI Integration:** Auto-updates status cards in overview.

Missing WebSocket Requirements
------------------------------

The following real-time features need WebSocket integration:

1. **Backup Progress Updates** - Current progress modal uses simulated progress
2. **Storage Health Monitoring** - Storage cards show static data
3. **Schedule Execution Events** - Schedule status doesn't update in real-time
4. **Notification Streaming** - Toast notifications are triggered manually
5. **System Health Alerts** - Health status is poll-based

Error Responses
---------------

All endpoints return standardized error responses:

.. sourcecode:: json

   {
     "error": {
       "code": "BACKUP_FAILED",
       "message": "Backup creation failed",
       "details": "Insufficient storage space",
       "timestamp": "2025-03-15T14:35:00Z"
     }
   }

Common error codes:

- ``BACKUP_FAILED`` - Backup operation failed
- ``RESTORE_FAILED`` - Restore operation failed
- ``STORAGE_FULL`` - Storage quota exceeded
- ``BACKUP_NOT_FOUND`` - Backup file not found
- ``VALIDATION_ERROR`` - Backup validation failed
- ``ENCRYPTION_ERROR`` - Encryption/decryption failed
- ``SCHEDULE_CONFLICT`` - Schedule configuration conflict

Rate Limiting
-------------

- **Backup Operations:** 5 concurrent operations maximum
- **File Uploads:** 5GB max file size, 1 concurrent upload
- **API Endpoints:** 60 requests/minute per IP
- **WebSocket Connections:** 3 concurrent connections per user
- **Report Generation:** 1 report/minute maximum

Authentication
--------------

All endpoints require Bearer token authentication:

.. sourcecode:: http

   Authorization: Bearer <jwt_token>

Tokens are obtained via the authentication service and include user roles and permissions.

Versioning
----------

API version is included in the URL path:

::

   /cloud-integration/v1/backup/...

Current version: **v1**


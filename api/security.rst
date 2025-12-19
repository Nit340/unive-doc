Over-the-Air Updates (OTA)
==========================

Overview
--------

The OTA API allows you to manage firmware updates for connected IoT devices and gateways. This API supports both cloud-initiated and manual updates with safety constraints and progress monitoring.

.. contents:: Table of Contents
   :depth: 2
   :local:

API Endpoints
-------------

### Check for Updates

.. http:get:: /api/v1/ota/updates/check

   Check for available updates from the cloud update server.
   
   **Response**:
   
   .. code-block:: json
   
      {
        "available": true,
        "updates": [
          {
            "id": "update_12345",
            "version": "v5.0.5",
            "description": "Security patches and performance improvements",
            "size": 452000000,
            "release_date": "2025-03-10T14:30:00Z",
            "critical": true,
            "requires_reboot": true
          }
        ]
      }

### Download Update

.. http:post:: /api/v1/ota/updates/download

   Start downloading an available update.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "update_id": "update_12345",
        "target_partition": "B",
        "priority": "normal",
        "verify_signature": true
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "started",
        "job_id": "OTA-JOB-8821X",
        "estimated_duration": "00:05:30",
        "message": "Download started successfully"
      }

### Monitor Progress

.. http:get:: /api/v1/ota/updates/progress

   Get the current progress of an ongoing update job.
   
   **Query Parameters**:
   
   * ``job_id`` - The ID of the job to monitor
   
   **Response**:
   
   .. code-block:: json
   
      {
        "job_id": "OTA-JOB-8821X",
        "status": "downloading",
        "progress": {
          "percentage": 65.5,
          "bytes_downloaded": 295000000,
          "bytes_total": 452000000,
          "speed": "5.2MB/s"
        },
        "current_operation": "Downloading update package",
        "eta": "00:02:15"
      }

### Install Update

.. http:post:: /api/v1/ota/updates/install

   Install a downloaded update.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "job_id": "OTA-JOB-8821X",
        "strategy": "seamless",
        "reboot_after": true,
        "rollback_on_failure": true
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "installing",
        "message": "Update installation in progress",
        "requires_reboot": true,
        "estimated_duration": "00:03:00"
      }

### Cancel Update

.. http:post:: /api/v1/ota/updates/cancel

   Cancel an ongoing update operation.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "job_id": "OTA-JOB-8821X",
        "reason": "User requested cancellation"
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "cancelled",
        "message": "Update operation cancelled successfully"
      }

### Rollback Update

.. http:post:: /api/v1/ota/updates/rollback

   Rollback to the previous version.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "target_version": "v5.0.4",
        "reason": "New version has compatibility issues",
        "pin": "7392"
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "rollback_started",
        "message": "Rolling back to version v5.0.4",
        "estimated_duration": "00:04:00"
      }

### Factory Reset

.. http:post:: /api/v1/ota/factory-reset

   Perform a factory reset of the system.
   
   **Warning**: This operation will erase all user data and settings.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "pin": "7392",
        "backup_settings": true,
        "confirmation": "I understand this will erase all data"
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "reset_scheduled",
        "message": "Factory reset scheduled for next boot",
        "reboot_required": true
      }

### Upload Custom Image

.. http:post:: /api/v1/ota/upload

   Upload a custom firmware image for deployment.
   
   **Content-Type**: multipart/form-data
   
   **Form Parameters**:
   
   * ``file`` - The firmware image file
   * ``version`` - Version label for the image
   * ``description`` - Optional description
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "uploaded",
        "file": {
          "name": "custom-v5.1.0.img",
          "size": 452000000,
          "version": "v5.1.0-custom",
          "checksum": "sha256:abc123..."
        },
        "verification_required": true
      }

### Verify Uploaded Image

.. http:post:: /api/v1/ota/upload/verify

   Verify the integrity and compatibility of an uploaded image.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "filename": "custom-v5.1.0.img",
        "verify_signature": true,
        "check_compatibility": true
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "verified",
        "file": {
          "name": "custom-v5.1.0.img",
          "version": "v5.1.0-custom",
          "signature_valid": true,
          "compatible": true,
          "partition_layout": "standard"
        },
        "ready_for_deployment": true
      }

### Deploy Custom Image

.. http:post:: /api/v1/ota/upload/deploy

   Deploy a verified custom image.
   
   **Request**:
   
   .. code-block:: json
   
      {
        "filename": "custom-v5.1.0.img",
        "target_partition": "B",
        "strategy": "seamless",
        "pin": "7392"
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "deployment_started",
        "deployment_id": "DEP-12345",
        "message": "Custom image deployment in progress"
      }

### Safety Status

.. http:get:: /api/v1/ota/safety/status

   Check safety constraints that may block updates.
   
   **Response**:
   
   .. code-block:: json
   
      {
        "update_allowed": false,
        "constraints": [
          {
            "name": "crane_motion",
            "status": "violated",
            "message": "Crane movement detected",
            "since": "2025-03-15T10:25:30Z"
          },
          {
            "name": "high_temperature",
            "status": "ok",
            "message": "Temperature within limits"
          }
        ],
        "maintenance_window": {
          "active": false,
          "next_window": "2025-03-16T02:00:00Z"
        }
      }

### Safety Override

.. http:post:: /api/v1/ota/safety/override

   Temporarily override safety constraints (requires PIN).
   
   **Request**:
   
   .. code-block:: json
   
      {
        "enable": true,
        "pin": "7392",
        "reason": "Emergency security update",
        "duration": 1800,
        "acknowledge_risks": true
      }
   
   **Response**:
   
   .. code-block:: json
   
      {
        "status": "override_enabled",
        "duration": 1800,
        "expires_at": "2025-03-15T11:00:30Z",
        "constraints_overridden": ["crane_motion"],
        "warning": "Safety constraints overridden. Proceed with caution."
      }

### Get System Status

.. http:get:: /api/v1/ota/system/status

   Get current system status and partition information.
   
   **Response**:
   
   .. code-block:: json
   
      {
        "gateway": {
          "name": "iot-gateway-01",
          "model": "IGW-5000",
          "serial": "SN123456789"
        },
        "partitions": {
          "active": {
            "name": "A",
            "version": "v5.0.4",
            "boot_count": 42,
            "last_boot": "2025-03-15T08:00:00Z"
          },
          "inactive": {
            "name": "B",
            "version": "v5.0.3",
            "boot_count": 38
          }
        },
        "resources": {
          "disk_free": 15000000000,
          "memory_free": 800000000,
          "cpu_load": 0.25
        },
        "last_update": {
          "version": "v5.0.4",
          "timestamp": "2025-03-10T14:30:00Z",
          "status": "success"
        }
      }

Real-time Updates
-----------------

Connect to the WebSocket endpoint for real-time update notifications:

.. code-block:: bash

   ws://your-gateway/api/v1/ota/ws

### Message Types

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

### Error Response Format

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

### Common Error Codes

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

### Python - Complete Update Flow

.. code-block:: python

   import requests
   import time
   
   # Check gateway status
   headers = {"Authorization": "Bearer your_token"}
   
   status_response = requests.get(
       "https://api.example.com/v1/ota/system/status",
       headers=headers
   )
   
   status = status_response.json()
   print(f"Gateway: {status['gateway']['name']}")
   print(f"Active Partition: {status['partitions']['active']['version']}")
   
   # Check for updates
   updates_response = requests.get(
       "https://api.example.com/v1/ota/updates/check",
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
           "https://api.example.com/v1/ota/updates/download",
           json=download_request,
           headers=headers
       )
       
       download_result = download_response.json()
       job_id = download_result['job_id']
       print(f"Download started: {job_id}")
       
       # Monitor progress
       while True:
           progress_response = requests.get(
               f"https://api.example.com/v1/ota/updates/progress?job_id={job_id}",
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
           "https://api.example.com/v1/ota/updates/install",
           json=install_request,
           headers=headers
       )
       
       print("Update installation started. System will reboot.")

### Python - Manual Image Upload

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
           "https://api.example.com/v1/ota/upload/verify",
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
           "https://api.example.com/v1/ota/upload/deploy",
           json=deploy_request,
           headers={"Authorization": "Bearer your_token"}
       )
       
       print(f"Deployment started: {deploy_response.json()['deployment_id']}")
   else:
       print(f"Verification failed: {verify_result.get('error', 'Unknown error')}")

### Python - Safety Override and Update

.. code-block:: python

   # Check safety status
   safety_response = requests.get(
       "https://api.example.com/v1/ota/safety/status",
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
           "https://api.example.com/v1/ota/safety/override",
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

### JavaScript - Real-time Monitoring

.. code-block:: javascript

   // WebSocket connection for real-time updates
   const ws = new WebSocket('ws://localhost/api/v1/ota/ws');
   
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

### JavaScript - OTA Dashboard Control

.. code-block:: javascript

   // Check for updates
   async function checkForUpdates() {
       const token = localStorage.getItem('token');
       
       try {
           const response = await fetch('/api/v1/ota/updates/check', {
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
           const response = await fetch('/api/v1/ota/updates/download', {
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
               const response = await fetch(`/api/v1/ota/updates/progress?job_id=${jobId}`, {
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

### Update Planning

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

### Safety Considerations

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

### Troubleshooting

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

### Authentication & Authorization

1. **PIN Protection**
   - Required for sensitive operations
   - Rate limiting on attempts
   - Audit logging

2. **Operation Limits**
   - Limit concurrent updates
   - Validate user permissions
   - Session timeouts

### Data Protection

1. **Encryption**
   - Encrypt sensitive data
   - Secure credential storage
   - TLS for all communications

2. **Integrity Checking**
   - Verify update signatures
   - Check file checksums
   - Validate partition integrity

### Audit & Compliance

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

### Minimum Requirements

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

### Supported Formats

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

### Getting Help

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

### Common Solutions

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

- **Critical Operations**: Some operations require system reboot
- **Data Loss Warning**: Factory reset erases all user data
- **Safety First**: Always respect safety constraints
- **Backup Regularly**: Create snapshots before major updates
- **Test Updates**: Test in non-production first when possible
- **Monitor Progress**: Use WebSocket for real-time updates
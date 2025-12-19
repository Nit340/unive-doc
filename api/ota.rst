
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
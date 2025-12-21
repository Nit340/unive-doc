Licensing & Subscriptions API
=============================

API for managing gateway licenses, subscriptions, feature entitlements, trials, and cloud integration.

Base Path: /api/v1/license

.. contents:: Table of Contents
   :depth: 3
   :local:

Overview
--------

This API handles all licensing operations for the current gateway only. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

License Overview
~~~~~~~~~~~~~~~~

Get License Overview
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/overview

   Get current license status and overview information.
   
   **UI Element**: SECTION 1: License Overview cards
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "license_type": "pro",
        "license_tier": "Pro License",
        "license_status": "active",
        "subscription_status": "active",
        "auto_renewal": true,
        "activation_date": "2024-03-15",
        "expiry_date": "2026-08-12",
        "days_remaining": 615,
        "gateway_id": "GW-3920A9",
        "license_id": "LIC-PRO-2024-3920A9",
        "issued_to": "Innospace Solutions",
        "issued_on": "2024-03-15",
        "last_sync": "2024-03-20T14:25:22Z",
        "sync_status": "success"
      }

Sync with Cloud
^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/sync

   Synchronize licenses with cloud.
   
   **UI Element**: SECTION 1: "Sync with Cloud" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Request**:

   .. sourcecode:: http

      POST /api/v1/license/sync HTTP/1.1
      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "synced": true,
        "sync_time": "2024-03-20T14:30:22Z",
        "message": "Cloud sync completed successfully"
      }

Deactivate License
^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/deactivate

   Deactivate current license.
   
   **UI Element**: SECTION 1: "Deactivate" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "reason": "decommissioning"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "deactivated": true,
        "message": "License deactivated successfully"
      }

Feature Access Matrix
~~~~~~~~~~~~~~~~~~~~~

Get Features
^^^^^^^^^^^^

.. http:get:: /api/v1/license/features

   Get all features with their status.
   
   **UI Element**: SECTION 2: Feature Access Matrix table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "features": [
          {
            "id": "craneiq_pro",
            "name": "CraneIQ Pro",
            "category": "Safety",
            "status": "active",
            "source": "cloud",
            "expiry": "2026-08-12",
            "limits": "Full Access"
          },
          {
            "id": "advanced_analytics",
            "name": "Advanced Analytics",
            "category": "Analytics",
            "status": "trial",
            "source": "trial",
            "expiry": "2024-04-10",
            "limits": "Limited Access"
          }
        ],
        "total_features": 12,
        "active_count": 8
      }

Toggle Feature
^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/features/{feature_id}/toggle

   Enable/disable a feature.
   
   **UI Element**: SECTION 2: Enable/Disable buttons
   
   **Path Parameters**:

   * **feature_id** (string): Feature identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enabled": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "toggled": true,
        "feature_id": "craneiq_pro",
        "new_status": "inactive",
        "message": "Feature disabled successfully"
      }

License Key Management
~~~~~~~~~~~~~~~~~~~~~~

Validate License Key
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/validate

   Validate and activate a license key.
   
   **UI Element**: SECTION 3: "Validate & Activate" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "license_key": "XXXX-XXXX-XXXX-XXXX",
        "activation_mode": "online"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "valid": true,
        "activated": true,
        "license_type": "pro",
        "message": "License activated successfully"
      }

Get Current License Info
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/current

   Get current license information.
   
   **UI Element**: SECTION 3: Current License card
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "issued_to": "Innospace Solutions",
        "issued_on": "2024-03-15",
        "activation_count": "1 of 3"
      }

Generate Demo Key
^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/demo-key

   Generate a demo license key.
   
   **UI Element**: SECTION 3: Magic wand icon
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "demo_key": "DEMO-XXXX-XXXX-XXXX",
        "expires_in": "30 days"
      }

Export License
^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/export

   Export current license data.
   
   **UI Element**: SECTION 3: "Export" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "download_url": "/api/v1/license/export/download/license_export.json",
        "filename": "license_export.json",
        "expires_in": 3600
      }

Import License
^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/import

   Import license from file.
   
   **UI Element**: SECTION 3: "Import File" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /api/v1/license/import HTTP/1.1
      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="license.lic"

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "imported": true,
        "license_id": "LIC-PRO-2024-3920A9",
        "message": "License imported successfully"
      }

Cloud Subscription
~~~~~~~~~~~~~~~~~~

Get Cloud Status
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/cloud

   Get cloud subscription status.
   
   **UI Element**: SECTION 4: Cloud Subscription Link section
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "linked": true,
        "account_email": "admin@innospace.com",
        "subscription_tier": "Enterprise",
        "auto_renewal": true,
        "sync_status": {
          "last_sync": "2024-03-20T14:25:22Z",
          "next_sync": "2024-03-20T20:30:00Z",
          "sync_enabled": true
        }
      }

Unlink Cloud
^^^^^^^^^^^^

.. http:post:: /api/v1/license/cloud/unlink

   Unlink cloud account.
   
   **UI Element**: SECTION 4: "Unlink" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "confirm": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "unlinked": true,
        "message": "Cloud account unlinked successfully"
      }

Get Sync Settings
^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/cloud/sync-settings

   Get cloud sync settings.
   
   **UI Element**: SECTION 4: "Sync Settings" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "auto_sync": true,
        "sync_frequency": "every_6_hours",
        "next_sync": "2024-03-20T20:30:00Z"
      }

Update Sync Settings
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/cloud/sync-settings

   Update cloud sync settings.
   
   **UI Element**: SECTION 4: Sync settings modal save
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "auto_sync": true,
        "sync_frequency": "daily"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Sync settings updated successfully"
      }

Trials Management
~~~~~~~~~~~~~~~~~

Get Active Trials
^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/trials

   Get all active trials.
   
   **UI Element**: SECTION 5: Active Trials cards
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "trials": [
          {
            "id": "TRIAL-20240315-001",
            "name": "CraneIQ Pro Trial",
            "days_left": 10,
            "start_date": "2024-03-15",
            "end_date": "2024-04-15",
            "progress": 70
          }
        ]
      }

Get Available Trials
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/trials/available

   Get available trials.
   
   **UI Element**: SECTION 5: Available Trials list
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "trials": [
          {
            "id": "rule_engine_pro",
            "name": "Rule Engine Pro",
            "trial_days": 14
          }
        ]
      }

Start Trial
^^^^^^^^^^^

.. http:post:: /api/v1/license/trials/start

   Start a trial.
   
   **UI Element**: SECTION 5: "Start 14-day Trial" buttons
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "trial_id": "rule_engine_pro"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "started": true,
        "trial_id": "TRIAL-20240320-001",
        "end_date": "2024-04-03",
        "message": "Trial started successfully"
      }

Extend Trial
^^^^^^^^^^^^

.. http:post:: /api/v1/license/trials/{trial_id}/extend

   Extend a trial.
   
   **UI Element**: SECTION 5: "Extend Trial" button
   
   **Path Parameters**:

   * **trial_id** (string): Trial identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "days": 7
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "extended": true,
        "new_end_date": "2024-04-22",
        "message": "Trial extended by 7 days"
      }

Upgrade Trial
^^^^^^^^^^^^^

.. http:post:: /api/v1/license/trials/{trial_id}/upgrade

   Upgrade trial to full license.
   
   **UI Element**: SECTION 5: "Upgrade Now" button
   
   **Path Parameters**:

   * **trial_id** (string): Trial identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "license_tier": "pro"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "upgraded": true,
        "payment_url": "https://checkout.innospace.com/upgrade-abc123",
        "message": "Upgrade initiated successfully"
      }

Cancel Trial
^^^^^^^^^^^^

.. http:post:: /api/v1/license/trials/{trial_id}/cancel

   Cancel a trial.
   
   **UI Element**: SECTION 5: "Cancel Trial" button
   
   **Path Parameters**:

   * **trial_id** (string): Trial identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "reason": "no_longer_needed"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cancelled": true,
        "message": "Trial cancelled successfully"
      }

License Logs
~~~~~~~~~~~~

Get License Logs
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/logs

   Get license activity logs.
   
   **UI Element**: SECTION 6: License Change Log
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "logs": [
          {
            "timestamp": "2024-03-20 14:30:22",
            "type": "sync",
            "message": "Synced with cloud - All entitlements updated",
            "user": "System"
          }
        ],
        "total": 42
      }

Footer Actions
~~~~~~~~~~~~~~

Validate All Licenses
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/validate-all

   Validate all licenses.
   
   **UI Element**: Footer: "Validate All Licenses" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "validated": true,
        "message": "All licenses validated successfully"
      }

Create Backup
^^^^^^^^^^^^^

.. http:post:: /api/v1/license/backup

   Create license backup.
   
   **UI Element**: Footer: "Backup License Data" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "format": "json",
        "encrypt": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backup_id": "BACKUP-20240320-001",
        "download_url": "/api/v1/license/backup/download/BACKUP-20240320-001.json",
        "message": "Backup created successfully"
      }

Reset Trials
^^^^^^^^^^^^

.. http:post:: /api/v1/license/trials/reset

   Reset all trials.
   
   **UI Element**: Footer: "Reset Trials" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "confirm": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "message": "All trials reset successfully"
      }

Save Settings
^^^^^^^^^^^^^

.. http:post:: /api/v1/license/settings

   Save license settings.
   
   **UI Element**: Footer: "Save License Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "auto_sync": true,
        "notify_on_expiry": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "saved": true,
        "message": "Settings saved successfully"
      }

Modal Actions
~~~~~~~~~~~~~

Get Renewal Options
^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/license/renewal-options

   Get renewal options (opens modal).
   
   **UI Element**: SECTION 1: "Renew License" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "options": [
          {
            "period": "1_year",
            "price": 499,
            "currency": "USD"
          },
          {
            "period": "3_years",
            "price": 1199,
            "currency": "USD",
            "savings": "20%"
          }
        ]
      }

Initiate Transfer
^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/license/transfer

   Initiate license transfer (opens modal).
   
   **UI Element**: SECTION 1: "Transfer License" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "target_gateway": "GW-2024-00234",
        "transfer_type": "copy"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "transfer_initiated": true,
        "support_ticket": "SUP-20240320-001",
        "message": "Transfer request submitted"
      }

Get Help
^^^^^^^^

.. http:get:: /api/v1/license/help

   Get help documentation.
   
   **UI Element**: Header: "Help" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "title": "Licensing & Subscriptions Help",
        "content": "This page manages all gateway-level feature entitlements...",
        "contact": {
          "email": "licenses@innospace.com",
          "phone": "+1 (555) 123-4567"
        }
      }

Error Codes
-----------

.. list-table:: Simplified Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - LICENSE_INVALID
     - Invalid license key
   * - LICENSE_EXPIRED
     - License has expired
   * - LICENSE_MAX_ACTIVATIONS
     - Maximum activations reached
   * - TRIAL_NOT_AVAILABLE
     - Trial not available
   * - TRIAL_MAX_REACHED
     - Maximum concurrent trials reached
   * - CLOUD_SYNC_FAILED
     - Cloud sync failed
   * - CLOUD_NOT_LINKED
     - Cloud account not linked
   * - FEATURE_NOT_LICENSED
     - Feature not licensed
   * - TRANSFER_NOT_ALLOWED
     - Transfer not allowed
   * - PAYMENT_REQUIRED
     - Payment required
   * - BACKUP_FAILED
     - Backup creation failed

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
     "error": null,
     "success": true
   }

Error responses:

.. code-block:: json

   {
     "data": null,
     "error": {
       "code": "LICENSE_INVALID",
       "message": "Invalid license key"
     },
     "success": false
   }
Licensing & Subscriptions API
=============================

This document describes the licensing management page and its related API endpoints for managing gateway licenses, subscriptions, feature entitlements, trials, and cloud integration.

Page Route (Frontend)
---------------------

.. http:get:: /license-management

   **Description**: Renders the complete licensing and subscriptions management page with all license data, features, trials, and cloud status embedded in the HTML.

   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9

   **Response**::

      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Licensing & Subscriptions - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-license='{...}' data-features='[...]' data-trials='[...]' data-cloud='{...}'>
          <!-- Licensing page with:
               SECTION 1: License Overview cards
               SECTION 2: Feature Access Matrix table
               SECTION 3: License Key Management
               SECTION 4: Cloud Subscription Link
               SECTION 5: Trials Management
               SECTION 6: License Change Log
               FOOTER: Validation, Backup, Save actions
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded license data
   - All license information, feature access, active trials, and cloud status are embedded
   - JavaScript reads this data and renders the complete licensing interface
   - No separate API calls needed on initial page load
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**::

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all licensing operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

License Overview (SECTION 1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/overview

   **Description**: Get current license status and overview information.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "license_type": "pro",
        "license_status": "active",
        "activation_date": "2024-03-15",
        "expiry_date": "2026-08-12",
        "days_remaining": 615,
        "gateway_id": "GW-3920A9",
        "auto_renewal": true
      }

.. http:post:: /api/license-management/sync

   **Description**: Synchronize licenses with cloud.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "synced": true,
        "sync_time": "2024-03-20T14:30:22Z",
        "message": "Cloud sync completed successfully"
      }

.. http:post:: /api/license-management/deactivate

   **Description**: Deactivate current license.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "deactivated": true,
        "message": "License deactivated successfully"
      }

.. http:post:: /api/license-management/transfer

   **Description**: Transfer license to another gateway.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/license-management/transfer HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "target_gateway_id": "GW-1234B5",
        "transfer_type": "copy"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "transferred": true,
        "transfer_id": "TRANSFER-20240320-001",
        "message": "License transfer initiated successfully"
      }

Feature Access Matrix (SECTION 2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/features

   **Description**: Get all features with their status.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "features": [
          {
            "id": "craneiq_pro",
            "name": "CraneIQ Pro",
            "category": "Safety",
            "status": "active",
            "expiry": "2026-08-12"
          }
        ],
        "total_features": 12,
        "active_count": 8
      }

.. http:post:: /api/license-management/features/{feature_id}/toggle

   **Description**: Enable/disable a feature.
   
   **Path Parameters**:
   
   * **feature_id** (string): Feature identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "toggled": true,
        "feature_id": "craneiq_pro",
        "new_status": "inactive",
        "message": "Feature disabled successfully"
      }

License Key Management (SECTION 3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/license-management/validate

   **Description**: Validate and activate a license key.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Request**::

      POST /api/license-management/validate HTTP/1.1
      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
      
      {
        "license_key": "XXXX-XXXX-XXXX-XXXX",
        "activation_mode": "online"
      }
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "valid": true,
        "activated": true,
        "license_type": "pro",
        "message": "License activated successfully"
      }

.. http:get:: /api/license-management/license-info

   **Description**: Get current license information for display in UI.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "issued_to": "Innospace Solutions",
        "issued_on": "2024-03-15",
        "activation_count": "1 of 3 devices"
      }

.. http:post:: /api/license-management/import

   **Description**: Import license from file.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "imported": true,
        "message": "License imported successfully"
      }

.. http:get:: /api/license-management/export

   **Description**: Export current license data.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      Content-Disposition: attachment; filename="license_export.json"
      
      {
        "license_data": { ... }
      }

.. http:get:: /api/license-management/license-history

   **Description**: Get license activation history.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "history": [
          {
            "timestamp": "2024-03-15 10:30:00",
            "license_key": "XXXX-XXXX-XXXX-XXXX",
            "license_type": "pro",
            "action": "activated",
            "gateway_id": "GW-3920A9"
          }
        ],
        "total_activations": 2
      }

Cloud Subscription (SECTION 4)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/cloud

   **Description**: Get cloud subscription status.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "linked": true,
        "account_email": "admin@innospace.com",
        "subscription_tier": "Enterprise",
        "auto_renewal": true
      }

.. http:post:: /api/license-management/cloud/unlink

   **Description**: Unlink cloud account.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "unlinked": true,
        "message": "Cloud account unlinked successfully"
      }

.. http:get:: /api/license-management/cloud/sync-settings

   **Description**: Get cloud sync settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "auto_sync": true,
        "sync_frequency": "every_6_hours",
        "next_sync": "2024-03-20T20:30:00Z"
      }

.. http:post:: /api/license-management/cloud/sync-settings

   **Description**: Update cloud sync settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Sync settings updated successfully"
      }

Trials Management (SECTION 5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/trials

   **Description**: Get all active trials.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

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

.. http:get:: /api/license-management/trials/available

   **Description**: Get available trials.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

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

.. http:post:: /api/license-management/trials/start

   **Description**: Start a trial.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "started": true,
        "trial_id": "TRIAL-20240320-001",
        "end_date": "2024-04-03",
        "message": "Trial started successfully"
      }

.. http:post:: /api/license-management/trials/{trial_id}/cancel

   **Description**: Cancel a trial.
   
   **Path Parameters**:
   
   * **trial_id** (string): Trial identifier
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cancelled": true,
        "message": "Trial cancelled successfully"
      }

License Renewal Options
~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/renewal-options

   **Description**: Get simple renewal options for modal display.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "current_expiry": "2026-08-12",
        "options": [
          {
            "period": "1_year",
            "display_name": "1 Year",
            "price": 499,
            "currency": "USD",
            "description": "Per gateway"
          },
          {
            "period": "3_years",
            "display_name": "3 Years",
            "price": 1199,
            "currency": "USD",
            "description": "Save 20% • $399/year"
          }
        ]
      }

License Logs (SECTION 6)
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/license-management/logs

   **Description**: Get license activity logs.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

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

.. http:post:: /api/license-management/validate-all

   **Description**: Validate all licenses.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "validated": true,
        "message": "All licenses validated successfully"
      }

.. http:post:: /api/license-management/backup

   **Description**: Create license backup.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backup_id": "BACKUP-20240320-001",
        "download_url": "/api/license-management/backup/download/BACKUP-20240320-001.json",
        "message": "Backup created successfully"
      }

.. http:post:: /api/license-management/settings

   **Description**: Save license settings.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json
   
   **Success Response**::

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "saved": true,
        "message": "Settings saved successfully"
      }

Modal Actions
~~~~~~~~~~~~~

.. http:get:: /api/license-management/help

   **Description**: Get help documentation.
   
   **Headers**::

      Cookie: session_token=<token>
      X-Gateway-ID: GW-3920A9
   
   **Success Response**::

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

Route Summary
-------------

.. list-table:: Licensing Management Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/license-management`` - Main licensing page
     - Yes
   * - Overview
     - GET
     - ``/api/license-management/overview`` - License overview
     - Yes
   * - Overview
     - POST
     - ``/api/license-management/sync`` - Sync with cloud
     - Yes
   * - Overview
     - POST
     - ``/api/license-management/deactivate`` - Deactivate license
     - Yes
   * - Overview
     - POST
     - ``/api/license-management/transfer`` - Transfer license
     - Yes
   * - Features
     - GET
     - ``/api/license-management/features`` - Get features
     - Yes
   * - Features
     - POST
     - ``/api/license-management/features/{id}/toggle`` - Toggle feature
     - Yes
   * - License Key
     - POST
     - ``/api/license-management/validate`` - Validate & activate license key
     - Yes
   * - License Key
     - GET
     - ``/api/license-management/license-info`` - Current license info
     - Yes
   * - License Key
     - POST
     - ``/api/license-management/import`` - Import license file
     - Yes
   * - License Key
     - GET
     - ``/api/license-management/export`` - Export license data
     - Yes
   * - License Key
     - GET
     - ``/api/license-management/license-history`` - License activation history
     - Yes
   * - Cloud
     - GET
     - ``/api/license-management/cloud`` - Cloud status
     - Yes
   * - Cloud
     - POST
     - ``/api/license-management/cloud/unlink`` - Unlink cloud
     - Yes
   * - Cloud
     - GET
     - ``/api/license-management/cloud/sync-settings`` - Get sync settings
     - Yes
   * - Cloud
     - POST
     - ``/api/license-management/cloud/sync-settings`` - Update sync settings
     - Yes
   * - Trials
     - GET
     - ``/api/license-management/trials`` - Active trials
     - Yes
   * - Trials
     - GET
     - ``/api/license-management/trials/available`` - Available trials
     - Yes
   * - Trials
     - POST
     - ``/api/license-management/trials/start`` - Start trial
     - Yes
   * - Trials
     - POST
     - ``/api/license-management/trials/{id}/cancel`` - Cancel trial
     - Yes
   * - Renewal
     - GET
     - ``/api/license-management/renewal-options`` - Renewal options
     - Yes
   * - Logs
     - GET
     - ``/api/license-management/logs`` - License logs
     - Yes
   * - Footer
     - POST
     - ``/api/license-management/validate-all`` - Validate all licenses
     - Yes
   * - Footer
     - POST
     - ``/api/license-management/backup`` - Create backup
     - Yes
   * - Footer
     - POST
     - ``/api/license-management/settings`` - Save settings
     - Yes
   * - Modal
     - GET
     - ``/api/license-management/help`` - Get help
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /license-management``
   - Server renders HTML with embedded license data
   - All 6 sections populated with current data

2. **User views license overview** (SECTION 1):
   - License status cards show active/expired status
   - "Sync with Cloud" button → ``POST /api/license-management/sync``
   - "Renew License" button → ``GET /api/license-management/renewal-options``
   - "Transfer License" button → ``POST /api/license-management/transfer``
   - "Deactivate" button → ``POST /api/license-management/deactivate``

3. **User manages features** (SECTION 2):
   - Feature table shows all available features
   - Enable/Disable buttons → ``POST /api/license-management/features/{id}/toggle``
   - "View All Features" shows complete list

4. **User manages license keys** (SECTION 3):
   - "Validate & Activate" button → ``POST /api/license-management/validate``
   - "Import File" button → ``POST /api/license-management/import``
   - "Export" button → ``GET /api/license-management/export``
   - "History" button → ``GET /api/license-management/license-history``
   - Current license info from ``GET /api/license-management/license-info``

5. **User manages cloud subscription** (SECTION 4):
   - Cloud link status shown
   - "Sync Now" button → ``POST /api/license-management/sync``
   - "Sync Settings" modal → ``GET/POST /api/license-management/cloud/sync-settings``
   - "Unlink" button → ``POST /api/license-management/cloud/unlink``

6. **User manages trials** (SECTION 5):
   - Active trials shown with progress bars
   - Available trials list for starting new ones
   - "Start Trial" → ``POST /api/license-management/trials/start``
   - "Cancel Trial" → ``POST /api/license-management/trials/{id}/cancel``
   - "Upgrade" and "Extend" buttons for active trials

7. **User views logs** (SECTION 6):
   - License activity log shows all changes
   - Filter logs by type (All, Activations, Sync Events)
   - Export logs for audit purposes

8. **User uses footer actions**:
   - "Validate All Licenses" → ``POST /api/license-management/validate-all``
   - "Backup License Data" → ``POST /api/license-management/backup``
   - "Reset Trials" button (triggers trial reset)
   - "Save License Settings" → ``POST /api/license-management/settings``

9. **User accesses modal actions**:
   - "Help" button → ``GET /api/license-management/help``
   - "Renew License" modal → ``GET /api/license-management/renewal-options``

Error Codes
-----------

.. list-table:: Licensing Error Codes
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
   * - CLOUD_SYNC_FAILED
     - Cloud sync failed
   * - FEATURE_NOT_LICENSED
     - Feature not licensed
   * - BACKUP_FAILED
     - Backup creation failed

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Cookie: session_token=<token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages licenses for the current gateway specified in `X-Gateway-ID` header. Each gateway has its own independent license state.

License Types & Features
------------------------

### Supported License Tiers

.. list-table:: License Tiers
   :widths: 30 40 30
   :header-rows: 1

   * - Tier
     - Description
     - Max Features
   * - **Basic**
     - Core gateway functionality
     - 5
   * - **Pro**
     - Advanced features + analytics
     - 12
   * - **Enterprise**
     - Full feature set + cloud sync
     - Unlimited

### Feature Categories

1. **Safety Features**: CraneIQ Pro, Load Monitoring, Sway Detection
2. **Analytics Features**: Advanced Analytics, Historical Reports, Predictive Maintenance
3. **Connectivity Features**: Cloud Sync, Multi-Protocol Support, API Access
4. **Management Features**: Remote Configuration, Bulk Operations, Role-Based Access

Trial Management
----------------

### Trial Types
- **Feature Trials**: Individual feature trials (14-30 days)
- **Tier Trials**: Full license tier trials (30 days)
- **Demo Mode**: Limited functionality for evaluation

### Trial Limitations
- Maximum 2 concurrent trials per gateway
- Trials cannot be extended beyond original period
- Some features may be limited during trial

Cloud Integration
-----------------

### Sync Modes
- **Manual Sync**: User-initiated synchronization
- **Auto Sync**: Scheduled synchronization (every 6 hours, daily, weekly)
- **Real-time Sync**: Immediate sync on changes (Enterprise only)

### Data Synchronized
- License status and expiry
- Feature entitlements
- Trial information
- Usage statistics
- Configuration settings

Renewal Process
---------------

### Simple Renewal Flow
1. User clicks "Renew License" button in SECTION 1
2. Modal opens with renewal options (1 Year, 3 Years)
3. User selects option and clicks "Proceed to Payment"
4. System redirects to payment gateway
5. After payment, license expiry is extended

### Renewal Options
- **1 Year**: $499 per gateway
- **3 Years**: $1,199 (Save 20% - $399/year)

### Auto-renewal
- Can be enabled/disabled in renewal modal
- Recommended for uninterrupted service
- Sends email notifications before renewal

Support Information
-------------------

- **Email**: licenses@univa.com
- **Phone**: +1 (555) 123-4567
- **Support Hours**: Monday-Friday 9am-6pm EST
- **Documentation**: https://docs.univa.com/licensing

Version History
---------------

.. list-table:: API Version History
   :widths: 15 85
   :header-rows: 1

   * - Version
     - Changes
   * - 1.0.0
     - Initial release of Licensing & Subscriptions API
   * - 1.1.0
     - Added license transfer functionality
   * - 1.2.0
     - Enhanced cloud sync with detailed status
   * - 1.3.0
     - Added license activation history

Deprecation Notes
-----------------

No endpoints are currently deprecated. All endpoints are fully supported in the current version.

Rate Limiting
-------------

- **General Endpoints**: 60 requests per minute per gateway
- **Sync Operations**: 10 requests per minute per gateway
- **License Validation**: 5 requests per minute per gateway
- **Backup Operations**: 2 requests per minute per gateway

All rate limits are per gateway ID. Exceeding limits will result in HTTP 429 Too Many Requests response.

Data Formats
------------

### License Key Format
- Format: XXXX-XXXX-XXXX-XXXX (24 characters)
- Characters: A-Z, 0-9
- Validation: Checksum validation included

### Import Formats
- **JSON**: Plain text JSON file with license data
- **Encrypted**: Encrypted license file (.lic extension)
- **Text**: Plain text license key

### Export Formats
- **JSON**: Human-readable JSON format
- **Encrypted**: Secure encrypted format for backup
- **CSV**: Comma-separated values for spreadsheet import

Troubleshooting
---------------

### Common Issues

1. **License activation fails**
   - Verify internet connection for online activation
   - Check license key format
   - Ensure gateway ID matches

2. **Cloud sync fails**
   - Check network connectivity
   - Verify cloud account credentials
   - Check sync settings configuration

3. **Feature not enabling**
   - Verify license tier supports the feature
   - Check feature expiry date
   - Ensure cloud sync completed successfully

4. **Backup creation fails**
   - Verify sufficient disk space
   - Check file system permissions
   - Ensure backup directory exists

### Debug Information

Include the following when contacting support:
- Gateway ID
- License ID
- Error message received
- Timestamp of issue
- Steps to reproduce

Glossary
--------

.. glossary::

   License Key
      A unique 24-character code used to activate license features.

   Feature Entitlement
      Permission to use a specific software feature based on license.

   Trial Period
      Temporary access to features for evaluation purposes.

   Cloud Sync
      Synchronization of license data with cloud subscription service.

   Auto-renewal
      Automatic renewal of subscription before expiry.

   Activation Count
      Number of devices where license is currently activated vs. maximum allowed.

   Gateway ID
      Unique identifier for the physical gateway device.

   Subscription Tier
      Level of service (Basic, Pro, Enterprise) determining available features.

   License Expiry
      Date when current license becomes invalid.

   Grace Period
      Time after license expiry where features remain active before deactivation.


---
*Document last updated: March 20, 2024*
*API Version: 1.3.0*
*Gateway Version: 2.5.1*
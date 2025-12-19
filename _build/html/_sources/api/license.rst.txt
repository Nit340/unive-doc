Licensing & Subscriptions API
=============================

API for managing gateway licenses, subscriptions, feature entitlements, trials, and cloud integration.

Overview
--------

This API handles all licensing operations including license activation, feature management, cloud synchronization, trial management, and compliance checking.

Endpoints
---------

License Status & Overview
~~~~~~~~~~~~~~~~~~~~~~~~~

Get License Overview
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/overview

   Get current license status and overview information.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "license_type": "pro",
        "license_status": "active",
        "license_tier": "Pro License",
        "subscription_status": "active",
        "auto_renewal": true,
        "activation_date": "2024-03-15T00:00:00Z",
        "expiry_date": "2026-08-12T00:00:00Z",
        "days_remaining": 615,
        "gateway_id": "GW-3920A9",
        "hardware_bound": true,
        "activation_count": 1,
        "max_activations": 3
      }

Get License Statistics
^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/stats

   Get detailed license statistics and feature counts.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "total_features": 12,
        "active_features": 8,
        "trial_features": 2,
        "inactive_features": 2,
        "cloud_linked": true,
        "last_sync": "2024-03-20T14:30:22Z",
        "sync_status": "success",
        "next_sync": "2024-03-20T20:30:00Z"
      }

License Key Management
~~~~~~~~~~~~~~~~~~~~~~

Validate License Key
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/validate

   Validate a license key before activation.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/validate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_key": "XXXX-XXXX-XXXX-XXXX",
        "activation_mode": "online",
        "gateway_id": "GW-3920A9",
        "hardware_signature": "hw-sig-abc123"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "valid": true,
        "license_info": {
          "license_id": "LIC-PRO-2024-3920A9",
          "license_type": "pro",
          "expiry_date": "2026-08-12T00:00:00Z",
          "issued_to": "Innospace Solutions",
          "issued_on": "2024-03-15T00:00:00Z",
          "features": ["craneiq_pro", "anti_collision", "rule_engine", "analytics"]
        },
        "requires_activation": true
      }

Activate License
^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/activate

   Activate a validated license key.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/activate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_key": "XXXX-XXXX-XXXX-XXXX",
        "activation_type": "online",
        "offline_activation_code": null,
        "gateway_info": {
          "id": "GW-3920A9",
          "model": "Univa-GW-Pro",
          "serial": "SN-2024-00123",
          "hardware_signature": "hw-sig-abc123"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "activation_id": "ACT-20240320-001",
        "activation_date": "2024-03-20T10:30:00Z",
        "license_id": "LIC-PRO-2024-3920A9",
        "activation_token": "act_token_abc123",
        "entitlements": {
          "features": ["craneiq_pro", "anti_collision"],
          "expiry": "2026-08-12T00:00:00Z"
        }
      }

Generate Offline Activation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/offline/generate

   Generate an offline activation request code.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/offline/generate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_key": "XXXX-XXXX-XXXX-XXXX",
        "gateway_info": {
          "id": "GW-3920A9",
          "hardware_signature": "hw-sig-abc123"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "activation_request_code": "REQ-ABC123XYZ456",
        "expires_in": 86400,
        "instructions": "Submit this code at license.innospace.com/offline"
      }

Complete Offline Activation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/offline/complete

   Complete offline activation with response code.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/offline/complete HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "activation_request_code": "REQ-ABC123XYZ456",
        "activation_response_code": "RES-DEF789UVW012"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "activated": true,
        "license_id": "LIC-PRO-2024-3920A9",
        "entitlements": {
          "features": ["craneiq_pro", "anti_collision"],
          "expiry": "2026-08-12T00:00:00Z"
        }
      }

Export License Data
^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/export

   Export license data in various formats.

   **Query Parameters**:

   * **format** (string): "json", "encrypted" (default: json)
   * **include_keys** (boolean): Include license keys (default: true)
   * **include_entitlements** (boolean): Include feature entitlements (default: true)
   * **include_logs** (boolean): Include activation logs (default: true)

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "export_id": "EXP-20240320-001",
        "download_url": "/cloud-integration/license/export/download/EXP-20240320-001.lic",
        "expires": "2024-03-21T10:30:00Z",
        "format": "encrypted",
        "size": "2.5 KB"
      }

Import License Data
^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/import

   Import license data from file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/import HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="license.lic"
      Content-Type: application/octet-stream

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "imported": true,
        "licenses_imported": 1,
        "features_updated": 8,
        "activation_count": 1
      }

Deactivate License
^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/deactivate

   Deactivate a license from current gateway.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/deactivate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "reason": "decommissioning",
        "permanent": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "deactivated": true,
        "deactivation_id": "DEACT-20240320-001",
        "license_slot_freed": true,
        "can_reactivate": true
      }

Feature Management
~~~~~~~~~~~~~~~~~~

Get Feature Matrix
^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/features

   Get all available features with their status and limits.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "features": [
          {
            "id": "craneiq_pro",
            "name": "CraneIQ Pro",
            "category": "safety",
            "status": "active",
            "license_source": "cloud",
            "license_tier": "pro",
            "expiry": "2026-08-12T00:00:00Z",
            "limits": {
              "max_cranes": 10,
              "max_sensors": 100,
              "data_retention": "unlimited"
            },
            "trial": false,
            "trial_days_left": 0
          }
        ],
        "total_features": 12,
        "active_count": 8,
        "trial_count": 2
      }

Enable/Disable Feature
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/features/toggle

   Toggle feature activation status.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/features/toggle HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "feature_id": "craneiq_pro",
        "enabled": false,
        "reason": "maintenance"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "toggled": true,
        "feature_id": "craneiq_pro",
        "new_status": "inactive",
        "effective_immediately": true
      }

Get Feature Limits
^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/features/{feature_id}/limits

   Get current usage and limits for a specific feature.

   **Path Parameters**:

   * **feature_id** (string): Feature identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "feature_id": "craneiq_pro",
        "current_usage": {
          "cranes": 8,
          "sensors": 75,
          "alerts": 245
        },
        "limits": {
          "max_cranes": 10,
          "max_sensors": 100,
          "max_alerts": 1000,
          "data_retention_days": 365
        },
        "utilization": {
          "cranes": 80,
          "sensors": 75,
          "alerts": 24.5
        }
      }

Check Feature Availability
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/features/{feature_id}/availability

   Check if a feature is available for the current license.

   **Path Parameters**:

   * **feature_id** (string): Feature identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "feature_id": "advanced_analytics",
        "available": true,
        "license_required": "pro",
        "current_license": "pro",
        "trial_available": true,
        "trial_days": 14,
        "pricing": {
          "monthly": 99,
          "yearly": 999,
          "currency": "USD"
        }
      }

Cloud Subscription Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get Cloud Status
^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/cloud/status

   Get cloud account linking status and subscription information.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "linked": true,
        "account_email": "admin@innospace.com",
        "subscription_tier": "enterprise",
        "subscription_status": "active",
        "auto_renewal": true,
        "next_billing_date": "2024-04-15T00:00:00Z",
        "payment_method": "credit_card",
        "sync_status": {
          "last_sync": "2024-03-20T14:30:22Z",
          "next_sync": "2024-03-20T20:30:00Z",
          "sync_enabled": true,
          "sync_frequency": "every_6_hours"
        }
      }

Sync with Cloud
^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/cloud/sync

   Synchronize licenses and entitlements with cloud.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/cloud/sync HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "force": false,
        "sync_entitlements": true,
        "sync_usage": true,
        "sync_licenses": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "sync_id": "SYNC-20240320-001",
        "started": "2024-03-20T14:30:22Z",
        "completed": "2024-03-20T14:30:45Z",
        "duration": 23,
        "results": {
          "entitlements_updated": 3,
          "usage_synced": true,
          "licenses_verified": 2,
          "new_features": 1
        },
        "status": "success"
      }

Unlink Cloud Account
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/cloud/unlink

   Unlink cloud account while preserving local licenses.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/cloud/unlink HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "confirm": true,
        "reason": "migrating_to_self_hosted"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "unlinked": true,
        "local_licenses_preserved": true,
        "can_relink": true,
        "message": "Cloud account unlinked. Local licenses remain active."
      }

Get Sync Settings
^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/cloud/sync-settings

   Get current cloud synchronization settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "auto_sync": true,
        "sync_frequency": "every_6_hours",
        "sync_on_startup": true,
        "data_to_sync": {
          "entitlements": true,
          "usage_metrics": true,
          "license_updates": true,
          "gateway_health": false,
          "diagnostic_data": false
        },
        "compression_enabled": true,
        "encryption_enabled": true,
        "sync_timeout": 300,
        "retry_attempts": 3
      }

Update Sync Settings
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/cloud/sync-settings

   Update cloud synchronization settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/cloud/sync-settings HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "auto_sync": true,
        "sync_frequency": "daily",
        "data_to_sync": {
          "entitlements": true,
          "usage_metrics": true,
          "license_updates": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "next_sync": "2024-03-21T02:00:00Z"
      }

Trial Management
~~~~~~~~~~~~~~~~

Get Active Trials
^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/trials

   Get all active trials with their status.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "active_trials": [
          {
            "trial_id": "TRIAL-20240315-001",
            "feature_id": "craneiq_pro",
            "feature_name": "CraneIQ Pro",
            "start_date": "2024-03-15T00:00:00Z",
            "end_date": "2024-04-15T00:00:00Z",
            "days_left": 10,
            "progress": 70,
            "can_extend": true,
            "can_upgrade": true
          }
        ],
        "total_trials": 2
      }

Start Trial
^^^^^^^^^^^

.. http:post:: /cloud-integration/license/trials/start

   Start a new trial for a feature.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/trials/start HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "feature_id": "rule_engine_pro",
        "trial_days": 14,
        "notify_on_expiry": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "trial_id": "TRIAL-20240320-002",
        "feature_id": "rule_engine_pro",
        "start_date": "2024-03-20T14:30:00Z",
        "end_date": "2024-04-03T14:30:00Z",
        "days": 14,
        "limits": {
          "max_rules": 50,
          "max_executions": 1000
        },
        "restrictions": ["no_export", "watermarked_reports"]
      }

Extend Trial
^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/trials/extend

   Extend an existing trial period.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/trials/extend HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "trial_id": "TRIAL-20240315-001",
        "extension_days": 7,
        "reason": "evaluation_not_complete"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "extended": true,
        "trial_id": "TRIAL-20240315-001",
        "new_end_date": "2024-04-22T00:00:00Z",
        "days_added": 7,
        "total_days": 37
      }

Cancel Trial
^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/trials/cancel

   Cancel an active trial.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/trials/cancel HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "trial_id": "TRIAL-20240320-001",
        "reason": "no_longer_needed"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cancelled": true,
        "trial_id": "TRIAL-20240320-001",
        "feature_disabled": true,
        "data_preserved": true
      }

Upgrade Trial to Full License
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/trials/upgrade

   Upgrade a trial to a full license.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/trials/upgrade HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "trial_id": "TRIAL-20240315-001",
        "license_tier": "pro",
        "billing_period": "yearly",
        "auto_renew": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "upgraded": true,
        "trial_id": "TRIAL-20240315-001",
        "license_id": "LIC-PRO-20240320-001",
        "activation_date": "2024-03-20T14:30:00Z",
        "expiry_date": "2025-03-20T14:30:00Z",
        "payment_required": true,
        "payment_url": "https://checkout.innospace.com/license-abc123"
      }

Get Available Trials
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/trials/available

   Get list of available trials for current license tier.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "available_trials": [
          {
            "feature_id": "rule_engine_pro",
            "feature_name": "Rule Engine Pro",
            "trial_days": 14,
            "description": "Advanced automation and rule-based control system",
            "requirements": "pro_license",
            "limits": {
              "max_rules": 50,
              "max_executions": 1000
            }
          }
        ]
      }

License Transfer & Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Transfer License
^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/transfer

   Transfer license to another gateway.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/transfer HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "transfer_type": "copy", // "move" or "copy"
        "target_gateway_id": "GW-2024-00234",
        "target_hardware_signature": "hw-sig-def456",
        "reason": "gateway_replacement"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "transfer_id": "TRANSFER-20240320-001",
        "status": "pending_approval",
        "source_gateway": "GW-3920A9",
        "target_gateway": "GW-2024-00234",
        "transfer_type": "copy",
        "requires_support": true,
        "support_ticket": "SUP-20240320-001"
      }

Get Transfer Status
^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/transfer/{transfer_id}

   Get status of a license transfer.

   **Path Parameters**:

   * **transfer_id** (string): Transfer identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "transfer_id": "TRANSFER-20240320-001",
        "status": "completed",
        "source_gateway": "GW-3920A9",
        "target_gateway": "GW-2024-00234",
        "transfer_type": "copy",
        "transferred_date": "2024-03-20T15:30:00Z",
        "license_count": 1,
        "features_transferred": 8
      }

Cancel Transfer
^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/transfer/{transfer_id}/cancel

   Cancel a pending license transfer.

   **Path Parameters**:

   * **transfer_id** (string): Transfer identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "cancelled": true,
        "transfer_id": "TRANSFER-20240320-001",
        "reason": "cancelled_by_user",
        "original_license_restored": true
      }

Get License History
^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/history

   Get license activation and usage history.

   **Query Parameters**:

   * **days** (integer): Number of days to look back (default: 30)
   * **type** (string): Filter by event type

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "history": [
          {
            "timestamp": "2024-03-20T14:30:22Z",
            "event_type": "sync",
            "event": "Cloud sync completed",
            "user": "system",
            "details": {
              "entitlements_updated": 3,
              "duration": "23s"
            }
          }
        ],
        "total_events": 42
      }

Renewal & Billing
~~~~~~~~~~~~~~~~~

Get Renewal Options
^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/renewal/options

   Get license renewal options and pricing.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "current_license": {
          "license_id": "LIC-PRO-2024-3920A9",
          "license_tier": "pro",
          "expiry_date": "2026-08-12T00:00:00Z",
          "auto_renewal": true
        },
        "renewal_options": [
          {
            "period": "1_year",
            "price": 499,
            "currency": "USD",
            "per_gateway": true,
            "savings": "0%",
            "features_included": "all_current"
          }
        ]
      }

Initiate Renewal
^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/renewal/initiate

   Initiate license renewal process.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/renewal/initiate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "renewal_period": "3_years",
        "auto_renewal": true,
        "payment_method": "existing_card"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "renewal_id": "RENEW-20240320-001",
        "status": "payment_required",
        "amount": 1199,
        "currency": "USD",
        "new_expiry_date": "2029-08-12T00:00:00Z",
        "payment_url": "https://checkout.innospace.com/renewal-abc123"
      }

Update Auto-Renewal
^^^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/renewal/auto-renew

   Update auto-renewal settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/renewal/auto-renew HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "license_id": "LIC-PRO-2024-3920A9",
        "enabled": true,
        "payment_method": "credit_card"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "license_id": "LIC-PRO-2024-3920A9",
        "auto_renewal": true,
        "next_renewal_date": "2026-08-01T00:00:00Z"
      }

Get Billing Information
^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/billing

   Get billing information and invoice history.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "current_plan": "pro",
        "billing_period": "yearly",
        "next_billing_date": "2025-03-15T00:00:00Z",
        "amount_due": 499,
        "currency": "USD",
        "payment_method": {
          "type": "credit_card",
          "last_four": "4242",
          "expiry": "12/2025"
        },
        "invoices": [
          {
            "invoice_id": "INV-20240315-001",
            "date": "2024-03-15T00:00:00Z",
            "amount": 499,
            "currency": "USD",
            "status": "paid",
            "download_url": "/cloud-integration/license/invoices/INV-20240315-001.pdf"
          }
        ]
      }

License Backup & Restore
~~~~~~~~~~~~~~~~~~~~~~~~

Create Backup
^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/backup/create

   Create backup of license data.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/backup/create HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "include_keys": true,
        "include_entitlements": true,
        "include_logs": true,
        "encrypt": true,
        "password": "optional_password"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backup_id": "BACKUP-20240320-001",
        "created": "2024-03-20T14:30:00Z",
        "download_url": "/cloud-integration/license/backup/download/BACKUP-20240320-001.lic",
        "expires": "2024-03-27T14:30:00Z",
        "size": "3.2 KB",
        "encrypted": true
      }

Restore Backup
^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/backup/restore

   Restore license data from backup file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/backup/restore HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="backup.lic"
      Content-Type: application/octet-stream

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "restored": true,
        "backup_id": "BACKUP-20240320-001",
        "licenses_restored": 1,
        "features_restored": 8,
        "activations_restored": 1,
        "requires_reboot": false
      }

Get Backup History
^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/backup/history

   Get list of available backups.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backups": [
          {
            "backup_id": "BACKUP-20240320-001",
            "created": "2024-03-20T14:30:00Z",
            "size": "3.2 KB",
            "encrypted": true,
            "contains": ["licenses", "entitlements", "logs"]
          }
        ],
        "total_backups": 5
      }

Validation & Compliance
~~~~~~~~~~~~~~~~~~~~~~~

Run License Check
^^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/validate/all

   Run comprehensive license validation and compliance check.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/validate/all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "deep_check": true,
        "check_expiry": true,
        "check_entitlements": true,
        "check_compliance": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "check_id": "CHECK-20240320-001",
        "started": "2024-03-20T14:30:00Z",
        "completed": "2024-03-20T14:31:15Z",
        "results": {
          "licenses_valid": 2,
          "licenses_expired": 0,
          "licenses_expiring_soon": 0,
          "features_licensed": 8,
          "features_unlicensed": 4,
          "compliance_issues": 0
        },
        "issues": [],
        "warnings": [
          "Feature 'advanced_analytics' expires in 5 days"
        ],
        "compliance_status": "compliant"
      }

Get Compliance Report
^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cloud-integration/license/compliance

   Get current compliance status report.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "compliance_status": "compliant",
        "last_check": "2024-03-20T14:31:15Z",
        "next_check": "2024-03-21T14:31:15Z",
        "violations": 0,
        "warnings": 1,
        "features": {
          "licensed": 8,
          "unlicensed": 4,
          "trial": 2
        },
        "licenses": {
          "valid": 2,
          "expired": 0,
          "expiring_soon": 0
        }
      }

Reset All Trials
^^^^^^^^^^^^^^^^

.. http:post:: /cloud-integration/license/trials/reset-all

   Reset all active trials.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/license/trials/reset-all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "confirm": true,
        "reason": "system_reset"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "trials_reset": 2,
        "features_affected": ["advanced_analytics", "data_historian"],
        "restart_date": "2024-03-20T14:30:00Z",
        "new_expiry": "2024-04-03T14:30:00Z"
      }

WebSocket Endpoints
~~~~~~~~~~~~~~~~~~~

License Events Stream
^^^^^^^^^^^^^^^^^^^^^

.. http:ws:: /cloud-integration/license/ws/events

   Real-time stream for license events and updates.

   **Messages Received**:

   .. sourcecode:: json

      {
        "type": "license_updated",
        "timestamp": "2024-03-20T14:30:22Z",
        "data": {
          "license_id": "LIC-PRO-2024-3920A9",
          "action": "sync_completed",
          "features_updated": 3
        }
      }

Error Codes
-----------

.. list-table:: Licensing API Errors
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - LICENSE_INVALID
     - Invalid license key format
   * - LICENSE_EXPIRED
     - License has expired
   * - LICENSE_MAX_ACTIVATIONS
     - Maximum activations reached
   * - TRIAL_NOT_AVAILABLE
     - Trial not available for feature
   * - CLOUD_SYNC_FAILED
     - Cloud synchronization failed
   * - FEATURE_NOT_LICENSED
     - Feature not included in license
   * - TRANSFER_NOT_ALLOWED
     - License transfer not permitted
   * - PAYMENT_REQUIRED
     - Payment required for operation
   * - BACKUP_CORRUPTED
     - Backup file is corrupted
   * - HARDWARE_MISMATCH
     - Hardware signature mismatch
   * - ACTIVATION_FAILED
     - License activation failed
   * - SUBSCRIPTION_INACTIVE
     - Cloud subscription is inactive
   * - TRIAL_EXPIRED
     - Trial period has expired

Examples
--------

Python - Activate License
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Validate and activate license
   headers = {"Authorization": "Bearer your_token"}
   
   # 1. Validate license key
   validate_data = {
       "license_key": "ABCD-EFGH-IJKL-MNOP",
       "activation_mode": "online",
       "gateway_id": "GW-3920A9"
   }
   
   validate_response = requests.post(
       "http://localhost/cloud-integration/license/validate",
       json=validate_data,
       headers=headers
   )
   
   if validate_response.json()["valid"]:
       # 2. Activate license
       activate_data = {
           "license_key": "ABCD-EFGH-IJKL-MNOP",
           "gateway_info": {
               "id": "GW-3920A9",
               "model": "Univa-GW-Pro"
           }
       }
       
       activate_response = requests.post(
           "http://localhost/cloud-integration/license/activate",
           json=activate_data,
           headers=headers
       )
       
       activation_result = activate_response.json()
       print(f"License activated: {activation_result['license_id']}")

Python - Manage Trials
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Start and manage trials
   
   # 1. Start a trial
   trial_data = {
       "feature_id": "rule_engine_pro",
       "trial_days": 14
   }
   
   trial_response = requests.post(
       "http://localhost/cloud-integration/license/trials/start",
       json=trial_data,
       headers=headers
   )
   
   trial_info = trial_response.json()
   print(f"Trial started: {trial_info['trial_id']}")
   
   # 2. Get active trials
   active_trials_response = requests.get(
       "http://localhost/cloud-integration/license/trials",
       headers=headers
   )
   
   trials = active_trials_response.json()["active_trials"]
   for trial in trials:
       print(f"Trial: {trial['feature_name']}, Days left: {trial['days_left']}")
   
   # 3. Upgrade trial to full license
   if trials:
       upgrade_data = {
           "trial_id": trials[0]["trial_id"],
           "license_tier": "pro"
       }
       
       upgrade_response = requests.post(
           "http://localhost/cloud-integration/license/trials/upgrade",
           json=upgrade_data,
           headers=headers
       )
       
       upgrade_result = upgrade_response.json()
       if upgrade_result["upgraded"]:
           print(f"Trial upgraded to license: {upgrade_result['license_id']}")

Python - Cloud Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Cloud sync and management
   
   # 1. Get cloud status
   cloud_status_response = requests.get(
       "http://localhost/cloud-integration/license/cloud/status",
       headers=headers
   )
   
   cloud_status = cloud_status_response.json()
   print(f"Cloud linked: {cloud_status['linked']}")
   print(f"Next sync: {cloud_status['sync_status']['next_sync']}")
   
   # 2. Sync with cloud
   sync_data = {
       "force": False,
       "sync_entitlements": True,
       "sync_usage": True
   }
   
   sync_response = requests.post(
       "http://localhost/cloud-integration/license/cloud/sync",
       json=sync_data,
       headers=headers
   )
   
   sync_result = sync_response.json()
   print(f"Sync completed: {sync_result['duration']}s")
   
   # 3. Get feature matrix
   features_response = requests.get(
       "http://localhost/cloud-integration/license/features",
       headers=headers
   )
   
   features = features_response.json()["features"]
   for feature in features:
       status_icon = "✓" if feature["status"] == "active" else "✗"
       print(f"{status_icon} {feature['name']}: {feature['status']}")

JavaScript - License Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // JavaScript examples for browser integration
   
   // Activate license
   async function activateLicense(licenseKey, gatewayId) {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-integration/license/activate', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               license_key: licenseKey,
               gateway_info: {
                   id: gatewayId,
                   model: 'Univa-GW-Pro'
               }
           })
       });
       
       const result = await response.json();
       
       if (result.activation_id) {
           alert(`License activated successfully! ID: ${result.license_id}`);
           updateLicenseUI(result);
       } else {
           alert('License activation failed');
       }
       
       return result;
   }
   
   // Get license overview
   async function getLicenseOverview() {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-integration/license/overview', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       
       // Update UI with license info
       document.getElementById('license-type').textContent = data.license_tier;
       document.getElementById('license-status').textContent = data.license_status;
       document.getElementById('days-remaining').textContent = data.days_remaining;
       
       return data;
   }
   
   // Run license compliance check
   async function runLicenseCheck() {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-integration/license/validate/all', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               deep_check: true,
               check_expiry: true,
               check_compliance: true
           })
       });
       
       const result = await response.json();
       
       // Display results
       const issues = result.issues || [];
       const warnings = result.warnings || [];
       
       if (issues.length > 0) {
           console.error('License issues found:', issues);
       }
       
       if (warnings.length > 0) {
           console.warn('License warnings:', warnings);
       }
       
       return result;
   }

JavaScript - Feature Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Feature management functions
   
   // Get all features
   async function getFeatures() {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-integration/license/features', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       
       // Render features table
       const featuresTable = document.getElementById('features-table');
       featuresTable.innerHTML = '';
       
       data.features.forEach(feature => {
           const row = featuresTable.insertRow();
           row.innerHTML = `
               <td>${feature.name}</td>
               <td><span class="status-${feature.status}">${feature.status}</span></td>
               <td>${feature.expiry ? new Date(feature.expiry).toLocaleDateString() : 'N/A'}</td>
               <td>${feature.trial ? `${feature.trial_days_left} days left` : 'Full license'}</td>
               <td>
                   <button onclick="toggleFeature('${feature.id}', ${!feature.enabled})">
                       ${feature.enabled ? 'Disable' : 'Enable'}
                   </button>
               </td>
           `;
       });
       
       return data;
   }
   
   // Toggle feature status
   async function toggleFeature(featureId, enabled) {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-integration/license/features/toggle', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               feature_id: featureId,
               enabled: enabled
           })
       });
       
       const result = await response.json();
       
       if (result.toggled) {
           alert(`Feature ${enabled ? 'enabled' : 'disabled'} successfully`);
           getFeatures(); // Refresh features list
       }
       
       return result;
   }
   
   // Check feature availability
   async function checkFeatureAvailability(featureId) {
       const token = localStorage.getItem('token');
       
       const response = await fetch(`/cloud-integration/license/features/${featureId}/availability`, {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       
       if (data.available) {
           if (data.trial_available) {
               alert(`Feature available! ${data.trial_days}-day trial offered.`);
           } else {
               alert('Feature available for your license tier.');
           }
       } else {
           alert('Feature not available for your current license.');
       }
       
       return data;
   }

JavaScript - WebSocket Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // WebSocket for real-time license updates
   
   let licenseSocket = null;
   
   function connectLicenseWebSocket() {
       const token = localStorage.getItem('token');
       const wsUrl = `ws://${window.location.host}/cloud-integration/license/ws/events?token=${token}`;
       
       licenseSocket = new WebSocket(wsUrl);
       
       licenseSocket.onopen = function() {
           console.log('License WebSocket connected');
       };
       
       licenseSocket.onmessage = function(event) {
           const message = JSON.parse(event.data);
           
           switch(message.type) {
               case 'license_updated':
                   console.log('License updated:', message.data);
                   updateLicenseBadge(message.data);
                   break;
                   
               case 'trial_warning':
                   console.log('Trial warning:', message.data);
                   showTrialWarning(message.data);
                   break;
                   
               case 'expiry_warning':
                   console.log('Expiry warning:', message.data);
                   showExpiryWarning(message.data);
                   break;
           }
       };
       
       licenseSocket.onclose = function() {
           console.log('License WebSocket disconnected');
           // Attempt to reconnect after 5 seconds
           setTimeout(connectLicenseWebSocket, 5000);
       };
   }
   
   function updateLicenseBadge(data) {
       const badge = document.getElementById('license-badge');
       if (badge) {
           badge.textContent = `License: ${data.action || 'updated'}`;
           badge.classList.add('highlight');
           setTimeout(() => badge.classList.remove('highlight'), 2000);
       }
   }
   
   function showTrialWarning(data) {
       const warningDiv = document.createElement('div');
       warningDiv.className = 'trial-warning';
       warningDiv.innerHTML = `
           <h4>Trial Ending Soon</h4>
           <p>${data.feature_name} trial ends in ${data.days_left} days</p>
           <button onclick="upgradeTrial('${data.trial_id}')">Upgrade Now</button>
       `;
       
       document.body.appendChild(warningDiv);
   }
   
   // Connect WebSocket when page loads
   document.addEventListener('DOMContentLoaded', connectLicenseWebSocket);

Notes
-----

- **License Tiers**: Standard, Pro, Enterprise, Custom
- **Activation Types**: Online, Offline, Network
- **Feature Categories**: Safety, Analytics, Control, Reporting, Integration
- **Sync Frequencies**: Manual, Hourly, Daily, Weekly, On Startup
- **Trial Durations**: 7 days, 14 days, 30 days (configurable)
- **Export Formats**: JSON, Encrypted (.lic), CSV
- **WebSocket Events**: Real-time updates for license changes, trial warnings, expiry notifications

Troubleshooting
---------------

1. **License Activation Failed**:
   - Verify internet connection for online activation
   - Check license key format (XXXX-XXXX-XXXX-XXXX)
   - Ensure gateway hardware signature is correct

2. **Cloud Sync Issues**:
   - Check cloud account linking status
   - Verify API connectivity to license server
   - Review sync logs for specific errors

3. **Feature Not Available**:
   - Check current license tier supports the feature
   - Verify feature is not already in trial period
   - Ensure subscription is active and paid

4. **Trial Management**:
   - Trials are per-feature, not per-license
   - Trial extensions may require approval
   - Upgrading trial preserves trial data

5. **Backup/Restore**:
   - Backups expire after 7 days for security
   - Encrypted backups require password for restore
   - Restore may require gateway reboot for full effect

Security Considerations
-----------------------

- All license keys are masked in logs and UI
- Hardware binding prevents license cloning
- Activation limits prevent license abuse
- Encrypted backups protect sensitive data
- WebSocket connections require authentication tokens
- Rate limiting applied to activation endpoints
Security & Access Control API
==============================

API for managing system security, user access control, authentication policies, and network security.

Base Path: /api/v1/security

.. contents:: Table of Contents
   :depth: 3
   :local:

Overview
--------

This API handles all security and access control operations for the current gateway. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Security Status
~~~~~~~~~~~~~~~

Get Security Status
^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/status

   Get overall security status and component health.
   
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
        "security_level": "secure",
        "security_score": 92,
        "last_scan": "2025-03-15T10:30:00Z",
        "issues_found": 0,
        "components": {
          "authentication": {"status": "good", "issues": 0},
          "firewall": {"status": "good", "rules_active": 15},
          "ssh": {"status": "good", "active_sessions": 1},
          "certificates": {"status": "warning", "expiring_soon": 1}
        }
      }

Identity & Access Management (Section A)
-----------------------------------------

Get Users
^^^^^^^^^

.. http:get:: /api/v1/security/users

   Retrieve all system users for user table.
   
   **UI Element**: SECTION A - User management table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "users": [
          {
            "id": 1,
            "username": "admin",
            "full_name": "System Admin",
            "email": "admin@innospace.com",
            "role": "admin",
            "status": "online",
            "last_login": "2025-03-15T10:15:00Z",
            "login_count": 127,
            "two_factor_enabled": true,
            "created_at": "2024-01-15T09:00:00Z"
          }
        ],
        "total": 8,
        "active": 7
      }

Create User
^^^^^^^^^^^

.. http:post:: /api/v1/security/users

   Create new user from "Add User" modal.
   
   **UI Element**: SECTION A - "Add User" button modal
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "username": "tech02",
        "full_name": "Technician 02",
        "email": "tech02@example.com",
        "role": "technician",
        "password": "SecurePass123!",
        "two_factor_enabled": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "created": true,
        "user_id": 9,
        "message": "User created successfully"
      }

Update User
^^^^^^^^^^^

.. http:put:: /api/v1/security/users/{user_id}

   Update user from "Edit User" modal.
   
   **UI Element**: SECTION A - "Edit" button in user table
   
   **Path Parameters**:

   * **user_id** (integer): User identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "full_name": "Updated Name",
        "email": "updated@example.com",
        "role": "engineer",
        "two_factor_enabled": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "User updated successfully"
      }

Delete User
^^^^^^^^^^^

.. http:delete:: /api/v1/security/users/{user_id}

   Delete user from "Edit User" modal.
   
   **UI Element**: SECTION A - "Delete" button in user table
   
   **Path Parameters**:

   * **user_id** (integer): User identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "deleted": true,
        "message": "User deleted successfully"
      }

Get Roles
^^^^^^^^^

.. http:get:: /api/v1/security/roles

   Get all roles for sidebar display.
   
   **UI Element**: SECTION A - Role list sidebar
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "roles": [
          {
            "name": "Admin",
            "description": "Full system access",
            "permissions": ["*"],
            "user_count": 1,
            "default": false,
            "protected": true
          }
        ]
      }

Authentication Policies (Section B)
-----------------------------------

Get Authentication Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/authentication/settings

   Get all authentication policy settings.
   
   **UI Element**: SECTION B - Authentication policies form
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "password_policy": {
          "min_length": 8,
          "expiry_days": 90,
          "require_special_chars": true,
          "require_numbers": true,
          "require_uppercase": true,
          "prevent_reuse": false
        },
        "account_protection": {
          "lockout_attempts": 5,
          "lockout_duration": 15,
          "captcha_enabled": false
        },
        "session_settings": {
          "timeout_minutes": 15,
          "max_concurrent_sessions": 3
        },
        "two_factor": {
          "mode": "optional"
        }
      }

Update Authentication Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/authentication/settings

   Save authentication policy changes.
   
   **UI Element**: SECTION B - "Save Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "password_policy": {
          "min_length": 12,
          "expiry_days": 60,
          "require_special_chars": true
        },
        "account_protection": {
          "lockout_attempts": 3,
          "lockout_duration": 30
        },
        "two_factor": {
          "mode": "required"
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Authentication settings updated successfully"
      }

Test Password Strength
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/authentication/test-password

   Test password strength.
   
   **UI Element**: SECTION A - Password strength indicator
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "password": "MySecurePass123!"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "strength": "strong",
        "score": 92,
        "meets_policy": true,
        "issues": []
      }

API Keys & Tokens (Section C)
-----------------------------

Get API Keys
^^^^^^^^^^^^

.. http:get:: /api/v1/security/api-keys

   Get all API keys for table display.
   
   **UI Element**: SECTION C - API key table (left panel)
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "api_keys": [
          {
            "id": 1,
            "name": "CI Pipeline",
            "key_prefix": "AK-001",
            "scope": ["telemetry:write"],
            "user": "admin",
            "created": "2025-01-15T10:30:00Z",
            "last_used": "2025-03-15T09:45:00Z",
            "expires": "2026-06-01T00:00:00Z",
            "status": "active"
          }
        ],
        "total": 8,
        "active": 5
      }

Generate API Key
^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/api-keys

   Generate new API key from modal.
   
   **UI Element**: SECTION C - "Generate New API Key" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "name": "Webhook Integration",
        "scope": ["alerts:read", "alerts:write"],
        "expiry_days": 90
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "generated": true,
        "api_key": {
          "id": 9,
          "name": "Webhook Integration",
          "full_key": "sk_9b3e7c2a1d8f5e6c4b9a2d7e1c3f8a5b",
          "key_prefix": "AK-009"
        },
        "warning": "Store this key securely"
      }

Revoke API Key
^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/api-keys/{key_id}/revoke

   Revoke API key from table actions.
   
   **UI Element**: SECTION C - "Revoke" button in API key table
   
   **Path Parameters**:

   * **key_id** (integer): API key identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "reason": "Security rotation"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "revoked": true,
        "message": "API key revoked successfully"
      }

Get Token Settings
^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/tokens/settings

   Get token settings for form.
   
   **UI Element**: SECTION C - Token settings form (right panel)
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "token_expiry_days": 30,
        "permissions": {
          "read_data": true,
          "write_config": true,
          "ota_access": false,
          "terminal_access": false,
          "alert_management": true,
          "user_management": false
        },
        "webhook_secret": "whsec_abc123def456"
      }

Update Token Settings
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/tokens/settings

   Update token settings.
   
   **UI Element**: SECTION C - "Update Token Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "token_expiry_days": 60,
        "permissions": {
          "alert_management": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Token settings updated successfully"
      }

Regenerate Webhook Secret
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/tokens/regenerate-secret

   Regenerate webhook signing secret.
   
   **UI Element**: SECTION C - "Regenerate Secret" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "invalidate_old": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "regenerated": true,
        "new_secret": "whsec_xyz789uvw012",
        "message": "Webhook secret regenerated"
      }

SSH Access Control (Section D)
------------------------------

Get SSH Settings
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/ssh/settings

   Get SSH configuration for form.
   
   **UI Element**: SECTION D - SSH configuration form
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "port": 22,
        "login_method": "key",
        "restricted_shell": true,
        "session_timeout": 30,
        "max_sessions_per_user": 3,
        "allowed_commands": ["ls", "cat", "ifconfig", "ping"]
      }

Update SSH Settings
^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/ssh/settings

   Save SSH configuration changes.
   
   **UI Element**: SECTION D - "Save SSH Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enabled": true,
        "port": 2222,
        "login_method": "both",
        "restricted_shell": true,
        "session_timeout": 45,
        "max_sessions_per_user": 5
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "SSH settings updated successfully"
      }

Upload SSH Public Key
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/ssh/keys

   Upload SSH public key for authentication.
   
   **UI Element**: SECTION D - "Upload SSH Key" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /api/v1/security/ssh/keys HTTP/1.1
      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="id_rsa.pub"

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "uploaded": true,
        "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
        "message": "SSH public key uploaded"
      }

Get SSH Sessions
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/ssh/sessions

   Get active SSH sessions for monitoring.
   
   **UI Element**: SECTION D - Active SSH sessions monitoring
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "active_sessions": [
          {
            "session_id": "ssh-001",
            "username": "admin",
            "client_ip": "192.168.1.100",
            "started": "2025-03-15T09:30:00Z",
            "duration": "00:45:30"
          }
        ],
        "total_sessions_today": 5
      }

Firewall & Network Security (Section E)
---------------------------------------

Get Firewall Status
^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/firewall

   Get firewall status and rules.
   
   **UI Element**: SECTION E - Firewall rules table
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "rules": [
          {
            "id": 1,
            "name": "SSH Management Access",
            "port": "22",
            "protocol": "TCP",
            "source": "192.168.1.0/24",
            "action": "allow"
          }
        ],
        "ip_whitelist": ["192.168.0.0/24", "10.0.0.5"],
        "blocked_ports": [23, 21, 8000],
        "security_features": {
          "rate_limiting": {"enabled": true, "requests_per_second": 20},
          "dos_protection": true,
          "vpn_enabled": false
        }
      }

Add Firewall Rule
^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/firewall/rules

   Add new firewall rule from modal.
   
   **UI Element**: SECTION E - "Add Firewall Rule" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "name": "Allow NTP Time Sync",
        "port": "123",
        "protocol": "UDP",
        "source": "0.0.0.0/0",
        "action": "allow"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "added": true,
        "rule_id": 16,
        "message": "Firewall rule added successfully"
      }

Remove Firewall Rule
^^^^^^^^^^^^^^^^^^^^

.. http:delete:: /api/v1/security/firewall/rules/{rule_id}

   Remove firewall rule from table.
   
   **UI Element**: SECTION E - "Delete" button in firewall rules table
   
   **Path Parameters**:

   * **rule_id** (integer): Firewall rule identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "removed": true,
        "message": "Firewall rule removed successfully"
      }

Manage IP Whitelist
^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/firewall/whitelist

   Add or remove IP addresses from whitelist.
   
   **UI Element**: SECTION E - IP whitelist management sidebar
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "action": "add",
        "ip_address": "10.0.1.0/24"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "action": "added",
        "ip_address": "10.0.1.0/24",
        "message": "IP address added to whitelist"
      }

Manage Blocked Ports
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/firewall/ports

   Block or unblock network ports.
   
   **UI Element**: SECTION E - Blocked ports management sidebar
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "action": "block",
        "port": 8080,
        "protocol": "TCP"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "action": "blocked",
        "port": 8080,
        "message": "Port blocked successfully"
      }

Update Firewall Settings
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/firewall/settings

   Update firewall configuration.
   
   **UI Element**: SECTION E - "Update Firewall Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "enabled": true,
        "rate_limiting": {
          "enabled": true,
          "requests_per_second": 50
        },
        "dos_protection": true,
        "vpn_enabled": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Firewall settings updated successfully"
      }

Certificate Management (Section F)
----------------------------------

Get Certificates Status
^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/certificates/status

   Get status of all certificates.
   
   **UI Element**: SECTION F - Certificate status display
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "https_certificate": {
          "present": true,
          "valid_to": "2028-01-01T00:00:00Z",
          "days_remaining": 1020,
          "status": "valid"
        },
        "mqtt_certificate": {
          "present": true,
          "valid_to": "2025-03-29T00:00:00Z",
          "days_remaining": 14,
          "status": "expiring"
        },
        "settings": {
          "auto_renew": false
        }
      }

Upload Certificate
^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/certificates/upload

   Upload SSL/TLS certificate files.
   
   **UI Element**: SECTION F - "Upload Certificate" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /api/v1/security/certificates/upload HTTP/1.1
      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="certificate"; filename="server.crt"
      Content-Disposition: form-data; name="private_key"; filename="server.key"
      Content-Disposition: form-data; name="type"; filename="https"

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "uploaded": true,
        "certificate_info": {
          "subject": "CN=gateway.example.com",
          "valid_to": "2025-06-13T00:00:00Z"
        },
        "message": "Certificate uploaded successfully"
      }

Update Certificate Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/certificates/settings

   Update certificate management settings.
   
   **UI Element**: SECTION F - Certificate settings section
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "auto_renew": true,
        "renewal_days_before_expiry": 30,
        "acme_email": "admin@example.com"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "updated": true,
        "message": "Certificate settings updated successfully"
      }

Security Audit Log (Section G)
------------------------------

Get Audit Log
^^^^^^^^^^^^^

.. http:get:: /api/v1/security/audit/logs

   Get security audit logs for table.
   
   **UI Element**: SECTION G - Audit log table
   
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
            "timestamp": "2025-03-15T10:15:00Z",
            "user": "admin",
            "event": "Login",
            "resource": "/api/auth/login",
            "ip_address": "192.168.1.100",
            "status": "success"
          }
        ],
        "total": 125,
        "filtered": 1
      }

Export Audit Log
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/audit/export

   Export audit logs to CSV file.
   
   **UI Element**: SECTION G - "Export CSV" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="audit_log_20250315.csv"
      
      timestamp,user,event,resource,ip_address,status
      2025-03-15T10:15:00Z,admin,Login,/api/auth/login,192.168.1.100,success

Get Audit Statistics
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/audit/stats

   Get audit log statistics.
   
   **UI Element**: SECTION G - Statistics summary
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "last_24_hours": {
          "total_events": 45,
          "successful": 42,
          "failed": 3,
          "login_attempts": 15,
          "ssh_sessions": 8
        },
        "top_users": [
          {"username": "admin", "events": 25}
        ],
        "top_ips": [
          {"ip": "192.168.1.100", "events": 30}
        ]
      }

Security Operations
-------------------

Run Security Scan
^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/scan

   Run security vulnerability scan.
   
   **UI Element**: Footer - "Run Security Scan" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "scan_type": "full",
        "components": ["authentication", "firewall", "certificates"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "started": true,
        "scan_id": "scan-20250315-001",
        "started_at": "2025-03-15T10:30:00Z",
        "estimated_completion": "2025-03-15T10:35:00Z"
      }

Get Scan Results
^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/scan/results

   Get security scan results.
   
   **UI Element**: Footer - Security scan results display
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-20250315-001",
        "results": {
          "authentication": {
            "status": "pass",
            "score": 95,
            "issues": 0
          },
          "firewall": {
            "status": "warning",
            "score": 80,
            "issues": 2
          }
        },
        "total_issues": 4,
        "security_score": 84,
        "recommendations": ["Enable 2FA for all users"]
      }

Lock System
^^^^^^^^^^^

.. http:post:: /api/v1/security/lock

   Lock system access (emergency).
   
   **UI Element**: Footer - "Lock System" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "reason": "Security maintenance",
        "duration_minutes": 30
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "locked": true,
        "locked_until": "2025-03-15T11:00:00Z",
        "message": "System locked successfully"
      }

Reset Security Settings
^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/reset

   Reset security settings to defaults.
   
   **UI Element**: Footer - "Reset to Default" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "confirm": true,
        "preserve_users": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "reset": true,
        "message": "Security settings reset to defaults"
      }

Save All Security Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /api/v1/security/settings/save-all

   Save all security settings at once.
   
   **UI Element**: Footer - "Save Security Settings" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: application/json

   **Request**:

   .. sourcecode:: json

      {
        "authentication": {
          "password_policy": {"min_length": 12},
          "two_factor": {"mode": "required"}
        },
        "firewall": {
          "enabled": true,
          "default_policy": "DROP"
        },
        "ssh": {
          "enabled": true,
          "port": 2222
        },
        "certificates": {
          "auto_renew": true
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "saved": true,
        "timestamp": "2025-03-15T10:30:00Z",
        "message": "All security settings saved"
      }

Backup Security Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /api/v1/security/backup

   Backup security configuration.
   
   **UI Element**: Footer - "Backup Configuration" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backup_id": "backup-20250315-001",
        "filename": "security_backup_20250315.tar.gz",
        "download_url": "/api/v1/security/backup/download/backup-20250315-001",
        "size": "1.5 MB"
      }

Error Codes
-----------

.. list-table:: Simplified Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - SECURITY_ACCESS_DENIED
     - Insufficient permissions
   * - USER_EXISTS
     - Username already exists
   * - USER_NOT_FOUND
     - User does not exist
   * - INVALID_CREDENTIALS
     - Invalid login credentials
   * - RATE_LIMITED
     - Too many login attempts
   * - PASSWORD_POLICY_VIOLATION
     - Password doesn't meet policy
   * - API_KEY_INVALID
     - Invalid API key
   * - FIREWALL_RULE_CONFLICT
     - Conflicting firewall rule
   * - CERTIFICATE_EXPIRED
     - Certificate has expired
   * - SSH_KEY_INVALID
     - Invalid SSH key format
   * - AUDIT_LOG_FULL
     - Audit log storage full
   * - SYSTEM_LOCKED
     - System is locked
   * - BACKUP_FAILED
     - Backup creation failed
   * - SCAN_IN_PROGRESS
     - Security scan already running

Authentication
--------------

All endpoints require:

.. code-block:: http

   Authorization: Bearer <jwt_token>
   X-Gateway-ID: GW-3920A9

Rate Limiting
-------------

* 100 requests per minute per user
* Authentication endpoints: 10 requests per minute per IP
* File Uploads: 10MB max file size, 5 concurrent uploads
* Rate limit headers included in responses

Response Format
---------------

All successful responses:

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
       "code": "SECURITY_ACCESS_DENIED",
       "message": "Insufficient permissions"
     },
     "success": false
   }
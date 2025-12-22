Security & Access Control
=========================

This document describes the security and access control management page and its related API endpoints for managing user access, authentication policies, network security, and system protection.

Page Route (Frontend)
---------------------

.. http:get:: /security-access-control

   **Description**: Renders the complete security and access control management page with all security information, user data, authentication settings, and network security embedded in the HTML.

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
        <title>Security & Access Control - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-security='{...}' data-users='[...]' data-settings='{...}' data-firewall='{...}'>
          <!-- Security management page with:
               SECTION A: Identity & Access Management
               SECTION B: Authentication Policies
               SECTION C: API Keys & Tokens
               SECTION D: SSH Access Control
               SECTION E: Firewall & Network Security
               SECTION F: Certificate Management
               SECTION G: Security Audit Log
          -->
        </div>
      </body>
      </html>

   **How it works**:
   - Server renders the HTML page with embedded security and user data
   - All security status, user information, firewall rules, and audit logs are embedded
   - JavaScript reads this data and renders the complete security management interface
   - No separate API calls needed on initial page load
   - Gateway context is provided via X-Gateway-ID header

   **Error Response**:

   .. sourcecode:: http

      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all security and access control operations triggered from the page. All endpoints operate on the current gateway context identified by the `X-Gateway-ID` header.

Base Path: ``/api/v1/security``

Security Status (Overview)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/status

   **Description**: Get overall security status and component health.
   
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
        },
        "success": true,
        "message": "Security status retrieved successfully"
      }

Identity & Access Management (SECTION A)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/users

   **Description**: Get all system users for user table.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Users retrieved successfully"
      }

.. http:post:: /api/v1/security/users

   **Description**: Create new user from "Add User" modal.
   
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
        "data": {
          "created": true,
          "user_id": 9
        },
        "success": true,
        "message": "User created successfully"
      }

.. http:put:: /api/v1/security/users/{user_id}

   **Description**: Update user from "Edit User" modal.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "User updated successfully"
      }

.. http:delete:: /api/v1/security/users/{user_id}

   **Description**: Delete user from "Edit User" modal.
   
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
        "data": {
          "deleted": true
        },
        "success": true,
        "message": "User deleted successfully"
      }

.. http:get:: /api/v1/security/roles

   **Description**: Get all roles for sidebar display.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Roles retrieved successfully"
      }

Authentication Policies (SECTION B)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/authentication/settings

   **Description**: Get all authentication policy settings.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Authentication settings retrieved successfully"
      }

.. http:post:: /api/v1/security/authentication/settings

   **Description**: Save authentication policy changes.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "Authentication settings updated successfully"
      }

.. http:post:: /api/v1/security/authentication/test-password

   **Description**: Test password strength.
   
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
        "data": {
          "strength": "strong",
          "score": 92,
          "meets_policy": true,
          "issues": []
        },
        "success": true,
        "message": "Password test completed"
      }

API Keys & Tokens (SECTION C)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/api-keys

   **Description**: Get all API keys for table display.
   
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
        "data": {
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
        },
        "success": true,
        "message": "API keys retrieved successfully"
      }

.. http:post:: /api/v1/security/api-keys

   **Description**: Generate new API key from modal.
   
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
        "data": {
          "generated": true,
          "api_key": {
            "id": 9,
            "name": "Webhook Integration",
            "full_key": "sk_9b3e7c2a1d8f5e6c4b9a2d7e1c3f8a5b",
            "key_prefix": "AK-009"
          }
        },
        "success": true,
        "message": "API key generated successfully"
      }

.. http:post:: /api/v1/security/api-keys/{key_id}/revoke

   **Description**: Revoke API key from table actions.
   
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
        "data": {
          "revoked": true
        },
        "success": true,
        "message": "API key revoked successfully"
      }

.. http:get:: /api/v1/security/tokens/settings

   **Description**: Get token settings for form.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Token settings retrieved successfully"
      }

.. http:post:: /api/v1/security/tokens/settings

   **Description**: Update token settings.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "Token settings updated successfully"
      }

.. http:post:: /api/v1/security/tokens/regenerate-secret

   **Description**: Regenerate webhook signing secret.
   
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
        "data": {
          "regenerated": true,
          "new_secret": "whsec_xyz789uvw012"
        },
        "success": true,
        "message": "Webhook secret regenerated successfully"
      }

SSH Access Control (SECTION D)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/ssh/settings

   **Description**: Get SSH configuration for form.
   
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
        "data": {
          "enabled": true,
          "port": 22,
          "login_method": "key",
          "restricted_shell": true,
          "session_timeout": 30,
          "max_sessions_per_user": 3,
          "allowed_commands": ["ls", "cat", "ifconfig", "ping"]
        },
        "success": true,
        "message": "SSH settings retrieved successfully"
      }

.. http:post:: /api/v1/security/ssh/settings

   **Description**: Save SSH configuration changes.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "SSH settings updated successfully"
      }

.. http:post:: /api/v1/security/ssh/keys

   **Description**: Upload SSH public key for authentication.
   
   **UI Element**: SECTION D - "Upload SSH Key" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **file** (required): SSH public key file

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "uploaded": true,
          "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890"
        },
        "success": true,
        "message": "SSH public key uploaded successfully"
      }

.. http:get:: /api/v1/security/ssh/sessions

   **Description**: Get active SSH sessions for monitoring.
   
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
        "data": {
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
        },
        "success": true,
        "message": "SSH sessions retrieved successfully"
      }

Firewall & Network Security (SECTION E)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/firewall

   **Description**: Get firewall status and rules.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Firewall status retrieved successfully"
      }

.. http:post:: /api/v1/security/firewall/rules

   **Description**: Add new firewall rule from modal.
   
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
        "data": {
          "added": true,
          "rule_id": 16
        },
        "success": true,
        "message": "Firewall rule added successfully"
      }

.. http:delete:: /api/v1/security/firewall/rules/{rule_id}

   **Description**: Remove firewall rule from table.
   
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
        "data": {
          "removed": true
        },
        "success": true,
        "message": "Firewall rule removed successfully"
      }

.. http:post:: /api/v1/security/firewall/whitelist

   **Description**: Add or remove IP addresses from whitelist.
   
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
        "data": {
          "updated": true,
          "action": "added",
          "ip_address": "10.0.1.0/24"
        },
        "success": true,
        "message": "IP address added to whitelist"
      }

.. http:post:: /api/v1/security/firewall/ports

   **Description**: Block or unblock network ports.
   
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
        "data": {
          "updated": true,
          "action": "blocked",
          "port": 8080
        },
        "success": true,
        "message": "Port blocked successfully"
      }

.. http:post:: /api/v1/security/firewall/settings

   **Description**: Update firewall configuration.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "Firewall settings updated successfully"
      }

Certificate Management (SECTION F)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/certificates/status

   **Description**: Get status of all certificates.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Certificate status retrieved successfully"
      }

.. http:post:: /api/v1/security/certificates/upload

   **Description**: Upload SSL/TLS certificate files.
   
   **UI Element**: SECTION F - "Upload Certificate" button
   
   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      X-Gateway-ID: GW-3920A9
      Content-Type: multipart/form-data

   **Form Parameters**:

   * **certificate** (required): Certificate file (.crt, .pem)
   * **private_key** (required): Private key file (.key)
   * **type** (required): Certificate type (https, mqtt)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "data": {
          "uploaded": true,
          "certificate_info": {
            "subject": "CN=gateway.example.com",
            "valid_to": "2025-06-13T00:00:00Z"
          }
        },
        "success": true,
        "message": "Certificate uploaded successfully"
      }

.. http:post:: /api/v1/security/certificates/settings

   **Description**: Update certificate management settings.
   
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
        "data": {
          "updated": true
        },
        "success": true,
        "message": "Certificate settings updated successfully"
      }

Security Audit Log (SECTION G)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/v1/security/audit/logs

   **Description**: Get security audit logs for table.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Audit logs retrieved successfully"
      }

.. http:get:: /api/v1/security/audit/export

   **Description**: Export audit logs to CSV file.
   
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

.. http:get:: /api/v1/security/audit/stats

   **Description**: Get audit log statistics.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Audit statistics retrieved successfully"
      }

Security Operations
~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/v1/security/scan

   **Description**: Run security vulnerability scan.
   
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
        "data": {
          "started": true,
          "scan_id": "scan-20250315-001",
          "started_at": "2025-03-15T10:30:00Z",
          "estimated_completion": "2025-03-15T10:35:00Z"
        },
        "success": true,
        "message": "Security scan started"
      }

.. http:get:: /api/v1/security/scan/results

   **Description**: Get security scan results.
   
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
        "data": {
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
        },
        "success": true,
        "message": "Security scan results retrieved"
      }

.. http:post:: /api/v1/security/lock

   **Description**: Lock system access (emergency).
   
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
        "data": {
          "locked": true,
          "locked_until": "2025-03-15T11:00:00Z"
        },
        "success": true,
        "message": "System locked successfully"
      }

.. http:post:: /api/v1/security/reset

   **Description**: Reset security settings to defaults.
   
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
        "data": {
          "reset": true
        },
        "success": true,
        "message": "Security settings reset to defaults"
      }

.. http:post:: /api/v1/security/settings/save-all

   **Description**: Save all security settings at once.
   
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
        "data": {
          "saved": true,
          "timestamp": "2025-03-15T10:30:00Z"
        },
        "success": true,
        "message": "All security settings saved"
      }

.. http:get:: /api/v1/security/backup

   **Description**: Backup security configuration.
   
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
        "data": {
          "backup_id": "backup-20250315-001",
          "filename": "security_backup_20250315.tar.gz",
          "download_url": "/api/v1/security/backup/download/backup-20250315-001",
          "size": "1.5 MB"
        },
        "success": true,
        "message": "Security configuration backup created"
      }

Route Summary
-------------

.. list-table:: Security & Access Control Routes
   :header-rows: 1
   :widths: 15 15 50 20
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/security-access-control`` - Main security page
     - Yes
   * - Security Status
     - GET
     - ``/api/v1/security/status`` - Security overview
     - Yes
   * - Users
     - GET
     - ``/api/v1/security/users`` - Get all users
     - Yes
   * - Users
     - POST
     - ``/api/v1/security/users`` - Create user
     - Yes
   * - Users
     - PUT
     - ``/api/v1/security/users/{id}`` - Update user
     - Yes
   * - Users
     - DELETE
     - ``/api/v1/security/users/{id}`` - Delete user
     - Yes
   * - Roles
     - GET
     - ``/api/v1/security/roles`` - Get roles
     - Yes
   * - Authentication
     - GET
     - ``/api/v1/security/authentication/settings`` - Get auth settings
     - Yes
   * - Authentication
     - POST
     - ``/api/v1/security/authentication/settings`` - Update auth settings
     - Yes
   * - Authentication
     - POST
     - ``/api/v1/security/authentication/test-password`` - Test password
     - Yes
   * - API Keys
     - GET
     - ``/api/v1/security/api-keys`` - Get API keys
     - Yes
   * - API Keys
     - POST
     - ``/api/v1/security/api-keys`` - Generate API key
     - Yes
   * - API Keys
     - POST
     - ``/api/v1/security/api-keys/{id}/revoke`` - Revoke API key
     - Yes
   * - Tokens
     - GET
     - ``/api/v1/security/tokens/settings`` - Get token settings
     - Yes
   * - Tokens
     - POST
     - ``/api/v1/security/tokens/settings`` - Update token settings
     - Yes
   * - Tokens
     - POST
     - ``/api/v1/security/tokens/regenerate-secret`` - Regenerate webhook secret
     - Yes
   * - SSH
     - GET
     - ``/api/v1/security/ssh/settings`` - Get SSH settings
     - Yes
   * - SSH
     - POST
     - ``/api/v1/security/ssh/settings`` - Update SSH settings
     - Yes
   * - SSH
     - POST
     - ``/api/v1/security/ssh/keys`` - Upload SSH key
     - Yes
   * - SSH
     - GET
     - ``/api/v1/security/ssh/sessions`` - Get SSH sessions
     - Yes
   * - Firewall
     - GET
     - ``/api/v1/security/firewall`` - Get firewall status
     - Yes
   * - Firewall
     - POST
     - ``/api/v1/security/firewall/rules`` - Add firewall rule
     - Yes
   * - Firewall
     - DELETE
     - ``/api/v1/security/firewall/rules/{id}`` - Remove firewall rule
     - Yes
   * - Firewall
     - POST
     - ``/api/v1/security/firewall/whitelist`` - Manage IP whitelist
     - Yes
   * - Firewall
     - POST
     - ``/api/v1/security/firewall/ports`` - Manage blocked ports
     - Yes
   * - Firewall
     - POST
     - ``/api/v1/security/firewall/settings`` - Update firewall settings
     - Yes
   * - Certificates
     - GET
     - ``/api/v1/security/certificates/status`` - Get certificate status
     - Yes
   * - Certificates
     - POST
     - ``/api/v1/security/certificates/upload`` - Upload certificate
     - Yes
   * - Certificates
     - POST
     - ``/api/v1/security/certificates/settings`` - Update certificate settings
     - Yes
   * - Audit Log
     - GET
     - ``/api/v1/security/audit/logs`` - Get audit logs
     - Yes
   * - Audit Log
     - GET
     - ``/api/v1/security/audit/export`` - Export audit logs
     - Yes
   * - Audit Log
     - GET
     - ``/api/v1/security/audit/stats`` - Get audit statistics
     - Yes
   * - Security Ops
     - POST
     - ``/api/v1/security/scan`` - Run security scan
     - Yes
   * - Security Ops
     - GET
     - ``/api/v1/security/scan/results`` - Get scan results
     - Yes
   * - Security Ops
     - POST
     - ``/api/v1/security/lock`` - Lock system
     - Yes
   * - Security Ops
     - POST
     - ``/api/v1/security/reset`` - Reset settings
     - Yes
   * - Security Ops
     - POST
     - ``/api/v1/security/settings/save-all`` - Save all settings
     - Yes
   * - Security Ops
     - GET
     - ``/api/v1/security/backup`` - Backup configuration
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /security-access-control``
   - Server renders HTML with embedded security and user data
   - All 7 sections populated with current data

2. **User views security overview**:
   - Security score and status badges
   - Component health indicators

3. **User manages identity & access** (SECTION A):
   - User management table with create/edit/delete
   - Role definitions and permissions
   - Password strength testing

4. **User configures authentication** (SECTION B):
   - Password policy settings
   - Account protection (lockout, captcha)
   - Two-factor authentication settings
   - Session management

5. **User manages API keys & tokens** (SECTION C):
   - API key generation and revocation
   - Token permission settings
   - Webhook secret management

6. **User controls SSH access** (SECTION D):
   - SSH configuration (port, login methods)
   - SSH key upload and management
   - Active session monitoring

7. **User configures firewall** (SECTION E):
   - Firewall rule management
   - IP whitelist management
   - Port blocking configuration
   - Security features (rate limiting, DDoS protection)

8. **User manages certificates** (SECTION F):
   - Certificate status monitoring
   - Certificate upload and renewal
   - SSL/TLS configuration

9. **User monitors audit logs** (SECTION G):
   - Security event auditing
   - Export capabilities
   - Statistics and reporting

10. **User performs security operations**:
    - Run security vulnerability scans
    - Lock system for maintenance
    - Reset security settings
    - Backup security configuration
    - Save all security settings

Error Codes
-----------

.. list-table:: Security Error Codes
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

Authentication & Context
------------------------

All endpoints require gateway context identification::

   Authorization: Bearer <token>
   X-Gateway-ID: GW-3920A9

**Important**: This API only manages security for the current gateway specified in `X-Gateway-ID` header.

Support Information
-------------------

- **Security Support**: security-support@univa.com
- **Emergency Response**: +1 (555) 789-0456
- **Support Hours**: 24/7 for security incidents
- **Documentation**: https://docs.univa.com/security
- **Security Status**: https://status.univa.com/security

---
*Document last updated: March 15, 2025*
*API Version: 1.0.0*
*Security Module Version: 2.1.0*
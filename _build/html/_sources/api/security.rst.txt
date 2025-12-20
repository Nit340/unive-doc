Security & Access Control API
=============================

APIs for managing system security, user access control, authentication policies, and network security.

Overview
--------

The Security & Access Control API provides comprehensive security management capabilities including user management, authentication policies, API key management, SSH access control, firewall configuration, certificate management, and security auditing.

Security Status
---------------

Get Security Status
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/status

   Get overall security status and component health.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "security_level": "secure",
        "security_score": 92,
        "last_scan": "2025-03-15T10:30:00Z",
        "issues_found": 0,
        "recommendations": ["Enable 2FA for all users"],
        "components": {
          "authentication": {
            "status": "enabled",
            "health": "good",
            "issues": 0
          },
          "firewall": {
            "status": "active",
            "health": "good",
            "rules_active": 15
          },
          "ssh": {
            "status": "restricted",
            "health": "good",
            "active_sessions": 1
          },
          "certificates": {
            "status": "valid",
            "health": "warning",
            "expiring_soon": 1
          },
          "audit": {
            "status": "enabled",
            "health": "good",
            "events_24h": 45
          }
        }
      }

Identity & Access Management
----------------------------

Get Users List
~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/users

   Retrieve all system users with details.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional): Filter by status (active, inactive, locked)
   * **role** (optional): Filter by role
   * **limit** (optional): Number of results (default: 50, max: 200)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "users": [
          {
            "id": 1,
            "username": "admin",
            "full_name": "System Administrator",
            "email": "admin@example.com",
            "role": "admin",
            "status": "active",
            "last_login": "2025-03-15T10:15:00Z",
            "login_count": 127,
            "two_factor_enabled": true,
            "created_at": "2024-01-15T09:00:00Z",
            "avatar_color": "indigo",
            "permissions": ["*"],
            "session_count": 1
          },
          {
            "id": 2,
            "username": "ops01",
            "full_name": "Operations User 01",
            "email": "ops01@example.com",
            "role": "operator",
            "status": "active",
            "last_login": "2025-03-10T14:30:00Z",
            "login_count": 45,
            "two_factor_enabled": false,
            "created_at": "2024-02-20T10:30:00Z",
            "avatar_color": "orange",
            "permissions": ["device:read", "device:write", "config:read"],
            "session_count": 0
          }
        ],
        "total": 8,
        "active": 7,
        "roles_summary": {
          "admin": 1,
          "engineer": 2,
          "operator": 3,
          "viewer": 2
        }
      }

Create User
~~~~~~~~~~~

.. http:post:: /cloud-integration/security/users/create

   Create a new system user.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/users/create HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "username": "tech02",
        "full_name": "Technician 02",
        "email": "tech02@example.com",
        "role": "technician",
        "password": "SecurePass123!",
        "two_factor_enabled": true,
        "permissions": ["device:read", "device:maintenance", "alerts:read"],
        "send_welcome_email": true,
        "force_password_change": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "created",
        "user_id": 9,
        "user": {
          "id": 9,
          "username": "tech02",
          "full_name": "Technician 02",
          "email": "tech02@example.com",
          "role": "technician",
          "status": "active",
          "created_at": "2025-03-15T10:30:00Z",
          "permissions": ["device:read", "device:maintenance", "alerts:read"]
        },
        "message": "User created successfully",
        "welcome_email_sent": true,
        "temporary_password": "TempPass123!"  // Only if not provided
      }

Update User
~~~~~~~~~~~

.. http:post:: /cloud-integration/security/users/update

   Update user information and permissions.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/users/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "user_id": 2,
        "full_name": "Updated Operations User",
        "email": "ops.updated@example.com",
        "role": "engineer",
        "two_factor_enabled": true,
        "permissions": ["device:read", "device:write", "config:read", "config:write", "diagnostics:run"],
        "status": "active",
        "notes": "Promoted to engineer role"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "user": {
          "id": 2,
          "username": "ops01",
          "full_name": "Updated Operations User",
          "email": "ops.updated@example.com",
          "role": "engineer",
          "two_factor_enabled": true,
          "permissions": ["device:read", "device:write", "config:read", "config:write", "diagnostics:run"]
        },
        "message": "User updated successfully",
        "notify_user": true,
        "changes": {
          "role": {"old": "operator", "new": "engineer"},
          "two_factor_enabled": {"old": false, "new": true},
          "permissions": {"added": ["config:write", "diagnostics:run"]}
        }
      }

Delete User
~~~~~~~~~~~

.. http:post:: /cloud-integration/security/users/delete

   Delete a user from the system.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/users/delete HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "user_id": 3,
        "transfer_data_to": 1,  // Optional: Transfer owned data to another user
        "archive_data": true,
        "reason": "User no longer requires access"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "deleted",
        "user_id": 3,
        "message": "User deleted successfully",
        "data_transferred": true,
        "data_archived": true,
        "sessions_terminated": 2
      }

Get User Details
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/users/{user_id}

   Get detailed information about a specific user.

   **Path Parameters**:

   * **user_id** (integer): User identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": 1,
        "username": "admin",
        "full_name": "System Administrator",
        "email": "admin@example.com",
        "role": "admin",
        "status": "active",
        "last_login": "2025-03-15T10:15:00Z",
        "last_password_change": "2025-02-15T09:30:00Z",
        "login_count": 127,
        "failed_attempts": 0,
        "two_factor_enabled": true,
        "two_factor_method": "app",
        "created_at": "2024-01-15T09:00:00Z",
        "updated_at": "2025-03-10T14:20:00Z",
        "permissions": ["*"],
        "api_keys": [
          {
            "name": "Admin Script",
            "scope": ["*"],
            "created": "2025-01-20T10:30:00Z"
          }
        ],
        "ssh_keys": [
          {
            "fingerprint": "SHA256:AbCdEf...",
            "type": "RSA",
            "added": "2025-02-01T11:00:00Z"
          }
        ],
        "recent_activity": [
          {
            "timestamp": "2025-03-15T10:15:00Z",
            "action": "login",
            "ip": "192.168.1.100",
            "status": "success"
          }
        ]
      }

Get Roles
~~~~~~~~~

.. http:get:: /cloud-integration/security/roles

   Get all available roles with permissions.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "roles": [
          {
            "name": "Admin",
            "description": "Full system access. Can manage all aspects including users, security, and system configuration.",
            "permissions": ["*"],
            "user_count": 1,
            "default": false,
            "protected": true,
            "permission_groups": {
              "system": ["system:read", "system:write", "system:admin"],
              "users": ["users:read", "users:write", "users:admin"],
              "devices": ["devices:read", "devices:write", "devices:admin"],
              "config": ["config:read", "config:write", "config:admin"],
              "security": ["security:read", "security:write", "security:admin"]
            }
          },
          {
            "name": "Engineer",
            "description": "Technical configuration and diagnostics access. Can configure devices, run diagnostics, and manage tags.",
            "permissions": [
              "devices:read", "devices:write", "devices:configure",
              "config:read", "config:write", 
              "diagnostics:run", "diagnostics:read",
              "tags:read", "tags:write"
            ],
            "user_count": 3,
            "default": false,
            "protected": false
          },
          {
            "name": "Technician",
            "description": "Device maintenance and sensor management. Can view devices, perform maintenance, and view basic data.",
            "permissions": [
              "devices:read", "devices:maintenance",
              "alerts:read",
              "reports:read"
            ],
            "user_count": 2,
            "default": false,
            "protected": false
          },
          {
            "name": "Viewer",
            "description": "Read-only access. Can view devices, data, and reports but cannot make changes.",
            "permissions": [
              "devices:read",
              "alerts:read",
              "reports:read",
              "dashboard:read"
            ],
            "user_count": 2,
            "default": true,
            "protected": false
          }
        ],
        "permission_categories": {
          "system": ["system:read", "system:write", "system:admin"],
          "users": ["users:read", "users:write", "users:admin"],
          "devices": ["devices:read", "devices:write", "devices:configure", "devices:admin", "devices:maintenance"],
          "config": ["config:read", "config:write", "config:admin"],
          "security": ["security:read", "security:write", "security:admin"],
          "diagnostics": ["diagnostics:read", "diagnostics:run"],
          "tags": ["tags:read", "tags:write"],
          "alerts": ["alerts:read", "alerts:write"],
          "reports": ["reports:read", "reports:write"],
          "dashboard": ["dashboard:read"]
        }
      }

Reset User Password
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/users/reset-password

   Reset a user's password.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/users/reset-password HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "user_id": 2,
        "new_password": "NewSecurePass456!",
        "force_change": true,
        "send_email": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "reset",
        "user_id": 2,
        "username": "ops01",
        "message": "Password reset successfully",
        "email_sent": true,
        "requires_change": true
      }

Lock/Unlock User Account
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/users/lock

   Lock or unlock a user account.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/users/lock HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "user_id": 4,
        "action": "lock",  // or "unlock"
        "reason": "Multiple failed login attempts",
        "duration_minutes": 60  // Optional for lock
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "locked",
        "user_id": 4,
        "username": "user04",
        "locked_until": "2025-03-15T11:30:00Z",
        "message": "User account locked due to security concerns",
        "notified": true
      }

Authentication Policies
-----------------------

Get Authentication Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/authentication/settings

   Get current authentication and password policies.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "password_policy": {
          "min_length": 8,
          "max_length": 64,
          "expiry_days": 90,
          "history_size": 5,
          "require_special_chars": true,
          "require_numbers": true,
          "require_uppercase": true,
          "require_lowercase": true,
          "prevent_reuse": true,
          "prevent_common": true,
          "prevent_sequential": true,
          "prevent_username": true
        },
        "account_protection": {
          "lockout_attempts": 5,
          "lockout_duration": 15,
          "lockout_window": 15,
          "captcha_enabled": true,
          "captcha_threshold": 3,
          "ip_blacklist_enabled": true,
          "ip_blacklist_threshold": 10
        },
        "session_settings": {
          "timeout_minutes": 15,
          "max_concurrent_sessions": 3,
          "session_renewal": true,
          "idle_timeout": 30,
          "remember_me_days": 30,
          "secure_cookies": true,
          "http_only_cookies": true
        },
        "two_factor": {
          "mode": "optional",
          "enforced_for_roles": ["admin"],
          "methods": ["app", "sms", "email"],
          "default_method": "app",
          "backup_codes_count": 10,
          "recovery_enabled": true
        },
        "brute_force_protection": {
          "enabled": true,
          "max_attempts_per_ip": 50,
          "window_minutes": 60,
          "block_duration": 1440
        }
      }

Update Authentication Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/authentication/update

   Update authentication settings and policies.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/authentication/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "password_policy": {
          "min_length": 12,
          "expiry_days": 60,
          "require_special_chars": true,
          "require_numbers": true,
          "require_uppercase": true,
          "history_size": 10
        },
        "account_protection": {
          "lockout_attempts": 3,
          "lockout_duration": 30,
          "captcha_enabled": false
        },
        "session_settings": {
          "timeout_minutes": 30,
          "max_concurrent_sessions": 5,
          "idle_timeout": 60
        },
        "two_factor": {
          "mode": "required",
          "enforced_for_roles": ["admin", "engineer"],
          "methods": ["app", "sms"],
          "default_method": "app"
        },
        "brute_force_protection": {
          "max_attempts_per_ip": 100,
          "block_duration": 2880
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "message": "Authentication settings updated successfully",
        "affected_users": 5,
        "notifications_sent": true,
        "changes": {
          "password_policy.min_length": {"old": 8, "new": 12},
          "two_factor.mode": {"old": "optional", "new": "required"},
          "session_settings.timeout_minutes": {"old": 15, "new": 30}
        }
      }

Test Password Strength
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/authentication/test-password

   Test password strength against current policy.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/authentication/test-password HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "password": "MySecurePass123!",
        "username": "user01",  // Optional: For checking against username
        "old_passwords": ["OldPass123!", "Previous456@"]  // Optional: For reuse check
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "strength": "strong",
        "score": 92,
        "meets_policy": true,
        "issues": [],
        "suggestions": ["Consider adding special characters"],
        "checks": {
          "length": {"passed": true, "actual": 16, "required": 12},
          "uppercase": {"passed": true, "count": 2},
          "lowercase": {"passed": true, "count": 10},
          "numbers": {"passed": true, "count": 3},
          "special": {"passed": true, "count": 1},
          "sequential": {"passed": true},
          "common": {"passed": true},
          "username_match": {"passed": true},
          "reuse": {"passed": true}
        },
        "estimated_crack_time": "centuries"
      }

API Keys & Tokens
-----------------

Get API Keys
~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/api-keys

   Get all API keys with details.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **status** (optional): Filter by status (active, expired, revoked)
   * **user_id** (optional): Filter by user
   * **limit** (optional): Number of results (default: 50)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "api_keys": [
          {
            "id": 1,
            "name": "CI/CD Pipeline",
            "key_prefix": "AK-001",
            "full_key": "sk_3fa85f64c471b238f0c0c4a1...",  // Only on creation
            "scope": ["telemetry:write", "device:read", "alerts:read"],
            "user": {
              "id": 1,
              "username": "admin"
            },
            "created": "2025-01-15T10:30:00Z",
            "last_used": "2025-03-15T09:45:00Z",
            "expires": "2026-06-01T00:00:00Z",
            "status": "active",
            "usage_count": 1250,
            "rate_limit": 1000,
            "ip_restrictions": ["192.168.1.0/24"],
            "description": "Used for automated CI/CD deployments"
          },
          {
            "id": 2,
            "name": "Mobile App Integration",
            "key_prefix": "AK-002",
            "scope": ["telemetry:read", "alerts:read", "dashboard:read"],
            "user": {
              "id": 2,
              "username": "ops01"
            },
            "created": "2025-02-20T14:15:00Z",
            "last_used": "2025-03-14T16:30:00Z",
            "expires": null,
            "status": "active",
            "usage_count": 340,
            "rate_limit": 100,
            "ip_restrictions": [],
            "description": "Mobile application API access"
          },
          {
            "id": 3,
            "name": "Webhook Service",
            "key_prefix": "AK-003",
            "scope": ["alerts:write", "events:write"],
            "user": {
              "id": 1,
              "username": "admin"
            },
            "created": "2024-12-01T11:20:00Z",
            "last_used": "2024-12-15T14:00:00Z",
            "expires": "2025-01-01T00:00:00Z",
            "status": "expired",
            "usage_count": 45,
            "rate_limit": 500,
            "description": "External monitoring service integration"
          }
        ],
        "total": 8,
        "active": 5,
        "expired": 2,
        "revoked": 1
      }

Generate API Key
~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/api-keys/generate

   Generate a new API key.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/api-keys/generate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Webhook Integration",
        "description": "For external webhook notifications",
        "scope": ["alerts:read", "alerts:write", "events:read"],
        "user_id": 2,
        "expiry_days": 90,
        "rate_limit": 100,
        "ip_restrictions": ["10.0.0.0/8", "192.168.1.100"],
        "notification_on_creation": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "generated",
        "api_key": {
          "id": 9,
          "name": "Webhook Integration",
          "full_key": "sk_9b3e7c2a1d8f5e6c4b9a2d7e1c3f8a5b",
          "key_prefix": "AK-009",
          "scope": ["alerts:read", "alerts:write", "events:read"],
          "created": "2025-03-15T10:30:00Z",
          "expires": "2025-06-15T10:30:00Z",
          "rate_limit": 100,
          "ip_restrictions": ["10.0.0.0/8", "192.168.1.100"]
        },
        "warning": "Store this key securely. The full key will not be shown again.",
        "download_url": "/cloud-integration/security/api-keys/download/9",
        "notifications_sent": true
      }

Revoke API Key
~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/api-keys/revoke

   Revoke an active API key.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/api-keys/revoke HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "key_id": 1,
        "reason": "Security rotation",
        "notify_user": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "revoked",
        "key_id": 1,
        "key_name": "CI/CD Pipeline",
        "message": "API key revoked successfully",
        "user_notified": true,
        "sessions_terminated": 3
      }

Update API Key
~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/api-keys/update

   Update API key properties.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/api-keys/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "key_id": 2,
        "name": "Mobile App v2",
        "description": "Updated mobile app integration",
        "scope": ["telemetry:read", "alerts:read", "dashboard:read", "config:read"],
        "expiry_days": 180,
        "rate_limit": 200,
        "ip_restrictions": ["0.0.0.0/0"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "key_id": 2,
        "key_name": "Mobile App v2",
        "message": "API key updated successfully",
        "changes": {
          "scope": {"added": ["config:read"]},
          "rate_limit": {"old": 100, "new": 200},
          "expiry": {"old": null, "new": "2025-09-15T10:30:00Z"}
        }
      }

Get Token Settings
~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/tokens/settings

   Get token and webhook settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "token_settings": {
          "jwt_expiry_days": 30,
          "refresh_token_expiry_days": 90,
          "issuer": "IoT Gateway",
          "audience": ["gateway-web", "gateway-mobile"],
          "algorithm": "HS256"
        },
        "permission_mapping": {
          "admin": ["*"],
          "engineer": ["device:read", "device:write", "config:read", "config:write", "diagnostics:run"],
          "operator": ["device:read", "device:write", "config:read"],
          "technician": ["device:read", "device:maintenance"],
          "viewer": ["device:read", "alerts:read", "reports:read"]
        },
        "webhook": {
          "secret": "whsec_abc123def456",
          "url": "https://webhook.example.com/events",
          "enabled": true,
          "events": ["login", "logout", "user_created", "user_updated", "user_deleted", "api_key_generated", "api_key_revoked"],
          "retry_count": 3,
          "timeout_seconds": 10,
          "signature_header": "X-Webhook-Signature"
        },
        "rate_limiting": {
          "enabled": true,
          "requests_per_minute": 60,
          "burst_size": 10
        }
      }

Regenerate Webhook Secret
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/tokens/regenerate-secret

   Regenerate webhook signing secret.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/tokens/regenerate-secret HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "invalidate_old": true,
        "notify_webhooks": true,
        "reason": "Security rotation"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "regenerated",
        "new_secret": "whsec_xyz789uvw012",
        "old_secret": "whsec_abc123def456",
        "message": "Webhook secret regenerated successfully",
        "webhooks_notified": true,
        "invalidation_time": "2025-03-15T10:35:00Z"
      }

SSH Access Control
------------------

Get SSH Settings
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/ssh/settings

   Get SSH server configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "port": 22,
        "listen_address": "0.0.0.0",
        "login_method": "key",
        "password_authentication": false,
        "restricted_shell": true,
        "session_timeout": 30,
        "max_sessions_per_user": 3,
        "max_auth_attempts": 3,
        "banner_enabled": true,
        "banner_message": "Authorized access only. All activities are monitored.",
        "allowed_commands": ["ls", "cat", "ifconfig", "ping", "tail", "grep", "df", "ps", "top", "free", "uptime", "who"],
        "allowed_directories": ["/var/log", "/tmp"],
        "public_key_uploaded": true,
        "public_keys": [
          {
            "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
            "type": "RSA",
            "size": 4096,
            "comment": "admin@workstation",
            "added": "2025-02-01T11:00:00Z"
          }
        ],
        "last_connection": "2025-03-15T09:45:00Z",
        "connections_today": 5,
        "failed_attempts_today": 2
      }

Update SSH Settings
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/ssh/update

   Update SSH server configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/ssh/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "port": 2222,
        "listen_address": "192.168.1.100",
        "login_method": "both",
        "password_authentication": true,
        "restricted_shell": true,
        "session_timeout": 45,
        "max_sessions_per_user": 5,
        "max_auth_attempts": 5,
        "banner_enabled": true,
        "banner_message": "WARNING: Unauthorized access is prohibited.",
        "allowed_commands": ["ls", "cat", "tail", "ps", "df", "free", "uptime"],
        "allowed_directories": ["/var/log", "/tmp", "/home/admin"],
        "require_tty": true,
        "compression": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "message": "SSH settings updated successfully",
        "restart_required": true,
        "changes": {
          "port": {"old": 22, "new": 2222},
          "login_method": {"old": "key", "new": "both"},
          "session_timeout": {"old": 30, "new": 45}
        }
      }

Upload SSH Public Key
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/ssh/upload-key

   Upload SSH public key for authentication.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/ssh/upload-key HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="id_rsa.pub"
      Content-Type: text/plain
      
      --form-data; name="username"
      
      admin
      
      --form-data; name="comment"
      
      admin-laptop

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "uploaded",
        "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
        "key_type": "RSA",
        "key_size": 4096,
        "comment": "admin-laptop",
        "username": "admin",
        "added": "2025-03-15T10:30:00Z",
        "message": "SSH public key uploaded successfully"
      }

Get SSH Sessions
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/ssh/sessions

   Get active SSH sessions.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **active_only** (optional): Show only active sessions (default: true)

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
            "client_hostname": "workstation.local",
            "started": "2025-03-15T09:30:00Z",
            "duration": "00:45:30",
            "command_count": 12,
            "last_command": "ls -la",
            "last_command_time": "2025-03-15T10:14:30Z",
            "tty": "pts/0",
            "process_id": 1234,
            "idle_time": "00:02:15"
          }
        ],
        "recent_sessions": [
          {
            "session_id": "ssh-002",
            "username": "ops01",
            "client_ip": "10.0.0.15",
            "started": "2025-03-15T08:45:00Z",
            "ended": "2025-03-15T09:15:00Z",
            "duration": "00:30:00",
            "command_count": 8,
            "termination_reason": "user_exit"
          }
        ],
        "total_sessions_today": 5,
        "unique_users_today": 2,
        "unique_ips_today": 2
      }

Terminate SSH Session
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/ssh/terminate

   Terminate an active SSH session.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/ssh/terminate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "session_id": "ssh-001",
        "reason": "Security maintenance",
        "notify_user": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "terminated",
        "session_id": "ssh-001",
        "username": "admin",
        "message": "SSH session terminated successfully",
        "user_notified": true,
        "session_duration": "00:45:30"
      }

Remove SSH Key
~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/ssh/remove-key

   Remove SSH public key.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/ssh/remove-key HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
        "username": "admin"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "removed",
        "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
        "username": "admin",
        "message": "SSH public key removed successfully"
      }

Firewall & Network Security
---------------------------

Get Firewall Status
~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/firewall/status

   Get firewall status and statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "enabled": true,
        "rules_count": 15,
        "ip_whitelist": ["192.168.0.0/24", "10.0.0.5", "172.16.0.0/16"],
        "ip_blacklist": ["203.0.113.5", "45.33.32.156"],
        "blocked_ports": [23, 21, 8000],
        "allowed_ports": [22, 80, 443, 1883, 8883],
        "security_features": {
          "rate_limiting": {
            "enabled": true,
            "requests_per_second": 20,
            "burst_size": 40
          },
          "dos_protection": true,
          "syn_flood_protection": true,
          "port_scan_detection": true,
          "vpn_enabled": false,
          "geo_blocking": false
        },
        "statistics": {
          "packets_blocked_today": 1245,
          "packets_allowed_today": 89000,
          "unique_ips_blocked": 8,
          "dos_attacks_blocked": 0,
          "port_scans_blocked": 3
        },
        "last_updated": "2025-03-15T09:30:00Z"
      }

Get Firewall Rules
~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/firewall/rules

   Get all firewall rules.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **type** (optional): Filter by type (input, output, forward)
   * **enabled** (optional): Filter by enabled status
   * **limit** (optional): Number of results (default: 100)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "rules": [
          {
            "id": 1,
            "name": "SSH Management Access",
            "description": "Allow SSH from internal network",
            "chain": "INPUT",
            "port": "22",
            "protocol": "TCP",
            "source": "192.168.1.0/24",
            "destination": "0.0.0.0/0",
            "action": "ACCEPT",
            "enabled": true,
            "priority": 100,
            "created": "2024-12-01T10:00:00Z",
            "updated": "2025-01-15T14:30:00Z",
            "hit_count": 1250,
            "last_hit": "2025-03-15T09:45:00Z"
          },
          {
            "id": 2,
            "name": "HTTP Dashboard Access",
            "description": "Allow HTTP access to web dashboard",
            "chain": "INPUT",
            "port": "80",
            "protocol": "TCP",
            "source": "0.0.0.0/0",
            "destination": "0.0.0.0/0",
            "action": "ACCEPT",
            "enabled": true,
            "priority": 200,
            "hit_count": 45000,
            "last_hit": "2025-03-15T10:15:00Z"
          },
          {
            "id": 3,
            "name": "Block Telnet",
            "description": "Block insecure Telnet protocol",
            "chain": "INPUT",
            "port": "23",
            "protocol": "TCP",
            "source": "0.0.0.0/0",
            "destination": "0.0.0.0/0",
            "action": "DROP",
            "enabled": true,
            "priority": 50,
            "hit_count": 1245,
            "last_hit": "2025-03-15T08:30:00Z"
          },
          {
            "id": 4,
            "name": "MQTT Broker Access",
            "description": "Allow MQTT connections",
            "chain": "INPUT",
            "port": "1883",
            "protocol": "TCP",
            "source": "192.168.1.0/24",
            "destination": "0.0.0.0/0",
            "action": "ACCEPT",
            "enabled": true,
            "priority": 150,
            "hit_count": 890,
            "last_hit": "2025-03-15T09:30:00Z"
          },
          {
            "id": 5,
            "name": "Default Deny",
            "description": "Default deny all incoming traffic",
            "chain": "INPUT",
            "source": "0.0.0.0/0",
            "destination": "0.0.0.0/0",
            "action": "DROP",
            "enabled": true,
            "priority": 1000,
            "hit_count": 1245,
            "last_hit": "2025-03-15T10:20:00Z"
          }
        ],
        "chains": {
          "INPUT": 12,
          "OUTPUT": 2,
          "FORWARD": 1
        },
        "actions": {
          "ACCEPT": 10,
          "DROP": 4,
          "REJECT": 1
        }
      }

Add Firewall Rule
~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/rules/add

   Add a new firewall rule.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/rules/add HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Allow NTP Time Sync",
        "description": "Allow NTP for time synchronization",
        "chain": "INPUT",
        "port": "123",
        "protocol": "UDP",
        "source": "0.0.0.0/0",
        "destination": "0.0.0.0/0",
        "action": "ACCEPT",
        "enabled": true,
        "priority": 120,
        "interface": "eth0",
        "log_matches": true,
        "log_prefix": "NTP_TRAFFIC"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "added",
        "rule_id": 16,
        "rule": {
          "id": 16,
          "name": "Allow NTP Time Sync",
          "chain": "INPUT",
          "port": "123",
          "protocol": "UDP",
          "action": "ACCEPT",
          "enabled": true,
          "priority": 120
        },
        "message": "Firewall rule added successfully",
        "requires_reload": true
      }

Remove Firewall Rule
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/rules/remove

   Remove a firewall rule.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/rules/remove HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "rule_id": 3,
        "force": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "removed",
        "rule_id": 3,
        "rule_name": "Block Telnet",
        "message": "Firewall rule removed successfully",
        "requires_reload": true
      }

Update Firewall Rule
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/rules/update

   Update an existing firewall rule.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/rules/update HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "rule_id": 4,
        "name": "MQTT Broker Secure Access",
        "description": "Allow MQTT from internal network only",
        "source": "192.168.0.0/16",
        "enabled": true,
        "priority": 110,
        "log_matches": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "rule_id": 4,
        "rule_name": "MQTT Broker Secure Access",
        "message": "Firewall rule updated successfully",
        "changes": {
          "source": {"old": "192.168.1.0/24", "new": "192.168.0.0/16"},
          "priority": {"old": 150, "new": 110}
        },
        "requires_reload": true
      }

Manage IP Whitelist
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/whitelist/manage

   Add or remove IP addresses from whitelist.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/whitelist/manage HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "action": "add",  // or "remove"
        "ip_address": "10.0.1.0/24",
        "description": "Engineering department network",
        "expires": "2025-06-15T00:00:00Z"  // Optional
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "action": "added",
        "ip_address": "10.0.1.0/24",
        "whitelist": ["192.168.0.0/24", "10.0.0.5", "10.0.1.0/24"],
        "message": "IP address added to whitelist",
        "requires_reload": true
      }

Manage IP Blacklist
~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/blacklist/manage

   Add or remove IP addresses from blacklist.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/blacklist/manage HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "action": "add",
        "ip_address": "45.33.32.156",
        "reason": "Malicious activity detected",
        "duration_hours": 24
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "action": "added",
        "ip_address": "45.33.32.156",
        "blacklist": ["203.0.113.5", "45.33.32.156"],
        "message": "IP address added to blacklist",
        "expires": "2025-03-16T10:30:00Z",
        "requires_reload": true
      }

Manage Blocked Ports
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/ports/manage

   Block or unblock network ports.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/ports/manage HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "action": "block",  // or "unblock"
        "port": 8080,
        "protocol": "TCP",
        "reason": "Unauthorized web server"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "action": "blocked",
        "port": 8080,
        "protocol": "TCP",
        "blocked_ports": [23, 21, 8000, 8080],
        "message": "Port blocked successfully",
        "requires_reload": true
      }

Update Firewall Settings
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/firewall/settings

   Update firewall configuration settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/firewall/settings HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": true,
        "default_policy": "DROP",
        "rate_limiting": {
          "enabled": true,
          "requests_per_second": 50,
          "burst_size": 100
        },
        "dos_protection": true,
        "syn_flood_protection": true,
        "syn_flood_rate": 25,
        "port_scan_detection": true,
        "port_scan_threshold": 10,
        "vpn_enabled": true,
        "vpn_port": 1194,
        "geo_blocking": false,
        "log_level": "info"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "message": "Firewall settings updated successfully",
        "restart_required": true,
        "changes": {
          "rate_limiting.requests_per_second": {"old": 20, "new": 50},
          "default_policy": {"old": "ACCEPT", "new": "DROP"},
          "vpn_enabled": {"old": false, "new": true}
        }
      }

Get Firewall Logs
~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/firewall/logs

   Get firewall logs and blocked connections.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **hours** (optional): Time range in hours (default: 24)
   * **action** (optional): Filter by action (DROP, REJECT, ACCEPT)
   * **limit** (optional): Number of entries (default: 100)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "logs": [
          {
            "timestamp": "2025-03-15T10:25:30Z",
            "action": "DROP",
            "protocol": "TCP",
            "source_ip": "203.0.113.5",
            "source_port": 54321,
            "destination_ip": "192.168.1.100",
            "destination_port": 23,
            "interface": "eth0",
            "rule_name": "Block Telnet",
            "bytes": 60,
            "reason": "Port 23 blocked"
          },
          {
            "timestamp": "2025-03-15T10:20:00Z",
            "action": "ACCEPT",
            "protocol": "TCP",
            "source_ip": "192.168.1.50",
            "source_port": 54322,
            "destination_ip": "192.168.1.100",
            "destination_port": 22,
            "interface": "eth0",
            "rule_name": "SSH Management Access",
            "bytes": 120
          }
        ],
        "total": 1245,
        "filtered": 100,
        "statistics": {
          "dropped": 1245,
          "accepted": 89000,
          "rejected": 0,
          "top_source_ips": [
            {"ip": "203.0.113.5", "count": 450},
            {"ip": "45.33.32.156", "count": 320}
          ],
          "top_blocked_ports": [
            {"port": 23, "count": 450},
            {"port": 21, "count": 320},
            {"port": 8000, "count": 200}
          ]
        }
      }

Certificate Management
----------------------

Get Certificates Status
~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/certificates/status

   Get status of all certificates.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "https_certificate": {
          "present": true,
          "type": "letsencrypt",
          "subject": "CN=gateway.example.com",
          "issuer": "CN=Let's Encrypt R3",
          "valid_from": "2025-01-01T00:00:00Z",
          "valid_to": "2028-01-01T00:00:00Z",
          "days_remaining": 1020,
          "status": "valid",
          "algorithm": "RSA-SHA256",
          "key_size": 2048,
          "san": ["gateway.example.com", "www.gateway.example.com"]
        },
        "mqtt_certificate": {
          "present": true,
          "type": "self_signed",
          "subject": "CN=MQTT Broker",
          "issuer": "CN=MQTT Broker",
          "valid_from": "2024-12-01T00:00:00Z",
          "valid_to": "2025-03-29T00:00:00Z",
          "days_remaining": 14,
          "status": "expiring",
          "algorithm": "RSA-SHA256",
          "key_size": 2048
        },
        "client_certificates": [
          {
            "name": "Mobile App",
            "subject": "CN=Mobile App Client",
            "valid_to": "2025-06-01T00:00:00Z",
            "days_remaining": 78,
            "status": "valid"
          }
        ],
        "settings": {
          "auto_renew": false,
          "renewal_days_before_expiry": 30,
          "acme_email": "admin@example.com",
          "acme_server": "https://acme-v02.api.letsencrypt.org/directory",
          "last_renewal": null,
          "next_renewal_check": "2025-03-16T02:00:00Z"
        }
      }

Upload Certificate
~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/certificates/upload

   Upload SSL/TLS certificate.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/certificates/upload HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="certificate"; filename="server.crt"
      Content-Type: application/x-x509-ca-cert
      
      --form-data; name="private_key"; filename="server.key"
      Content-Type: application/x-pem-file
      
      --form-data; name="type"
      
      https
      
      --form-data; name="chain"; filename="chain.pem"
      Content-Type: application/x-pem-file

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "uploaded",
        "certificate_info": {
          "type": "https",
          "subject": "CN=gateway.example.com",
          "issuer": "CN=Let's Encrypt R3",
          "valid_from": "2025-03-15T00:00:00Z",
          "valid_to": "2025-06-13T00:00:00Z",
          "algorithm": "RSA-SHA256",
          "key_size": 2048,
          "san": ["gateway.example.com"],
          "serial_number": "0123456789ABCDEF"
        },
        "validation": {
          "certificate_valid": true,
          "private_key_match": true,
          "chain_valid": true,
          "not_expired": true,
          "key_strength": "adequate"
        },
        "message": "Certificate uploaded successfully",
        "services_restarted": ["nginx", "mqtt"]
      }

Generate Self-Signed Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/certificates/generate-self-signed

   Generate self-signed certificate.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/certificates/generate-self-signed HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "common_name": "gateway.local",
        "organization": "Example Corp",
        "organizational_unit": "IoT Department",
        "country": "US",
        "state": "California",
        "locality": "San Francisco",
        "valid_days": 365,
        "key_size": 2048,
        "algorithm": "RSA",
        "san": ["gateway.local", "192.168.1.100"],
        "password": "certpass123"  // Optional: For encrypted private key
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "generated",
        "certificate": {
          "certificate": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
          "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
          "csr": "-----BEGIN CERTIFICATE REQUEST-----\n...\n-----END CERTIFICATE REQUEST-----",
          "subject": "CN=gateway.local, O=Example Corp, OU=IoT Department, C=US, ST=California, L=San Francisco",
          "valid_from": "2025-03-15T10:30:00Z",
          "valid_to": "2026-03-15T10:30:00Z",
          "fingerprint": "SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890",
          "key_size": 2048,
          "algorithm": "RSA"
        },
        "download_urls": {
          "certificate": "/cloud-integration/security/certificates/download/cert",
          "private_key": "/cloud-integration/security/certificates/download/key",
          "bundle": "/cloud-integration/security/certificates/download/bundle"
        },
        "message": "Self-signed certificate generated successfully"
      }

Update Certificate Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/certificates/settings

   Update certificate management settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/certificates/settings HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "auto_renew": true,
        "renewal_days_before_expiry": 30,
        "acme_email": "admin@example.com",
        "acme_server": "https://acme-v02.api.letsencrypt.org/directory",
        "staging_mode": false,
        "webroot_path": "/var/www/html",
        "notify_before_expiry": true,
        "notify_days_before": [7, 3, 1],
        "key_rotation_days": 90
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "updated",
        "message": "Certificate settings updated successfully",
        "next_renewal_check": "2025-03-16T02:00:00Z",
        "changes": {
          "auto_renew": {"old": false, "new": true},
          "acme_email": {"old": null, "new": "admin@example.com"}
        }
      }

Validate Certificate
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/certificates/validate

   Validate certificate and private key.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/certificates/validate HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "certificate_type": "https",
        "certificate_data": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
        "private_key_data": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
        "chain_data": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "valid": true,
        "validation_result": {
          "signature_valid": true,
          "not_expired": true,
          "not_before_valid": true,
          "key_match": true,
          "chain_valid": true,
          "key_strength": "adequate",
          "subject_valid": true,
          "san_present": true
        },
        "certificate_info": {
          "subject": "CN=gateway.example.com",
          "issuer": "CN=Let's Encrypt R3",
          "valid_from": "2025-03-15T00:00:00Z",
          "valid_to": "2025-06-13T00:00:00Z",
          "algorithm": "RSA-SHA256",
          "key_size": 2048
        },
        "recommendations": ["Certificate expires in 90 days"]
      }

Renew Certificate
~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/certificates/renew

   Renew an expiring certificate.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/certificates/renew HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "certificate_type": "https",
        "force": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "renewing",
        "certificate_type": "https",
        "renewal_id": "RENEW-001",
        "estimated_time": 60,
        "steps": [
          "Generating CSR",
          "Requesting certificate from CA",
          "Validating domain ownership",
          "Downloading certificate",
          "Installing certificate"
        ],
        "message": "Certificate renewal started"
      }

Security Audit Log
------------------

Get Audit Log
~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/audit/logs

   Get security audit logs.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **hours** (optional): Time range in hours (default: 24, max: 720)
   * **event_type** (optional): Filter by event type
   * **username** (optional): Filter by username
   * **ip_address** (optional): Filter by IP address
   * **status** (optional): Filter by status (success, failed)
   * **resource** (optional): Filter by resource
   * **limit** (optional): Number of entries (default: 100, max: 1000)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "logs": [
          {
            "id": "audit-001",
            "timestamp": "2025-03-15T10:15:00Z",
            "user": "admin",
            "event": "Login",
            "resource": "/api/auth/login",
            "ip_address": "192.168.1.100",
            "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
            "status": "success",
            "details": {
              "method": "POST",
              "parameters": {},
              "session_id": "session_abc123"
            },
            "severity": "info"
          },
          {
            "id": "audit-002",
            "timestamp": "2025-03-15T09:45:00Z",
            "user": "unknown",
            "event": "Failed Login",
            "resource": "/api/auth/login",
            "ip_address": "203.0.113.5",
            "user_agent": "curl/7.68.0",
            "status": "failed",
            "details": {
              "method": "POST",
              "parameters": {"username": "admin"},
              "attempt_count": 5,
              "lockout_triggered": true
            },
            "severity": "warning"
          },
          {
            "id": "audit-003",
            "timestamp": "2025-03-15T08:30:00Z",
            "user": "admin",
            "event": "User Modified",
            "resource": "/api/users/ops01",
            "ip_address": "192.168.1.100",
            "status": "success",
            "details": {
              "method": "PUT",
              "parameters": {"role": "engineer"},
              "changes": {"role": "operator -> engineer"}
            },
            "severity": "info"
          },
          {
            "id": "audit-004",
            "timestamp": "2025-03-15T08:15:00Z",
            "user": "admin",
            "event": "Firewall Rule Added",
            "resource": "/api/firewall/rules",
            "ip_address": "192.168.1.100",
            "status": "success",
            "details": {
              "rule_id": 16,
              "rule_name": "Allow NTP",
              "parameters": {"port": 123, "protocol": "UDP"}
            },
            "severity": "info"
          },
          {
            "id": "audit-005",
            "timestamp": "2025-03-15T07:30:00Z",
            "user": "ops01",
            "event": "SSH Session Started",
            "resource": "ssh",
            "ip_address": "10.0.0.15",
            "status": "success",
            "details": {
              "session_id": "ssh-002",
              "client_version": "OpenSSH_8.9p1"
            },
            "severity": "info"
          }
        ],
        "total": 125,
        "filtered": 5,
        "pagination": {
          "page": 1,
          "limit": 5,
          "pages": 25
        },
        "stats": {
          "successful_events": 120,
          "failed_events": 5,
          "unique_users": 4,
          "unique_ips": 8,
          "event_types": {
            "login": 15,
            "user_modified": 2,
            "firewall_modified": 3,
            "ssh_session": 8,
            "api_key_created": 1
          }
        }
      }

Export Audit Log
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/audit/export

   Export audit logs to file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **format** (optional): Export format (csv, json, pdf) - default: csv
   * **start_date** (optional): Start date (YYYY-MM-DD)
   * **end_date** (optional): End date (YYYY-MM-DD)
   * **event_type** (optional): Filter by event type
   * **username** (optional): Filter by username

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="audit_log_20250315.csv"
      
      timestamp,user,event,resource,ip_address,status,details
      2025-03-15T10:15:00Z,admin,Login,/api/auth/login,192.168.1.100,success,Successful login from workstation
      2025-03-15T09:45:00Z,unknown,Failed Login,/api/auth/login,203.0.113.5,failed,Invalid credentials (5 attempts)
      ...

Get Audit Statistics
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/audit/stats

   Get audit log statistics.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **time_range** (optional): Time range (24h, 7d, 30d, custom)
   * **start_date** (optional): Custom start date
   * **end_date** (optional): Custom end date

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "time_range": "24h",
        "last_24_hours": {
          "total_events": 45,
          "successful": 42,
          "failed": 3,
          "login_attempts": 15,
          "ssh_sessions": 8,
          "user_changes": 2,
          "firewall_changes": 1,
          "api_calls": 19
        },
        "top_users": [
          {"username": "admin", "events": 25, "success_rate": 100},
          {"username": "ops01", "events": 10, "success_rate": 100},
          {"username": "unknown", "events": 5, "success_rate": 0}
        ],
        "top_ips": [
          {"ip": "192.168.1.100", "events": 30, "success_rate": 100},
          {"ip": "10.0.0.15", "events": 8, "success_rate": 100},
          {"ip": "203.0.113.5", "events": 3, "success_rate": 0}
        ],
        "event_timeline": [
          {"hour": "00:00", "events": 2},
          {"hour": "01:00", "events": 1},
          {"hour": "02:00", "events": 0},
          // ... more hours
          {"hour": "10:00", "events": 15},
          {"hour": "11:00", "events": 8}
        ],
        "security_alerts": [
          {
            "timestamp": "2025-03-15T09:45:00Z",
            "type": "brute_force",
            "ip": "203.0.113.5",
            "severity": "high",
            "description": "Multiple failed login attempts from single IP (5 attempts in 5 minutes)",
            "action_taken": "IP temporarily blocked"
          },
          {
            "timestamp": "2025-03-15T08:30:00Z",
            "type": "unusual_location",
            "user": "admin",
            "ip": "45.33.32.156",
            "severity": "medium",
            "description": "Login from new location (previously only from 192.168.1.100)"
          }
        ],
        "compliance": {
          "retention_days": 90,
          "encryption_enabled": true,
          "tamper_protection": true,
          "export_capability": true
        }
      }

Clear Audit Log
~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/audit/clear

   Clear audit logs (admin only).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/audit/clear HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "older_than_days": 30,
        "confirm": true,
        "archive_before_delete": true,
        "reason": "Routine cleanup"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "cleared",
        "deleted_count": 1250,
        "archived_count": 1250,
        "retained_count": 45,
        "archive_file": "/var/log/audit/archive_20250315.tar.gz",
        "message": "Audit logs cleared successfully"
      }

Security Operations
-------------------

Run Security Scan
~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/scan

   Run security vulnerability scan.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/scan HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "scan_type": "full",  // "quick", "full", "compliance", "vulnerability"
        "components": ["authentication", "firewall", "certificates", "ssh", "network"],
        "depth": "deep",  // "quick", "standard", "deep"
        "schedule": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "running",
        "scan_id": "scan-20250315-001",
        "scan_type": "full",
        "started": "2025-03-15T10:30:00Z",
        "estimated_completion": "2025-03-15T10:35:00Z",
        "components": ["authentication", "firewall", "certificates", "ssh", "network"],
        "progress_url": "/cloud-integration/security/scan/progress/scan-20250315-001"
      }

Get Scan Results
~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/scan/results

   Get security scan results.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **scan_id** (optional): Specific scan ID
   * **latest** (optional): Get latest scan results (default: true)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-20250315-001",
        "scan_type": "full",
        "started": "2025-03-15T10:30:00Z",
        "completed": "2025-03-15T10:32:00Z",
        "duration": "00:02:00",
        "results": {
          "authentication": {
            "status": "pass",
            "score": 95,
            "issues": 0,
            "checks": [
              {"check": "password_policy", "status": "pass", "details": "Password policy compliant", "severity": "low"},
              {"check": "session_security", "status": "pass", "details": "Session timeout properly configured", "severity": "low"},
              {"check": "two_factor", "status": "warning", "details": "2FA not enforced for all users", "severity": "medium"}
            ]
          },
          "firewall": {
            "status": "warning",
            "score": 80,
            "issues": 2,
            "checks": [
              {"check": "rules_active", "status": "pass", "details": "Firewall rules active and loaded", "severity": "low"},
              {"check": "default_deny", "status": "warning", "details": "Default policy not set to deny", "severity": "medium"},
              {"check": "unused_rules", "status": "warning", "details": "3 unused firewall rules detected", "severity": "low"}
            ]
          },
          "certificates": {
            "status": "warning",
            "score": 70,
            "issues": 1,
            "checks": [
              {"check": "https_certificate", "status": "pass", "details": "HTTPS certificate valid", "severity": "low"},
              {"check": "mqtt_certificate", "status": "warning", "details": "MQTT certificate expires in 14 days", "severity": "medium"}
            ]
          },
          "ssh": {
            "status": "pass",
            "score": 90,
            "issues": 1,
            "checks": [
              {"check": "password_auth", "status": "pass", "details": "Password authentication disabled", "severity": "low"},
              {"check": "root_login", "status": "pass", "details": "Root login disabled", "severity": "low"},
              {"check": "key_strength", "status": "warning", "details": "One weak SSH key detected (1024-bit)", "severity": "medium"}
            ]
          },
          "network": {
            "status": "pass",
            "score": 85,
            "issues": 0,
            "checks": [
              {"check": "open_ports", "status": "pass", "details": "Only necessary ports open", "severity": "low"},
              {"check": "service_versions", "status": "pass", "details": "Services up to date", "severity": "low"}
            ]
          }
        },
        "total_issues": 4,
        "security_score": 84,
        "recommendations": [
          "Enable 2FA for all users",
          "Set firewall default policy to DROP",
          "Renew MQTT certificate",
          "Replace weak SSH keys"
        ],
        "risk_level": "medium",
        "next_scan_scheduled": "2025-03-22T02:00:00Z"
      }

Get Scan Progress
~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/scan/progress/{scan_id}

   Get progress of ongoing security scan.

   **Path Parameters**:

   * **scan_id** (string): Scan identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-20250315-001",
        "status": "running",
        "progress": 65,
        "current_component": "certificates",
        "current_check": "validity_check",
        "started": "2025-03-15T10:30:00Z",
        "elapsed": "00:01:18",
        "estimated_remaining": "00:00:42",
        "components_completed": ["authentication", "firewall"],
        "components_remaining": ["ssh", "network"],
        "issues_found": 2
      }

Lock System
~~~~~~~~~~~

.. http:post:: /cloud-integration/security/lock

   Lock system access (emergency).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/lock HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "reason": "Security breach detected",
        "duration_minutes": 30,
        "lock_users": true,
        "lock_api": true,
        "lock_ssh": true,
        "notify_users": true,
        "emergency_contact": "security@example.com"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "locked",
        "lock_id": "LOCK-001",
        "locked_until": "2025-03-15T11:00:00Z",
        "components_locked": ["users", "api", "ssh"],
        "active_sessions_terminated": 3,
        "message": "System locked. All users will be logged out.",
        "emergency_access_code": "EMG-123456"  // For emergency unlock
      }

Unlock System
~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/unlock

   Unlock system access.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/unlock HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "admin_pin": "7392",
        "emergency_code": "EMG-123456"  // Optional, if available
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "unlocked",
        "unlocked_by": "admin",
        "unlocked_at": "2025-03-15T10:35:00Z",
        "lock_duration": "00:05:00",
        "message": "System unlocked successfully",
        "post_unlock_scan_scheduled": true
      }

Reset Security Settings
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/reset

   Reset security settings to defaults.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/reset HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "confirm": true,
        "preserve_users": true,
        "preserve_api_keys": false,
        "preserve_certificates": true,
        "backup_before_reset": true,
        "reason": "Misconfiguration recovery"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "reset",
        "backup_file": "/backup/security_backup_20250315.tar.gz",
        "components_reset": ["authentication", "firewall", "ssh"],
        "components_preserved": ["users", "certificates"],
        "message": "Security settings reset to defaults",
        "restart_required": true,
        "default_admin_password": "Admin123!"  // If admin password reset
      }

Bulk Operations
---------------

Save All Security Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/settings/save-all

   Save all security settings at once.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/settings/save-all HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "authentication": {
          "password_policy": {
            "min_length": 12,
            "expiry_days": 60
          },
          "two_factor": {
            "mode": "required"
          }
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
        },
        "audit": {
          "retention_days": 90
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "saved",
        "timestamp": "2025-03-15T10:30:00Z",
        "changes": {
          "authentication": {
            "password_policy.min_length": {"old": 8, "new": 12},
            "two_factor.mode": {"old": "optional", "new": "required"}
          },
          "firewall": {
            "default_policy": {"old": "ACCEPT", "new": "DROP"}
          },
          "ssh": {
            "port": {"old": 22, "new": 2222}
          },
          "certificates": {
            "auto_renew": {"old": false, "new": true}
          }
        },
        "restart_required": true,
        "affected_services": ["nginx", "ssh", "firewall"]
      }

Backup Security Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-integration/security/backup

   Backup security configuration.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **components** (optional): Comma-separated components to backup
   * **encrypt** (optional): Encrypt backup (default: true)
   * **password** (optional): Encryption password

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "backup_id": "backup-20250315-001",
        "filename": "security_backup_20250315_103000.tar.gz",
        "download_url": "/cloud-integration/security/backup/download/backup-20250315-001",
        "size": "1.5 MB",
        "encrypted": true,
        "contains": [
          "users.json",
          "roles.json", 
          "api_keys.json",
          "firewall_rules.json",
          "certificates/",
          "ssh_keys/",
          "audit_logs.csv"
        ],
        "checksum": "sha256:abcd1234...",
        "created": "2025-03-15T10:30:00Z",
        "expires": "2025-04-15T10:30:00Z"
      }

Restore Security Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-integration/security/restore

   Restore security configuration from backup.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /cloud-integration/security/restore HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="security_backup.tar.gz"
      Content-Type: application/gzip
      
      --form-data; name="password"
      
      backup_password
      
      --form-data; name="components"
      
      ["users", "firewall_rules"]

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "restored",
        "backup_id": "backup-20250315-001",
        "restored_components": ["users", "firewall_rules"],
        "skipped_components": ["certificates"],
        "message": "Security configuration restored successfully",
        "users_restored": 8,
        "rules_restored": 15,
        "restart_required": true,
        "validation_report": "/cloud-integration/security/restore/report/backup-20250315-001"
      }

Real-time Monitoring (WebSocket)
--------------------------------

### Security Events Stream

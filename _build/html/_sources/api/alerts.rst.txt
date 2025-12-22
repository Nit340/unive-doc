Alert Messages API
=====================

This document describes the alert messages management page and its related API endpoints for managing alert severity classes, message templates, and configuration.

Page Route (Frontend)
---------------------

.. http:get:: /alert-messages

   **Description**: Renders the complete alert messages management page with all alert classes, messages, and configuration embedded in the HTML.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!DOCTYPE html>
      <html>
      <head>
        <title>Alert Messages - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-alert-classes='[...]' data-alert-messages='[...]' data-configuration='{...}'>
          <!-- Alert messages page with:
               - Alert classes table (severity, colors, icons)
               - Alert messages table (templates, variables, channels)
               - Preview functionality
               - Import/export configuration
               - Live preview interface
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - All alert classes, messages, and configuration data are embedded in the page
   - JavaScript reads this data and renders the alert management interface
   - No separate API call needed on initial page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all alert operations triggered from the page.

Alert Classes Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/alert-messages/alert-classes

   **Description**: Add new alert class (when user clicks "Add Class").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/alert-classes HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "Emergency",
        "severity": 5,
        "color": "#DC2626",
        "icon": "fa-fire",
        "description": "Emergency situation requiring immediate action"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "alert_class": {
          "id": 6,
          "name": "Emergency",
          "severity": 5,
          "color": "#DC2626",
          "icon": "fa-fire",
          "description": "Emergency situation requiring immediate action"
        }
      }

.. http:put:: /api/alert-messages/alert-classes/{class_id}

   **Description**: Edit alert class (when user clicks "Edit" on an alert class).
   
   **Path Parameters**:
   
   * **class_id** (integer): Alert class identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "alert_class": {
          "id": 6,
          "name": "Emergency",
          "severity": 5,
          "color": "#B91C1C",
          "icon": "fa-fire-flame-curved"
        }
      }

.. http:delete:: /api/alert-messages/alert-classes/{class_id}

   **Description**: Remove alert class (when user clicks "Delete" on an alert class).
   
   **Path Parameters**:
   
   * **class_id** (integer): Alert class identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "class_id": 6
      }

Alert Messages Operations
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/alert-messages/alert-messages

   **Description**: Add new alert message (when user clicks "Add Message").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/alert-messages HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "High Pressure Alert",
        "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
        "class_id": 3,
        "channels": ["mqtt", "email", "dashboard"]
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "alert_message": {
          "id": 6,
          "name": "High Pressure Alert",
          "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
          "class_id": 3,
          "channels": ["mqtt", "email", "dashboard"]
        }
      }

.. http:put:: /api/alert-messages/alert-messages/{message_id}

   **Description**: Edit alert message (when user clicks "Edit" on an alert message).
   
   **Path Parameters**:
   
   * **message_id** (integer): Alert message identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "alert_message": {
          "id": 6,
          "name": "High Pressure Warning",
          "message": "⚠️ WARNING: {device} pressure is {value} bar (Limit: {limit} bar)",
          "class_id": 3,
          "channels": ["mqtt", "email", "dashboard", "sms"]
        }
      }

.. http:delete:: /api/alert-messages/alert-messages/{message_id}

   **Description**: Remove alert message (when user clicks "Delete" on an alert message).
   
   **Path Parameters**:
   
   * **message_id** (integer): Alert message identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message_id": 6
      }

Preview Operations
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/alert-messages/preview

   **Description**: Preview an alert message with sample data (when user tests a message template).
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/preview HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "message": "🔥 CRITICAL: {device} temperature is {value}°C at {timestamp}",
        "data": {
          "device": "Crane-01",
          "value": "85.5",
          "timestamp": "2024-01-15T10:30:00Z"
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "original": "🔥 CRITICAL: {device} temperature is {value}°C at {timestamp}",
        "rendered": "🔥 CRITICAL: Crane-01 temperature is 85.5°C at 2024-01-15T10:30:00Z",
        "missing_variables": ["threshold", "operator"],
        "extracted_variables": ["device", "value", "timestamp"]
      }

.. http:get:: /api/alert-messages/alert-messages/{message_id}/preview

   **Description**: Preview a specific alert message with default sample data.
   
   **Path Parameters**:
   
   * **message_id** (integer): Alert message identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": 1,
        "name": "Critical Temperature",
        "original_message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
        "rendered_message": "🔥 CRITICAL: Device-001 temperature is 85.5°C (Threshold: 80°C)",
        "sample_data": {
          "device": "Device-001",
          "value": "85.5",
          "threshold": "80"
        },
        "class_info": {
          "name": "Critical",
          "severity": 5,
          "color": "#DC2626",
          "icon": "fa-triangle-exclamation"
        },
        "channels": ["mqtt", "email", "sms", "dashboard"]
      }

Configuration Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/alert-messages/configuration/export

   **Description**: Export all alert configuration (when user clicks "Export Config").
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "alert_classes": [
          {
            "id": 1,
            "name": "Critical",
            "severity": 5,
            "color": "#DC2626",
            "icon": "fa-triangle-exclamation",
            "description": "Immediate action required"
          }
        ],
        "alert_messages": [
          {
            "id": 1,
            "name": "Critical Temperature",
            "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
            "class_id": 1,
            "channels": ["mqtt", "email", "sms", "dashboard"]
          }
        ],
        "export_timestamp": "2024-01-15T10:30:00Z",
        "version": "1.0.0"
      }

.. http:post:: /api/alert-messages/configuration/import

   **Description**: Import alert configuration from file (when user imports configuration).
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: multipart/form-data
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "imported_classes": 5,
        "imported_messages": 5,
        "skipped_classes": 0,
        "skipped_messages": 0,
        "errors": []
      }

.. http:post:: /api/alert-messages/configuration/reset

   **Description**: Reset all alert configuration to defaults.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/configuration/reset HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "confirm": true
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "reset": true,
        "default_classes_count": 5,
        "default_messages_count": 5
      }

Bulk Operations
~~~~~~~~~~~~~~~

.. http:post:: /api/alert-messages/alert-messages/bulk-delete

   **Description**: Delete multiple alert messages in a single request.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/alert-messages/bulk-delete HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "ids": [6, 7, 8]
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "deleted_count": 3,
        "failed_ids": [],
        "errors": []
      }

Live Preview Operations
~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/alert-messages/live-preview

   **Description**: Get live preview of all messages with sample data (when user clicks "Live Preview").
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      [
        {
          "id": 1,
          "name": "Critical Temperature",
          "original_message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
          "rendered_message": "🔥 CRITICAL: Crane-01 temperature is 85.5°C (Threshold: 80°C)",
          "class_info": {
            "name": "Critical",
            "severity": 5,
            "color": "#DC2626",
            "icon": "fa-triangle-exclamation"
          },
          "channels": ["mqtt", "email", "sms", "dashboard"],
          "sample_data": {
            "device": "Crane-01",
            "value": "85.5",
            "threshold": "80",
            "unit": "°C",
            "timestamp": "2024-01-15T10:30:00Z"
          }
        }
      ]

.. http:post:: /api/alert-messages/live-preview/generate

   **Description**: Generate custom live preview with provided data.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/live-preview/generate HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "message_id": 1,
        "custom_data": {
          "device": "Motor-A",
          "value": "120",
          "threshold": "100",
          "unit": "°F"
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "id": 1,
        "name": "Critical Temperature",
        "original_message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
        "rendered_message": "🔥 CRITICAL: Motor-A temperature is 120°F (Threshold: 100°C)",
        "missing_variables": [],
        "variables_used": ["device", "value", "threshold", "unit"],
        "custom_data_applied": true
      }

Utilities Operations
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/alert-messages/variables

   **Description**: Get available variables for message templates.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      [
        {
          "name": "device",
          "description": "Device name",
          "type": "string",
          "example": "Crane-01"
        },
        {
          "name": "value",
          "description": "Current value",
          "type": "string",
          "example": "85.5"
        },
        {
          "name": "timestamp",
          "description": "Time of alert",
          "type": "datetime",
          "example": "2024-01-15T10:30:00Z"
        },
        {
          "name": "threshold",
          "description": "Trigger threshold",
          "type": "number",
          "example": "80"
        }
      ]

.. http:get:: /api/alert-messages/channels

   **Description**: Get available delivery channels.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      [
        {
          "name": "mqtt",
          "description": "MQTT protocol",
          "enabled": true,
          "config_required": true
        },
        {
          "name": "email",
          "description": "Email notification",
          "enabled": true,
          "config_required": true
        },
        {
          "name": "sms",
          "description": "SMS notification",
          "enabled": false,
          "config_required": true
        },
        {
          "name": "dashboard",
          "description": "Dashboard display",
          "enabled": true,
          "config_required": false
        },
        {
          "name": "webhook",
          "description": "Webhook endpoint",
          "enabled": true,
          "config_required": true
        }
      ]

.. http:get:: /api/alert-messages/alert-colors

   **Description**: Get available color options for alert classes.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      [
        {
          "name": "Red",
          "value": "#DC2626"
        },
        {
          "name": "Orange",
          "value": "#F97316"
        },
        {
          "name": "Amber",
          "value": "#F59E0B"
        },
        {
          "name": "Blue",
          "value": "#3B82F6"
        },
        {
          "name": "Green",
          "value": "#10B981"
        },
        {
          "name": "Purple",
          "value": "#8B5CF6"
        },
        {
          "name": "Pink",
          "value": "#EC4899"
        },
        {
          "name": "Gray",
          "value": "#6B7280"
        }
      ]

.. http:get:: /api/alert-messages/alert-icons

   **Description**: Get available icon options for alert classes.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      [
        {
          "name": "Critical",
          "value": "fa-triangle-exclamation"
        },
        {
          "name": "High",
          "value": "fa-circle-exclamation"
        },
        {
          "name": "Warning",
          "value": "fa-exclamation"
        },
        {
          "name": "Info",
          "value": "fa-circle-info"
        },
        {
          "name": "Low",
          "value": "fa-circle-check"
        },
        {
          "name": "Fire/Temperature",
          "value": "fa-fire"
        },
        {
          "name": "Power/Device",
          "value": "fa-plug-circle-xmark"
        },
        {
          "name": "Vibration",
          "value": "fa-wave-square"
        }
      ]

.. http:post:: /api/alert-messages/validate/message

   **Description**: Validate a message template.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/alert-messages/validate/message HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "message": "🔥 {device} is at {value}°C"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "valid": true,
        "extracted_variables": ["device", "value"],
        "error": null
      }

Save Configuration
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/alert-messages/configuration/save

   **Description**: Save current alert configuration (persist changes).
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "saved": true,
        "timestamp": "2024-01-15T10:30:00Z",
        "classes_count": 5,
        "messages_count": 5
      }

.. http:get:: /api/alert-messages/configuration/status

   **Description**: Get configuration status and statistics.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "active",
        "last_saved": "2024-01-15T10:30:00Z",
        "classes_count": 5,
        "messages_count": 5,
        "channels_enabled": 3,
        "has_unsaved_changes": false,
        "memory_usage": "2.5MB"
      }

Route Summary
-------------

.. list-table:: Alert Messages Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/alert-messages`` - Main alert messages page
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/alert-classes`` - Add alert class
     - Yes
   * - API
     - PUT
     - ``/api/alert-messages/alert-classes/{id}`` - Edit alert class
     - Yes
   * - API
     - DELETE
     - ``/api/alert-messages/alert-classes/{id}`` - Delete alert class
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/alert-messages`` - Add alert message
     - Yes
   * - API
     - PUT
     - ``/api/alert-messages/alert-messages/{id}`` - Edit alert message
     - Yes
   * - API
     - DELETE
     - ``/api/alert-messages/alert-messages/{id}`` - Delete alert message
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/preview`` - Preview message
     - Yes
   * - API
     - GET
     - ``/api/alert-messages/configuration/export`` - Export config
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/configuration/import`` - Import config
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/configuration/reset`` - Reset to defaults
     - Yes
   * - API
     - GET
     - ``/api/alert-messages/live-preview`` - Live preview
     - Yes
   * - API
     - GET
     - ``/api/alert-messages/variables`` - Get variables
     - Yes
   * - API
     - GET
     - ``/api/alert-messages/channels`` - Get channels
     - Yes
   * - API
     - POST
     - ``/api/alert-messages/configuration/save`` - Save config
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /alert-messages``
   - Server renders HTML with embedded alert data
   - Page shows alert classes table, messages table, action buttons

2. **User adds alert class**: 
   - Clicks "Add Class" → opens modal
   - Fills form → ``POST /api/alert-messages/alert-classes``
   - New class appears in table

3. **User adds alert message**:
   - Clicks "Add Message" → ``POST /api/alert-messages/alert-messages``
   - Selects class, enters template, chooses channels
   - New message appears in table

4. **User previews message**:
   - Clicks "Preview" → ``POST /api/alert-messages/preview``
   - Shows rendered message with sample data

5. **User exports configuration**:
   - Clicks "Export Config" → ``GET /api/alert-messages/configuration/export``
   - Downloads JSON configuration file

6. **User uses live preview**:
   - Clicks "Live Preview" → ``GET /api/alert-messages/live-preview``
   - Shows all messages rendered with sample data

7. **User saves configuration**:
   - Changes are auto-saved or user clicks save → ``POST /api/alert-messages/configuration/save``
   - Configuration persists to database

Error Codes
-----------

.. list-table:: Alert Messages Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - ALERT_001
     - Alert class not found
   * - ALERT_002
     - Invalid alert configuration
   * - ALERT_003
     - Alert class already exists
   * - ALERT_004
     - Alert message not found
   * - ALERT_005
     - Invalid message template
   * - ALERT_006
     - Invalid CSV format for import
   * - ALERT_007
     - Cannot delete default resource
   * - ALERT_008
     - Resource in use (cannot delete)
   * - ALERT_009
     - Preview generation failed
   * - ALERT_010
     - Configuration save failed

Key Features
------------

- **Single page** with embedded initial data
- **Real-time preview** of message templates
- **Variable substitution** with sample data
- **Multiple delivery channels** (MQTT, email, SMS, dashboard, webhook)
- **Import/export** of complete configuration
- **Live preview** with custom data
- **Validation** of message templates
- **Color and icon** selection for alert classes
- **Bulk operations** for efficiency
- **Save/load** configuration persistence

Security Notes
--------------

- **Session-based authentication** for web interface
- **API key authentication** for programmatic access
- **CSRF protection** on all POST/PUT/DELETE endpoints
- **Input validation** on all user-provided data
- **Rate limiting** on API endpoints
- **Permission-based access control**

Performance Considerations
--------------------------

- **Server-side rendering** for initial page load
- **Client-side caching** of configuration data
- **Lazy loading** of preview data
- **Compression** of exported configuration files
- **Database indexing** on frequently queried fields
- **Connection pooling** for database access

Browser Compatibility
---------------------

- **Modern browsers** (Chrome 80+, Firefox 75+, Safari 13+, Edge 80+)
- **JavaScript required** for interactive features
- **Responsive design** for mobile/tablet devices
- **Progressive enhancement** for basic functionality
- **Accessibility** (WCAG 2.1 AA compliant)

Testing
-------

- **Unit tests** for API endpoints
- **Integration tests** for user flows
- **UI tests** for interactive components
- **Load testing** for performance validation
- **Security testing** for vulnerability assessment

Monitoring
----------

- **API response times** tracked
- **Error rates** monitored
- **User activity** logged
- **Configuration changes** audited
- **System health** checked regularly

Deployment
----------

- **Docker containers** for easy deployment
- **Kubernetes** for orchestration
- **CI/CD pipeline** for automated testing and deployment
- **Environment-specific configurations**
- **Rollback capabilities** for failed deployments

Support
-------

- **Documentation** available at /docs/alert-messages
- **API reference** at /api-docs/alert-messages
- **Support email**: alerts-support@univagateway.com
- **Issue tracker**: https://github.com/univagateway/issues
- **Community forum**: https://community.univagateway.com
Alert Messages API 
================================

Purpose
-------

The Alert Messages API manages alert severity classes and message templates for the Univa Gateway system. It provides CRUD operations for defining visual properties of alerts (colors, icons, severity levels) and creating message templates with variables and delivery channels.

Base URL
--------

.. code-block:: http

    https://api.univagateway.com/v1

Authentication
--------------

All API requests require authentication via API key.

**Headers:**

.. code-block:: http

    Authorization: Bearer <api_key>
    Content-Type: application/json

Endpoints
---------

1. Alert Classes API
~~~~~~~~~~~~~~~~~~~~

GET /alert-classes
^^^^^^^^^^^^^^^^^^

Retrieve all alert classes with optional filtering.

**Request:**

.. code-block:: http

    GET /alert-classes HTTP/1.1
    Authorization: Bearer <api_key>

**Query Parameters:**

- ``min_severity`` (optional, integer): Filter classes with severity >= this value
- ``max_severity`` (optional, integer): Filter classes with severity <= this value
- ``sort`` (optional, string): Sort field (``name``, ``severity``, ``id``)
- ``order`` (optional, string): Sort order (``asc``, ``desc``)

**Response:**

.. code-block:: json

    [
      {
        "id": 1,
        "name": "Critical",
        "severity": 5,
        "color": "#DC2626",
        "icon": "fa-triangle-exclamation",
        "description": "Immediate action required"
      },
      {
        "id": 2,
        "name": "High",
        "severity": 4,
        "color": "#F97316",
        "icon": "fa-circle-exclamation",
        "description": "Attention required soon"
      }
    ]

**Status Codes:**

- **200 OK**: Successful retrieval
- **401 Unauthorized**: Invalid or missing API key
- **500 Internal Server Error**: Server error

GET /alert-classes/{id}
^^^^^^^^^^^^^^^^^^^^^^^

Retrieve a specific alert class by ID.

**Request:**

.. code-block:: http

    GET /alert-classes/1 HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "id": 1,
      "name": "Critical",
      "severity": 5,
      "color": "#DC2626",
      "icon": "fa-triangle-exclamation",
      "description": "Immediate action required"
    }

**Status Codes:**

- **200 OK**: Successful retrieval
- **404 Not Found**: Alert class not found
- **401 Unauthorized**: Invalid or missing API key

GET /alert-classes/{id}/messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Get all alert messages for a specific alert class.

**Request:**

.. code-block:: http

    GET /alert-classes/1/messages HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    [
      {
        "id": 1,
        "name": "Critical Temperature",
        "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
        "class_id": 1,
        "channels": ["mqtt", "email", "sms", "dashboard"]
      }
    ]

**Status Codes:**

- **200 OK**: Successful retrieval
- **404 Not Found**: Alert class not found
- **401 Unauthorized**: Invalid or missing API key

POST /alert-classes
^^^^^^^^^^^^^^^^^^^

Create a new alert class.

**Request:**

.. code-block:: http

    POST /alert-classes HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "name": "Emergency",
      "severity": 5,
      "color": "#DC2626",
      "icon": "fa-fire",
      "description": "Emergency situation requiring immediate action"
    }

**Required Fields:**

- ``name`` (string, max 50 chars): Name of the alert class
- ``severity`` (integer, 1-5): Severity level
- ``color`` (string, hex format): Hex color code (e.g., #DC2626)
- ``icon`` (string, max 50 chars): FontAwesome icon class

**Optional Fields:**

- ``description`` (string, max 200 chars): Description of the alert class

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "Emergency",
      "severity": 5,
      "color": "#DC2626",
      "icon": "fa-fire",
      "description": "Emergency situation requiring immediate action"
    }

**Status Codes:**

- **201 Created**: Alert class created successfully
- **400 Bad Request**: Missing or invalid fields
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Alert class with same name already exists

PUT /alert-classes/{id}
^^^^^^^^^^^^^^^^^^^^^^^

Update an existing alert class (full update).

**Request:**

.. code-block:: http

    PUT /alert-classes/6 HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "name": "Emergency",
      "severity": 5,
      "color": "#B91C1C",
      "icon": "fa-fire-flame-curved",
      "description": "Emergency situation - highest priority"
    }

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "Emergency",
      "severity": 5,
      "color": "#B91C1C",
      "icon": "fa-fire-flame-curved",
      "description": "Emergency situation - highest priority"
    }

**Status Codes:**

- **200 OK**: Alert class updated successfully
- **400 Bad Request**: Invalid fields
- **404 Not Found**: Alert class not found
- **401 Unauthorized**: Invalid or missing API key

PATCH /alert-classes/{id}
^^^^^^^^^^^^^^^^^^^^^^^^^

Partially update an alert class.

**Request:**

.. code-block:: http

    PATCH /alert-classes/6 HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "color": "#B91C1C",
      "description": "Updated description"
    }

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "Emergency",
      "severity": 5,
      "color": "#B91C1C",
      "icon": "fa-fire-flame-curved",
      "description": "Updated description"
    }

**Status Codes:**

- **200 OK**: Alert class updated successfully
- **400 Bad Request**: Invalid fields
- **404 Not Found**: Alert class not found
- **401 Unauthorized**: Invalid or missing API key

DELETE /alert-classes/{id}
^^^^^^^^^^^^^^^^^^^^^^^^^^

Delete an alert class.

**Request:**

.. code-block:: http

    DELETE /alert-classes/6 HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "deleted": true,
      "id": 6
    }

**Status Codes:**

- **200 OK**: Alert class deleted successfully
- **404 Not Found**: Alert class not found
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Cannot delete - alert messages are using this class

2. Alert Messages API
~~~~~~~~~~~~~~~~~~~~~

GET /alert-messages
^^^^^^^^^^^^^^^^^^^

Retrieve all alert messages with filtering.

**Request:**

.. code-block:: http

    GET /alert-messages HTTP/1.1
    Authorization: Bearer <api_key>

**Query Parameters:**

- ``class_id`` (optional, integer): Filter by alert class ID
- ``channel`` (optional, string): Filter by delivery channel
- ``search`` (optional, string): Search in name and message
- ``sort`` (optional, string): Sort field (``name``, ``class_id``, ``id``)
- ``order`` (optional, string): Sort order (``asc``, ``desc``)
- ``limit`` (optional, integer): Number of records per page (default: 50, max: 100)
- ``page`` (optional, integer): Page number (default: 1)

**Response:**

.. code-block:: json

    [
      {
        "id": 1,
        "name": "Critical Temperature",
        "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
        "class_id": 1,
        "channels": ["mqtt", "email", "sms", "dashboard"]
      },
      {
        "id": 2,
        "name": "Device Offline",
        "message": "📴 {device} is offline at {timestamp}",
        "class_id": 2,
        "channels": ["email", "dashboard"]
      }
    ]

**Status Codes:**

- **200 OK**: Successful retrieval
- **401 Unauthorized**: Invalid or missing API key

GET /alert-messages/{id}
^^^^^^^^^^^^^^^^^^^^^^^^

Retrieve a specific alert message by ID.

**Request:**

.. code-block:: http

    GET /alert-messages/1 HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "id": 1,
      "name": "Critical Temperature",
      "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
      "class_id": 1,
      "channels": ["mqtt", "email", "sms", "dashboard"],
      "variables": ["device", "value", "threshold"]
    }

**Status Codes:**

- **200 OK**: Successful retrieval
- **404 Not Found**: Alert message not found
- **401 Unauthorized**: Invalid or missing API key

POST /alert-messages
^^^^^^^^^^^^^^^^^^^^

Create a new alert message.

**Request:**

.. code-block:: http

    POST /alert-messages HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "name": "High Pressure Alert",
      "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
      "class_id": 3,
      "channels": ["mqtt", "email", "dashboard"]
    }

**Required Fields:**

- ``name`` (string, max 100 chars): Name of the alert message
- ``message`` (string, max 500 chars): Message template with variables
- ``class_id`` (integer): ID of the alert class
- ``channels`` (array): Array of delivery channels (min 1)

**Valid Channels:**

- ``mqtt`` - MQTT protocol
- ``email`` - Email notification
- ``sms`` - SMS notification
- ``dashboard`` - Dashboard display
- ``webhook`` - Webhook endpoint

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "High Pressure Alert",
      "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
      "class_id": 3,
      "channels": ["mqtt", "email", "dashboard"]
    }

**Status Codes:**

- **201 Created**: Alert message created successfully
- **400 Bad Request**: Missing or invalid fields
- **401 Unauthorized**: Invalid or missing API key
- **404 Not Found**: Alert class not found

PUT /alert-messages/{id}
^^^^^^^^^^^^^^^^^^^^^^^^

Update an existing alert message (full update).

**Request:**

.. code-block:: http

    PUT /alert-messages/6 HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "name": "High Pressure Warning",
      "message": "⚠️ WARNING: {device} pressure is {value} bar (Limit: {limit} bar)",
      "class_id": 3,
      "channels": ["mqtt", "email", "dashboard", "sms"]
    }

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "High Pressure Warning",
      "message": "⚠️ WARNING: {device} pressure is {value} bar (Limit: {limit} bar)",
      "class_id": 3,
      "channels": ["mqtt", "email", "dashboard", "sms"]
    }

**Status Codes:**

- **200 OK**: Alert message updated successfully
- **400 Bad Request**: Invalid fields
- **404 Not Found**: Alert message or class not found
- **401 Unauthorized**: Invalid or missing API key

PATCH /alert-messages/{id}
^^^^^^^^^^^^^^^^^^^^^^^^^^

Partially update an alert message.

**Request:**

.. code-block:: http

    PATCH /alert-messages/6 HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "name": "High Pressure Alert",
      "channels": ["mqtt", "dashboard"]
    }

**Response:**

.. code-block:: json

    {
      "id": 6,
      "name": "High Pressure Alert",
      "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
      "class_id": 3,
      "channels": ["mqtt", "dashboard"]
    }

**Status Codes:**

- **200 OK**: Alert message updated successfully
- **400 Bad Request**: Invalid fields
- **404 Not Found**: Alert message or class not found
- **401 Unauthorized**: Invalid or missing API key

DELETE /alert-messages/{id}
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Delete an alert message.

**Request:**

.. code-block:: http

    DELETE /alert-messages/6 HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "deleted": true,
      "id": 6
    }

**Status Codes:**

- **200 OK**: Alert message deleted successfully
- **404 Not Found**: Alert message not found
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Cannot delete - rules are using this message

POST /alert-messages/bulk-delete
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Delete multiple alert messages in a single request.

**Request:**

.. code-block:: http

    POST /alert-messages/bulk-delete HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "ids": [6, 7, 8]
    }

**Required Fields:**

- ``ids`` (array of integers): Array of alert message IDs to delete

**Response:**

.. code-block:: json

    {
      "deleted_count": 3,
      "failed_ids": [],
      "errors": []
    }

**Status Codes:**

- **200 OK**: Bulk delete completed
- **400 Bad Request**: Invalid request format
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Some messages cannot be deleted (will be in failed_ids)

3. Preview API
~~~~~~~~~~~~~~

POST /alert-messages/preview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Preview an alert message with sample data.

**Request:**

.. code-block:: http

    POST /alert-messages/preview HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "message": "🔥 CRITICAL: {device} temperature is {value}°C at {timestamp}",
      "data": {
        "device": "Crane-01",
        "value": "85.5",
        "timestamp": "2024-01-15T10:30:00Z"
      }
    }

**Required Fields:**

- ``message`` (string): Message template to preview
- ``data`` (object): Sample data for variables

**Response:**

.. code-block:: json

    {
      "original": "🔥 CRITICAL: {device} temperature is {value}°C at {timestamp}",
      "rendered": "🔥 CRITICAL: Crane-01 temperature is 85.5°C at 2024-01-15T10:30:00Z",
      "missing_variables": ["threshold", "operator"],
      "extracted_variables": ["device", "value", "timestamp"]
    }

**Status Codes:**

- **200 OK**: Preview generated successfully
- **400 Bad Request**: Invalid message template or data
- **401 Unauthorized**: Invalid or missing API key

GET /alert-messages/{id}/preview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Preview a specific alert message with default sample data.

**Request:**

.. code-block:: http

    GET /alert-messages/1/preview HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

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
      "missing_variables": [],
      "class_info": {
        "name": "Critical",
        "severity": 5,
        "color": "#DC2626",
        "icon": "fa-triangle-exclamation"
      },
      "channels": ["mqtt", "email", "sms", "dashboard"]
    }

**Status Codes:**

- **200 OK**: Preview generated successfully
- **404 Not Found**: Alert message not found
- **401 Unauthorized**: Invalid or missing API key

4. Configuration API
~~~~~~~~~~~~~~~~~~~~

GET /configuration/alert-settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Export all alert configuration.

**Request:**

.. code-block:: http

    GET /configuration/alert-settings HTTP/1.1
    Authorization: Bearer <api_key>

**Query Parameters:**

- ``format`` (optional, string): Export format (``json``, ``yaml``, ``csv``) - default: ``json``

**Response:**

.. code-block:: json

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

**Status Codes:**

- **200 OK**: Configuration exported successfully
- **401 Unauthorized**: Invalid or missing API key

POST /configuration/alert-settings/import
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Import alert configuration.

**Request:**

.. code-block:: http

    POST /configuration/alert-settings/import HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "alert_classes": [
        {
          "name": "Critical",
          "severity": 5,
          "color": "#DC2626",
          "icon": "fa-triangle-exclamation",
          "description": "Immediate action required"
        }
      ],
      "alert_messages": [
        {
          "name": "Critical Temperature",
          "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
          "class_id": 1,
          "channels": ["mqtt", "email", "sms", "dashboard"]
        }
      ],
      "options": {
        "overwrite_existing": true,
        "preserve_defaults": false
      }
    }

**Response:**

.. code-block:: json

    {
      "imported_classes": 5,
      "imported_messages": 5,
      "skipped_classes": 0,
      "skipped_messages": 0,
      "errors": []
    }

**Status Codes:**

- **200 OK**: Configuration imported successfully
- **400 Bad Request**: Invalid configuration format
- **401 Unauthorized**: Invalid or missing API key

POST /configuration/reset
^^^^^^^^^^^^^^^^^^^^^^^^^

Reset all alert configuration to defaults.

**Request:**

.. code-block:: http

    POST /configuration/reset HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "confirm": true
    }

**Required Fields:**

- ``confirm`` (boolean): Must be true to confirm reset

**Response:**

.. code-block:: json

    {
      "reset": true,
      "default_classes_count": 5,
      "default_messages_count": 5
    }

**Status Codes:**

- **200 OK**: Configuration reset successfully
- **400 Bad Request**: Missing confirmation
- **401 Unauthorized**: Invalid or missing API key

5. Utilities API
~~~~~~~~~~~~~~~~

GET /variables
^^^^^^^^^^^^^^

Get available variables for message templates.

**Request:**

.. code-block:: http

    GET /variables HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

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

**Status Codes:**

- **200 OK**: Variables retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

GET /channels
^^^^^^^^^^^^^

Get available delivery channels.

**Request:**

.. code-block:: http

    GET /channels HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

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

**Status Codes:**

- **200 OK**: Channels retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

GET /alert-colors
^^^^^^^^^^^^^^^^^

Get available color options for alert classes.

**Request:**

.. code-block:: http

    GET /alert-colors HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

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

**Status Codes:**

- **200 OK**: Colors retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

GET /alert-icons
^^^^^^^^^^^^^^^^

Get available icon options for alert classes.

**Request:**

.. code-block:: http

    GET /alert-icons HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

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

**Status Codes:**

- **200 OK**: Icons retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

POST /validate/message
^^^^^^^^^^^^^^^^^^^^^^

Validate a message template.

**Request:**

.. code-block:: http

    POST /validate/message HTTP/1.1
    Authorization: Bearer <api_key>
    Content-Type: application/json
    
    {
      "message": "🔥 {device} is at {value}°C"
    }

**Response:**

.. code-block:: json

    {
      "valid": true,
      "extracted_variables": ["device", "value"],
      "error": null
    }

**Status Codes:**

- **200 OK**: Validation completed
- **400 Bad Request**: Invalid message format
- **401 Unauthorized**: Invalid or missing API key

Error Responses
---------------

All error responses follow this format:

.. code-block:: json

    {
      "error": {
        "code": "ERROR_CODE",
        "message": "Human readable error message",
        "details": {
          "field": "specific error details"
        }
      }
    }

Common Error Codes:

.. list-table:: Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Code
     - Description
   * - **AUTH_001**
     - Invalid API key
   * - **AUTH_002**
     - Expired API key
   * - **AUTH_003**
     - Insufficient permissions
   * - **VAL_001**
     - Validation error
   * - **VAL_002**
     - Missing required field
   * - **VAL_003**
     - Invalid field value
   * - **NOT_FOUND_001**
     - Resource not found
   * - **CONFLICT_001**
     - Resource already exists
   * - **CONFLICT_002**
     - Cannot delete - in use
   * - **CONFLICT_003**
     - Cannot delete default resource
   * - **SERVER_001**
     - Internal server error

Rate Limiting
-------------

- **Standard Tier:** 100 requests per minute
- **Enterprise Tier:** 1000 requests per minute

**Headers:**

.. code-block:: http

    X-RateLimit-Limit: 100
    X-RateLimit-Remaining: 95
    X-RateLimit-Reset: 1609459200

Versioning
----------

API version is included in the URL path:

.. code-block:: http

    /v1/alert-classes

Changelog
---------

.. list-table:: API Changelog
   :widths: 25 75
   :header-rows: 1

   * - Version
     - Description
   * - **v1.0.0** (2024-01-15)
     - Initial release with basic CRUD operations
   * - **v1.1.0** (2024-02-01)
     - Added preview and configuration import/export
   * - **v1.2.0** (2024-03-01)
     - Added PATCH methods, bulk operations, and validation endpoints

SDK Examples
------------

Python Example
^^^^^^^^^^^^^^

.. code-block:: python

    import requests
    
    class UnivaAlertClient:
        def __init__(self, api_key, base_url="https://api.univagateway.com/v1"):
            self.base_url = base_url
            self.headers = {
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json"
            }
        
        def get_alert_classes(self):
            response = requests.get(f"{self.base_url}/alert-classes", headers=self.headers)
            return response.json()
        
        def create_alert_message(self, name, message, class_id, channels):
            payload = {
                "name": name,
                "message": message,
                "class_id": class_id,
                "channels": channels
            }
            response = requests.post(f"{self.base_url}/alert-messages", 
                                   headers=self.headers, 
                                   json=payload)
            return response.json()
        
        def preview_message(self, message, data):
            payload = {
                "message": message,
                "data": data
            }
            response = requests.post(f"{self.base_url}/alert-messages/preview",
                                   headers=self.headers,
                                   json=payload)
            return response.json()
    
    # Usage
    client = UnivaAlertClient("your_api_key")
    classes = client.get_alert_classes()
    print(classes)

JavaScript Example
^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

    class UnivaAlertAPI {
        constructor(apiKey) {
            this.baseUrl = 'https://api.univagateway.com/v1';
            this.headers = {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            };
        }
        
        async getAlertMessages(filters = {}) {
            const params = new URLSearchParams(filters);
            const response = await fetch(`${this.baseUrl}/alert-messages?${params}`, {
                headers: this.headers
            });
            return await response.json();
        }
        
        async createAlertClass(alertClass) {
            const response = await fetch(`${this.baseUrl}/alert-classes`, {
                method: 'POST',
                headers: this.headers,
                body: JSON.stringify(alertClass)
            });
            return await response.json();
        }
        
        async previewMessage(message, data) {
            const response = await fetch(`${this.baseUrl}/alert-messages/preview`, {
                method: 'POST',
                headers: this.headers,
                body: JSON.stringify({ message, data })
            });
            return await response.json();
        }
    }
    
    // Usage
    const api = new UnivaAlertAPI('your_api_key');
    const messages = await api.getAlertMessages({ class_id: 1 });

Support
-------

For API support, contact:

- **Email:** api-support@univagateway.com
- **Documentation:** https://docs.univagateway.com/api/alerts
- **Status Page:** https://status.univagateway.com
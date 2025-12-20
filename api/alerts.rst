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

Retrieve all alert classes.

**Request:**

.. code-block:: http

    GET /alert-classes HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": [
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
      ],
      "meta": {
        "count": 5,
        "timestamp": "2024-01-15T10:30:00Z"
      }
    }

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
      "status": "success",
      "data": {
        "id": 1,
        "name": "Critical",
        "severity": 5,
        "color": "#DC2626",
        "icon": "fa-triangle-exclamation",
        "description": "Immediate action required"
      }
    }

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

- ``name`` (string): Name of the alert class
- ``severity`` (integer): Severity level (1-5)
- ``color`` (string): Hex color code
- ``icon`` (string): FontAwesome icon class

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": {
        "id": 6,
        "name": "Emergency",
        "severity": 5,
        "color": "#DC2626",
        "icon": "fa-fire",
        "description": "Emergency situation requiring immediate action"
      },
      "message": "Alert class created successfully"
    }

**Status Codes:**

- **201 Created**: Alert class created successfully
- **400 Bad Request**: Missing or invalid fields
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Alert class with same name already exists

PUT /alert-classes/{id}
^^^^^^^^^^^^^^^^^^^^^^^

Update an existing alert class.

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
      "status": "success",
      "data": {
        "id": 6,
        "name": "Emergency",
        "severity": 5,
        "color": "#B91C1C",
        "icon": "fa-fire-flame-curved",
        "description": "Emergency situation - highest priority"
      },
      "message": "Alert class updated successfully"
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
      "status": "success",
      "message": "Alert class deleted successfully"
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

Retrieve all alert messages.

**Request:**

.. code-block:: http

    GET /alert-messages HTTP/1.1
    Authorization: Bearer <api_key>

**Query Parameters:**

- ``class_id`` (optional): Filter by alert class ID
- ``channel`` (optional): Filter by delivery channel
- ``limit`` (optional): Number of records per page (default: 20)
- ``page`` (optional): Page number (default: 1)

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": [
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
      ],
      "meta": {
        "count": 5,
        "page": 1,
        "total_pages": 1,
        "timestamp": "2024-01-15T10:30:00Z"
      }
    }

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
      "status": "success",
      "data": {
        "id": 1,
        "name": "Critical Temperature",
        "message": "🔥 CRITICAL: {device} temperature is {value}°C (Threshold: {threshold}°C)",
        "class_id": 1,
        "channels": ["mqtt", "email", "sms", "dashboard"],
        "variables": ["device", "value", "threshold"]
      }
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

- ``name`` (string): Name of the alert message
- ``message`` (string): Message template with variables
- ``class_id`` (integer): ID of the alert class
- ``channels`` (array): Array of delivery channels

**Valid Channels:**

- ``mqtt`` - MQTT protocol
- ``email`` - Email notification
- ``sms`` - SMS notification
- ``dashboard`` - Dashboard display
- ``webhook`` - Webhook endpoint

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": {
        "id": 6,
        "name": "High Pressure Alert",
        "message": "⚠️ {device} pressure is {value} bar (Threshold: {threshold} bar)",
        "class_id": 3,
        "channels": ["mqtt", "email", "dashboard"]
      },
      "message": "Alert message created successfully"
    }

**Status Codes:**

- **201 Created**: Alert message created successfully
- **400 Bad Request**: Missing or invalid fields
- **401 Unauthorized**: Invalid or missing API key
- **404 Not Found**: Alert class not found

PUT /alert-messages/{id}
^^^^^^^^^^^^^^^^^^^^^^^^

Update an existing alert message.

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
      "status": "success",
      "data": {
        "id": 6,
        "name": "High Pressure Warning",
        "message": "⚠️ WARNING: {device} pressure is {value} bar (Limit: {limit} bar)",
        "class_id": 3,
        "channels": ["mqtt", "email", "dashboard", "sms"]
      },
      "message": "Alert message updated successfully"
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
      "status": "success",
      "message": "Alert message deleted successfully"
    }

**Status Codes:**

- **200 OK**: Alert message deleted successfully
- **404 Not Found**: Alert message not found
- **401 Unauthorized**: Invalid or missing API key
- **409 Conflict**: Cannot delete - rules are using this message

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
        "timestamp": "2024-01-15T10:30:00Z",
        "location": "Workshop A",
        "tag": "Motor_Temperature"
      }
    }

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": {
        "original": "🔥 CRITICAL: {device} temperature is {value}°C at {timestamp}",
        "rendered": "🔥 CRITICAL: Crane-01 temperature is 85.5°C at 2024-01-15T10:30:00Z",
        "missing_variables": ["threshold", "operator"],
        "sample_data": {
          "device": "Crane-01",
          "value": "85.5",
          "timestamp": "2024-01-15T10:30:00Z",
          "location": "Workshop A",
          "tag": "Motor_Temperature"
        }
      }
    }

**Status Codes:**

- **200 OK**: Preview generated successfully
- **400 Bad Request**: Invalid message template
- **401 Unauthorized**: Invalid or missing API key

4. Configuration API
~~~~~~~~~~~~~~~~~~~~

GET /configuration/export
^^^^^^^^^^^^^^^^^^^^^^^^^

Export all alert configuration.

**Request:**

.. code-block:: http

    GET /configuration/export HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": {
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
        "meta": {
          "export_timestamp": "2024-01-15T10:30:00Z",
          "version": "1.0.0",
          "gateway": "Univa-GW-01"
        }
      }
    }

**Status Codes:**

- **200 OK**: Configuration exported successfully
- **401 Unauthorized**: Invalid or missing API key

POST /configuration/import
^^^^^^^^^^^^^^^^^^^^^^^^^^

Import alert configuration.

**Request:**

.. code-block:: http

    POST /configuration/import HTTP/1.1
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
      "status": "success",
      "data": {
        "imported_classes": 5,
        "imported_messages": 5,
        "skipped_classes": 0,
        "skipped_messages": 0,
        "errors": []
      },
      "message": "Configuration imported successfully"
    }

**Status Codes:**

- **200 OK**: Configuration imported successfully
- **400 Bad Request**: Invalid configuration format
- **401 Unauthorized**: Invalid or missing API key

5. Utilities API
~~~~~~~~~~~~~~~~

GET /variables/available
^^^^^^^^^^^^^^^^^^^^^^^^

Get available variables for message templates.

**Request:**

.. code-block:: http

    GET /variables/available HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": [
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
    }

**Status Codes:**

- **200 OK**: Variables retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

GET /channels/available
^^^^^^^^^^^^^^^^^^^^^^^

Get available delivery channels.

**Request:**

.. code-block:: http

    GET /channels/available HTTP/1.1
    Authorization: Bearer <api_key>

**Response:**

.. code-block:: json

    {
      "status": "success",
      "data": [
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
    }

**Status Codes:**

- **200 OK**: Channels retrieved successfully
- **401 Unauthorized**: Invalid or missing API key

Error Responses
---------------

All error responses follow this format:

.. code-block:: json

    {
      "status": "error",
      "error": {
        "code": "ERROR_CODE",
        "message": "Human readable error message",
        "details": {
          "field": "specific error details"
        }
      },
      "timestamp": "2024-01-15T10:30:00Z"
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

Webhooks
--------

Alert Trigger Webhook
^^^^^^^^^^^^^^^^^^^^^

When an alert is triggered, a webhook can be sent:

**Payload:**

.. code-block:: json

    {
      "event": "alert.triggered",
      "timestamp": "2024-01-15T10:30:00Z",
      "data": {
        "alert_id": 1,
        "alert_name": "Critical Temperature",
        "message": "🔥 CRITICAL: Crane-01 temperature is 85.5°C (Threshold: 80°C)",
        "class": {
          "name": "Critical",
          "severity": 5,
          "color": "#DC2626"
        },
        "channels": ["mqtt", "email", "sms", "dashboard"],
        "trigger_data": {
          "device": "Crane-01",
          "value": "85.5",
          "threshold": "80",
          "timestamp": "2024-01-15T10:30:00Z",
          "location": "Workshop A"
        },
        "gateway": "Univa-GW-01"
      }
    }

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
     - Added webhook support and rate limiting

SDK Examples
------------

Python Example
^^^^^^^^^^^^^^

.. code-block:: python

    import requests

    api_key = "your_api_key"
    base_url = "https://api.univagateway.com/v1"

    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    # Get all alert classes
    response = requests.get(f"{base_url}/alert-classes", headers=headers)
    classes = response.json()["data"]

JavaScript Example
^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

    const apiKey = "your_api_key";
    const baseUrl = "https://api.univagateway.com/v1";

    const headers = {
        "Authorization": `Bearer ${apiKey}`,
        "Content-Type": "application/json"
    };

    // Create new alert message
    fetch(`${baseUrl}/alert-messages`, {
        method: "POST",
        headers: headers,
        body: JSON.stringify({
            name: "High Vibration",
            message: "⚠️ {device} vibration is {value} mm/s",
            class_id: 3,
            channels: ["mqtt", "email", "dashboard"]
        })
    })
    .then(response => response.json())
    .then(data => console.log(data));

Support
-------

For API support, contact:

- **Email:** api-support@univagateway.com
- **Documentation:** https://docs.univagateway.com/api
- **Status Page:** https://status.univagateway.com
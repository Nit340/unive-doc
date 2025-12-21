Rules Engine API
================

Purpose
-------

The Rules Engine API manages conditional rules that monitor device data and trigger alerts when specific conditions are met. It provides CRUD operations for rule definitions with a simplified interface matching the Univa Gateway UI.

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

Rules Management API
~~~~~~~~~~~~~~~~~~~~

GET /rules
^^^^^^^^^^

Retrieve all rules with simplified response format.

**Query Parameters:**

* **status** (optional): Filter by enabled status - ``true``, ``false``
* **type** (optional): Filter by rule type - ``tag``, ``device``, ``schedule``
* **device** (optional): Filter by device name
* **sort** (optional): Sort field - ``name``, ``created_date``, ``last_triggered``
* **order** (optional): Sort order - ``asc``, ``desc`` (default: ``asc``)

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "RULE-2025-001",
        "name": "Overtemp-Motor",
        "enabled": true,
        "type": "tag",
        "last_triggered": "12m ago",
        "description": "Temperature > 80°C for 30s",
        "trigger_count": 12,
        "device": "Motor_Drive_01",
        "tag": "temperature_celsius",
        "selected_alerts": ["alert-001", "alert-002"],
        "created_date": "2025-01-15",
        "last_modified": "2025-01-15"
      }
    ]

**Status Codes:**

* **200 OK**: Successful retrieval
* **401 Unauthorized**: Invalid or missing API key
* **500 Internal Server Error**: Server error

GET /rules/{id}
^^^^^^^^^^^^^^^

Retrieve a specific rule by ID.

**Path Parameters:**

* **id** (string): Rule ID (e.g., "RULE-2025-001")

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "id": "RULE-2025-001",
      "name": "Overtemp-Motor",
      "enabled": true,
      "type": "tag",
      "description": "Temperature > 80°C for 30s",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 80,
      "unit": "°C",
      "duration": 30,
      "scaled_value": false,
      "rising_edge_only": true,
      "trigger_count": 12,
      "last_triggered": "12m ago",
      "created_date": "2025-01-15",
      "last_modified": "2025-01-15",
      "selected_alerts": ["alert-001", "alert-002"]
    }

**Status Codes:**

* **200 OK**: Successful retrieval
* **404 Not Found**: Rule not found
* **401 Unauthorized**: Invalid or missing API key

POST /rules
^^^^^^^^^^^

Create a new rule with simplified payload.

**Request:**

.. sourcecode:: http

    POST /rules HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "name": "High-Temperature-Alert",
      "enabled": true,
      "type": "tag",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 80,
      "unit": "°C",
      "duration": 30,
      "selected_alerts": ["alert-001"]
    }

**Required Fields:**

* **name** (string): Name of the rule (3-100 characters)
* **type** (string): Rule type - ``tag``, ``device``, ``schedule``
* **device** (string): Device name or "All Devices"
* **tag** (string): Tag to monitor
* **condition** (string): Condition operator - ``>``, ``<``, ``>=``, ``<=``, ``==``, ``!=``
* **value** (string/number): Threshold value
* **duration** (integer): Duration in seconds (0-86400)
* **selected_alerts** (array): Array of alert IDs to trigger

**Optional Fields:**

* **enabled** (boolean): Whether rule is enabled (default: true)
* **unit** (string): Unit of measurement
* **scaled_value** (boolean): Use scaled value (default: false)
* **rising_edge_only** (boolean): Trigger only on rising edge (default: true)

**Response:**

.. sourcecode:: http

    HTTP/1.1 201 Created
    Content-Type: application/json
    
    {
      "id": "RULE-2025-045",
      "name": "High-Temperature-Alert",
      "enabled": true,
      "type": "tag",
      "description": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 80,
      "unit": "°C",
      "duration": 30,
      "scaled_value": false,
      "rising_edge_only": true,
      "trigger_count": 0,
      "last_triggered": "Never",
      "created_date": "2025-01-15",
      "last_modified": "2025-01-15",
      "selected_alerts": ["alert-001"]
    }

**Status Codes:**

* **201 Created**: Rule created successfully
* **400 Bad Request**: Missing or invalid fields
* **401 Unauthorized**: Invalid or missing API key
* **404 Not Found**: One or more alerts not found
* **409 Conflict**: Rule with same name already exists

PUT /rules/{id}
^^^^^^^^^^^^^^^

Update an existing rule.

**Path Parameters:**

* **id** (string): Rule ID

**Request:**

.. sourcecode:: http

    PUT /rules/RULE-2025-001 HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "name": "Overtemp-Motor-Updated",
      "enabled": true,
      "type": "tag",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 85,
      "unit": "°C",
      "duration": 25,
      "selected_alerts": ["alert-001", "alert-002"]
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "id": "RULE-2025-001",
      "name": "Overtemp-Motor-Updated",
      "enabled": true,
      "type": "tag",
      "description": "Motor_Drive_01.temperature_celsius > 85°C for 25s",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 85,
      "unit": "°C",
      "duration": 25,
      "scaled_value": false,
      "rising_edge_only": true,
      "trigger_count": 12,
      "last_triggered": "12m ago",
      "created_date": "2025-01-15",
      "last_modified": "2025-01-15",
      "selected_alerts": ["alert-001", "alert-002"]
    }

**Status Codes:**

* **200 OK**: Rule updated successfully
* **400 Bad Request**: Invalid fields
* **404 Not Found**: Rule or alerts not found
* **401 Unauthorized**: Invalid or missing API key

PATCH /rules/{id}/status
^^^^^^^^^^^^^^^^^^^^^^^^

Update rule enabled status (UI toggle).

**Path Parameters:**

* **id** (string): Rule ID

**Request:**

.. sourcecode:: http

    PATCH /rules/RULE-2025-001/status HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "enabled": false
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "id": "RULE-2025-001",
      "name": "Overtemp-Motor-Updated",
      "enabled": false,
      "previous_status": true,
      "status_changed": true
    }

**Status Codes:**

* **200 OK**: Status updated successfully
* **404 Not Found**: Rule not found
* **401 Unauthorized**: Invalid or missing API key

DELETE /rules/{id}
^^^^^^^^^^^^^^^^^^

Delete a rule.

**Path Parameters:**

* **id** (string): Rule ID

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "deleted": true,
      "id": "RULE-2025-001",
      "name": "Overtemp-Motor-Updated"
    }

**Status Codes:**

* **200 OK**: Rule deleted successfully
* **404 Not Found**: Rule not found
* **401 Unauthorized**: Invalid or missing API key

Rule Templates API
~~~~~~~~~~~~~~~~~~

GET /rules/templates
^^^^^^^^^^^^^^^^^^^^

Get available rule templates for quick start (matching UI).

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "temp-template",
        "name": "Temperature Monitor",
        "icon": "fa-thermometer-half",
        "description": "Monitor temperature thresholds",
        "device": "Motor_Drive_01",
        "tag": "temperature_celsius",
        "condition": ">",
        "value": 80,
        "duration": 30,
        "unit": "°C",
        "default_alerts": ["alert-001"]
      },
      {
        "id": "device-template",
        "name": "Device Status",
        "icon": "fa-plug",
        "description": "Monitor device online/offline",
        "device": "All Devices",
        "tag": "status",
        "condition": "==",
        "value": "offline",
        "duration": 60,
        "unit": "",
        "default_alerts": ["alert-002"]
      },
      {
        "id": "vibration-template",
        "name": "Vibration Alert",
        "icon": "fa-wave-square",
        "description": "Monitor vibration levels",
        "device": "Pump_Station_A",
        "tag": "vibration_level",
        "condition": ">",
        "value": 5.0,
        "duration": 10,
        "unit": "mm/s",
        "default_alerts": ["alert-003"]
      },
      {
        "id": "pressure-template",
        "name": "Pressure Monitor",
        "icon": "fa-gauge-high",
        "description": "Monitor pressure values",
        "device": "Compressor_Unit_03",
        "tag": "pressure_bar",
        "condition": ">",
        "value": 10.0,
        "duration": 20,
        "unit": "bar",
        "default_alerts": ["alert-004"]
      },
      {
        "id": "current-template",
        "name": "Current Monitor",
        "icon": "fa-bolt",
        "description": "Monitor electrical current",
        "device": "Generator_Set_02",
        "tag": "current_amperes",
        "condition": ">",
        "value": 100,
        "duration": 15,
        "unit": "A",
        "default_alerts": ["alert-001"]
      },
      {
        "id": "schedule-template",
        "name": "Scheduled Alert",
        "icon": "fa-calendar-days",
        "description": "Time-based alerts",
        "device": "System",
        "tag": "time",
        "condition": "schedule",
        "value": "daily",
        "duration": 0,
        "unit": "",
        "default_alerts": ["alert-005"]
      }
    ]

**Status Codes:**

* **200 OK**: Templates retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

POST /rules/templates/{id}/create
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a rule from a template (UI "Start from Template" button).

**Path Parameters:**

* **id** (string): Template ID

**Request:**

.. sourcecode:: http

    POST /rules/templates/temp-template/create HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "name": "Custom Temperature Rule",
      "device": "Motor_02",
      "value": 75,
      "duration": 25
    }

**Optional Overrides:**

* **name** (string): Custom rule name
* **device** (string): Custom device
* **tag** (string): Custom tag
* **condition** (string): Custom condition
* **value** (string/number): Custom value
* **duration** (integer): Custom duration
* **selected_alerts** (array): Custom alert selection

**Response:**

.. sourcecode:: http

    HTTP/1.1 201 Created
    Content-Type: application/json
    
    {
      "id": "RULE-2025-046",
      "name": "Custom Temperature Rule",
      "enabled": true,
      "type": "tag",
      "description": "Motor_02.temperature_celsius > 75°C for 25s",
      "device": "Motor_02",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 75,
      "unit": "°C",
      "duration": 25,
      "scaled_value": false,
      "rising_edge_only": true,
      "trigger_count": 0,
      "last_triggered": "Never",
      "created_date": "2025-01-15",
      "last_modified": "2025-01-15",
      "selected_alerts": ["alert-001"]
    }

**Status Codes:**

* **201 Created**: Rule created successfully
* **404 Not Found**: Template not found
* **401 Unauthorized**: Invalid or missing API key

Rule Testing API
~~~~~~~~~~~~~~~~

POST /rules/test
^^^^^^^^^^^^^^^^

Test a rule configuration without saving (UI "Test Rule" button).

**Request:**

.. sourcecode:: http

    POST /rules/test HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "name": "Test Temperature Rule",
      "type": "tag",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 80,
      "unit": "°C",
      "duration": 30,
      "selected_alerts": ["alert-001"]
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "rule_name": "Test Temperature Rule",
      "rule_conditions": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
      "test_result": "would_trigger",
      "alerts_to_trigger": [
        {
          "id": "alert-001",
          "name": "High Temperature Alert",
          "status": "would_trigger",
          "channels": ["mqtt", "email", "dashboard"]
        }
      ],
      "simulation_result": "Rule would trigger 1 alert",
      "test_timestamp": "2025-01-15T11:50:30Z"
    }

**Status Codes:**

* **200 OK**: Test completed successfully
* **400 Bad Request**: Invalid rule configuration
* **401 Unauthorized**: Invalid or missing API key

POST /rules/{id}/test
^^^^^^^^^^^^^^^^^^^^^

Test an existing rule.

**Path Parameters:**

* **id** (string): Rule ID

**Request:**

.. sourcecode:: http

    POST /rules/RULE-2025-001/test HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "test_value": 90,
      "duration_met": 35
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "rule_id": "RULE-2025-001",
      "rule_name": "Overtemp-Motor",
      "test_conditions": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
      "test_value": 90,
      "duration_met": 35,
      "result": "condition_met",
      "alerts_to_trigger": [
        {
          "id": "alert-001",
          "name": "High Temperature Alert",
          "status": "would_trigger"
        }
      ],
      "test_summary": "Rule would trigger 1 alert"
    }

**Status Codes:**

* **200 OK**: Test completed successfully
* **404 Not Found**: Rule not found
* **401 Unauthorized**: Invalid or missing API key

Configuration Discovery API
~~~~~~~~~~~~~~~~~~~~~~~~~~~

GET /rules/devices
^^^^^^^^^^^^^^^^^^

Get available devices for dropdown (UI device selector).

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "Motor_Drive_01",
        "name": "Motor Drive 01",
        "type": "motor",
        "status": "online",
        "tags": ["temperature_celsius", "rpm_speed", "current_amperes"]
      },
      {
        "id": "Pump_Station_A",
        "name": "Pump Station A",
        "type": "pump",
        "status": "online",
        "tags": ["vibration_level", "pressure_bar", "flow_rate"]
      },
      {
        "id": "Compressor_Unit_03",
        "name": "Compressor Unit 03",
        "type": "compressor",
        "status": "online",
        "tags": ["pressure_bar", "temperature_celsius", "runtime"]
      },
      {
        "id": "Generator_Set_02",
        "name": "Generator Set 02",
        "type": "generator",
        "status": "offline",
        "tags": ["current_amperes", "voltage_volts", "frequency_hz"]
      }
    ]

**Status Codes:**

* **200 OK**: Devices retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

GET /rules/tags
^^^^^^^^^^^^^^^

Get available tags for dropdown (UI tag selector).

**Query Parameters:**

* **device** (optional): Filter tags by device ID

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "temperature_celsius",
        "name": "Temperature",
        "unit": "°C",
        "data_type": "float",
        "min_value": -40,
        "max_value": 150
      },
      {
        "id": "rpm_speed",
        "name": "Speed",
        "unit": "RPM",
        "data_type": "integer",
        "min_value": 0,
        "max_value": 3000
      },
      {
        "id": "vibration_level",
        "name": "Vibration",
        "unit": "mm/s",
        "data_type": "float",
        "min_value": 0,
        "max_value": 10.0
      },
      {
        "id": "current_amperes",
        "name": "Current",
        "unit": "A",
        "data_type": "float",
        "min_value": 0,
        "max_value": 200
      },
      {
        "id": "pressure_bar",
        "name": "Pressure",
        "unit": "bar",
        "data_type": "float",
        "min_value": 0,
        "max_value": 20.0
      },
      {
        "id": "status",
        "name": "Status",
        "unit": "",
        "data_type": "string",
        "valid_values": ["online", "offline", "error", "maintenance"]
      }
    ]

**Status Codes:**

* **200 OK**: Tags retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

GET /rules/alerts/available
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Get available alerts for selection (UI alert selector).

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "alert-001",
        "name": "High Temperature Alert",
        "enabled": true,
        "description": "Triggers when temperature exceeds limits"
      },
      {
        "id": "alert-002",
        "name": "Device Offline Alert",
        "enabled": true,
        "description": "Triggers when device goes offline"
      },
      {
        "id": "alert-003",
        "name": "Vibration Warning",
        "enabled": true,
        "description": "Triggers on high vibration levels"
      },
      {
        "id": "alert-004",
        "name": "Pressure Critical",
        "enabled": true,
        "description": "Triggers on critical pressure levels"
      },
      {
        "id": "alert-005",
        "name": "Maintenance Reminder",
        "enabled": false,
        "description": "Scheduled maintenance alerts"
      },
      {
        "id": "alert-006",
        "name": "Energy Consumption Alert",
        "enabled": true,
        "description": "High energy usage detection"
      }
    ]

**Status Codes:**

* **200 OK**: Alerts retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

Rule Import/Export API
~~~~~~~~~~~~~~~~~~~~~~

POST /rules/import
^^^^^^^^^^^^^^^^^^

Import rules from JSON (UI import button).

**Request:**

.. sourcecode:: http

    POST /rules/import HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "rules": [
        {
          "name": "Imported Temperature Rule",
          "type": "tag",
          "device": "Motor_Drive_01",
          "tag": "temperature_celsius",
          "condition": ">",
          "value": 80,
          "duration": 30,
          "selected_alerts": ["alert-001"]
        }
      ],
      "options": {
        "overwrite_existing": false,
        "enable_rules": true
      }
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "imported": 1,
      "skipped": 0,
      "failed": 0,
      "details": {
        "rules_created": 1,
        "rules_updated": 0
      },
      "import_id": "IMP-20250115-001"
    }

**Status Codes:**

* **200 OK**: Rules imported successfully
* **400 Bad Request**: Invalid import data
* **401 Unauthorized**: Invalid or missing API key

GET /rules/export
^^^^^^^^^^^^^^^^^

Export rules to JSON.

**Query Parameters:**

* **format** (optional): Export format - ``json`` (default)

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "export_url": "https://api.univagateway.com/v1/downloads/rules-export-2025-01-15.json",
      "expires_at": "2025-01-16T11:30:00Z",
      "summary": {
        "rules_exported": 45,
        "file_size": "45.2 KB"
      }
    }

**Status Codes:**

* **200 OK**: Export prepared successfully
* **401 Unauthorized**: Invalid or missing API key

Rule Statistics API
~~~~~~~~~~~~~~~~~~~

GET /rules/statistics
^^^^^^^^^^^^^^^^^^^^^

Get rule statistics for UI display.

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "total_rules": 45,
      "enabled_rules": 38,
      "disabled_rules": 7,
      "rules_by_type": {
        "tag": 32,
        "device": 8,
        "schedule": 5
      },
      "trigger_statistics": {
        "total_triggers": 1250,
        "triggers_today": 15,
        "most_active_rule": "RULE-2025-001",
        "most_active_count": 145
      }
    }

**Status Codes:**

* **200 OK**: Statistics retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

GET /rules/{id}/statistics
^^^^^^^^^^^^^^^^^^^^^^^^^^

Get statistics for a specific rule.

**Path Parameters:**

* **id** (string): Rule ID

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "rule_id": "RULE-2025-001",
      "rule_name": "Overtemp-Motor",
      "trigger_count": 12,
      "last_triggered": "12m ago",
      "last_triggered_timestamp": "2025-01-15T10:18:00Z",
      "created_date": "2025-01-15",
      "status": "enabled"
    }

**Status Codes:**

* **200 OK**: Statistics retrieved successfully
* **404 Not Found**: Rule not found
* **401 Unauthorized**: Invalid or missing API key

Rule Validation API
~~~~~~~~~~~~~~~~~~~

POST /rules/validate
^^^^^^^^^^^^^^^^^^^^

Validate rule configuration.

**Request:**

.. sourcecode:: http

    POST /rules/validate HTTP/1.1
    Content-Type: application/json
    Authorization: Bearer <api_key>
    
    {
      "name": "Test Rule",
      "type": "tag",
      "device": "Motor_Drive_01",
      "tag": "temperature_celsius",
      "condition": ">",
      "value": 80,
      "duration": 30
    }

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    {
      "valid": true,
      "issues": [],
      "suggestions": [
        "Consider adding hysteresis to prevent rapid triggering"
      ],
      "logic_preview": "IF Motor_Drive_01.temperature_celsius > 80°C for 30s THEN trigger alerts"
    }

**Status Codes:**

* **200 OK**: Validation completed
* **400 Bad Request**: Invalid configuration
* **401 Unauthorized**: Invalid or missing API key

Utility Endpoints
~~~~~~~~~~~~~~~~~

GET /rules/conditions
^^^^^^^^^^^^^^^^^^^^^

Get available condition operators.

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": ">",
        "name": "Greater than",
        "description": "Value is greater than threshold"
      },
      {
        "id": "<",
        "name": "Less than",
        "description": "Value is less than threshold"
      },
      {
        "id": ">=",
        "name": "Greater than or equal",
        "description": "Value is greater than or equal to threshold"
      },
      {
        "id": "<=",
        "name": "Less than or equal",
        "description": "Value is less than or equal to threshold"
      },
      {
        "id": "==",
        "name": "Equal to",
        "description": "Value is equal to threshold"
      },
      {
        "id": "!=",
        "name": "Not equal to",
        "description": "Value is not equal to threshold"
      }
    ]

**Status Codes:**

* **200 OK**: Conditions retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

GET /rules/units
^^^^^^^^^^^^^^^^

Get available units for tags.

**Response:**

.. sourcecode:: http

    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {
        "id": "°C",
        "name": "Celsius",
        "description": "Temperature in Celsius"
      },
      {
        "id": "RPM",
        "name": "Revolutions per minute",
        "description": "Rotational speed"
      },
      {
        "id": "mm/s",
        "name": "Millimeters per second",
        "description": "Vibration velocity"
      },
      {
        "id": "A",
        "name": "Amperes",
        "description": "Electrical current"
      },
      {
        "id": "bar",
        "name": "Bar",
        "description": "Pressure"
      }
    ]

**Status Codes:**

* **200 OK**: Units retrieved successfully
* **401 Unauthorized**: Invalid or missing API key

Error Responses
---------------

All error responses follow this format:

.. sourcecode:: json

   {
     "error": {
       "code": "ERROR_CODE",
       "message": "Human readable error message",
       "details": {}
     }
   }

Common Error Codes
^^^^^^^^^^^^^^^^^^

.. list-table:: Rules Engine Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **RULES_001**
     - Invalid rule configuration
   * - **RULES_002**
     - Rule not found
   * - **RULES_003**
     - Rule already exists
   * - **RULES_004**
     - Cannot delete - rule in use
   * - **RULES_005**
     - Invalid condition for rule type
   * - **RULES_006**
     - Device not found
   * - **RULES_007**
     - Tag not found on device
   * - **RULES_008**
     - Alert not found
   * - **RULES_009**
     - Rule validation failed
   * - **RULES_010**
     - Import/export error
   * - **RULES_011**
     - Test execution failed

Rate Limiting
-------------

* **Standard Tier:** 100 requests per minute
* **Enterprise Tier:** 1000 requests per minute

**Headers:**

.. code-block:: http

   X-RateLimit-Limit: 100
   X-RateLimit-Remaining: 95
   X-RateLimit-Reset: 1609459200

SDK Examples
------------

JavaScript Example (Matching UI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   const apiKey = "your_api_key";
   const baseUrl = "https://api.univagateway.com/v1";
   
   const headers = {
       "Authorization": `Bearer ${apiKey}`,
       "Content-Type": "application/json"
   };
   
   // Get all rules (for sidebar list)
   async function getRules() {
       const response = await fetch(`${baseUrl}/rules`, { headers });
       return await response.json();
   }
   
   // Create a new rule (from wizard)
   async function createRule(ruleData) {
       const response = await fetch(`${baseUrl}/rules`, {
           method: "POST",
           headers,
           body: JSON.stringify(ruleData)
       });
       return await response.json();
   }
   
   // Test a rule (from test button)
   async function testRule(ruleConfig) {
       const response = await fetch(`${baseUrl}/rules/test`, {
           method: "POST",
           headers,
           body: JSON.stringify(ruleConfig)
       });
       return await response.json();
   }
   
   // Get available alerts (for selection)
   async function getAvailableAlerts() {
       const response = await fetch(`${baseUrl}/rules/alerts/available`, { headers });
       return await response.json();
   }
   
   // Get devices (for dropdown)
   async function getDevices() {
       const response = await fetch(`${baseUrl}/rules/devices`, { headers });
       return await response.json();
   }
   
   // Get templates (for quick start)
   async function getTemplates() {
       const response = await fetch(`${baseUrl}/rules/templates`, { headers });
       return await response.json();
   }
   
   // Toggle rule status
   async function toggleRuleStatus(ruleId, enabled) {
       const response = await fetch(`${baseUrl}/rules/${ruleId}/status`, {
           method: "PATCH",
           headers,
           body: JSON.stringify({ enabled })
       });
       return await response.json();
   }

Python Example
^^^^^^^^^^^^^^

.. code-block:: python

   import requests
   
   class RulesAPI:
       def __init__(self, api_key):
           self.base_url = "https://api.univagateway.com/v1"
           self.headers = {
               "Authorization": f"Bearer {api_key}",
               "Content-Type": "application/json"
           }
       
       def get_rules(self, filters=None):
           params = filters or {}
           response = requests.get(f"{self.base_url}/rules", 
                                 headers=self.headers, 
                                 params=params)
           return response.json()
       
       def create_rule(self, rule_data):
           response = requests.post(f"{self.base_url}/rules",
                                  headers=self.headers,
                                  json=rule_data)
           return response.json()
       
       def test_rule(self, rule_config):
           response = requests.post(f"{self.base_url}/rules/test",
                                  headers=self.headers,
                                  json=rule_config)
           return response.json()
       
       def get_devices(self):
           response = requests.get(f"{self.base_url}/rules/devices",
                                 headers=self.headers)
           return response.json()
       
       def get_alerts(self):
           response = requests.get(f"{self.base_url}/rules/alerts/available",
                                 headers=self.headers)
           return response.json()
   
   # Usage example
   api = RulesAPI("your_api_key")
   rules = api.get_rules()
   print(f"Found {len(rules)} rules")

Support
-------

For API support, contact:

* **Email:** api-support@univagateway.com
* **Documentation:** https://docs.univagateway.com/api/rules
* **Status Page:** https://status.univagateway.com

Webhook Integration
-------------------

Rule Triggered Webhook
^^^^^^^^^^^^^^^^^^^^^^

When a rule is triggered, a webhook can be sent:

**Payload:**

.. code-block:: json

   {
     "event": "rule.triggered",
     "timestamp": "2025-01-15T10:18:00Z",
     "data": {
       "rule_id": "RULE-2025-001",
       "rule_name": "Overtemp-Motor",
       "trigger_value": 82.5,
       "condition": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
       "duration_met": 32,
       "alerts_triggered": ["alert-001", "alert-002"],
       "device": "Motor_Drive_01",
       "tag": "temperature_celsius"
     }
   }

Rule Status Changed Webhook
^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a rule status changes (enabled/disabled):

.. code-block:: json

   {
     "event": "rule.status_changed",
     "timestamp": "2025-01-15T11:30:00Z",
     "data": {
       "rule_id": "RULE-2025-001",
       "rule_name": "Overtemp-Motor",
       "previous_status": true,
       "new_status": false,
       "changed_by": "user@example.com"
     }
   }

Data Types and Constraints
--------------------------

Rule Object
^^^^^^^^^^^

.. list-table:: Rule Object Structure
   :widths: 25 50 25
   :header-rows: 1

   * - Field
     - Description
     - Constraints
   * - **id**
     - Unique rule identifier
     - Format: "RULE-YYYY-NNN"
   * - **name**
     - Rule name
     - 3-100 characters, unique
   * - **enabled**
     - Whether rule is active
     - boolean
   * - **type**
     - Rule type
     - "tag", "device", "schedule"
   * - **device**
     - Target device
     - Device ID or "All Devices"
   * - **tag**
     - Tag to monitor
     - Valid tag ID for device
   * - **condition**
     - Comparison operator
     - ">", "<", ">=", "<=", "==", "!="
   * - **value**
     - Threshold value
     - Number or string
   * - **unit**
     - Measurement unit
     - String, optional
   * - **duration**
     - Hold duration
     - 0-86400 seconds
   * - **selected_alerts**
     - Alerts to trigger
     - Array of alert IDs
   * - **description**
     - Auto-generated description
     - Read-only
   * - **trigger_count**
     - Times triggered
     - Read-only integer
   * - **last_triggered**
     - Human-readable timestamp
     - Read-only string
   * - **created_date**
     - Creation date
     - YYYY-MM-DD format
   * - **last_modified**
     - Last modified date
     - YYYY-MM-DD format

Validation Rules
^^^^^^^^^^^^^^^^

1. **Name Validation:**
   - Must be unique across all rules
   - 3-100 characters
   - Alphanumeric, hyphen, underscore, space allowed

2. **Value Validation:**
   - Must match tag data type
   - Numbers must be within tag's min/max range
   - Strings must match tag's valid_values if defined

3. **Duration Validation:**
   - 0-86400 seconds (0-24 hours)
   - 0 = immediate trigger (no hold)

4. **Alert Validation:**
   - Selected alerts must exist
   - Maximum 10 alerts per rule
   - Only enabled alerts can be selected

Performance Notes
-----------------

1. **Rule Evaluation:**
   - Active rules evaluated every 5 seconds
   - Up to 1000 rules per gateway
   - Complex conditions may impact performance

2. **Response Times:**
   - GET requests: < 100ms
   - POST/PUT requests: < 500ms
   - Test requests: < 1000ms

3. **Rate Limits:**
   - 100 requests/minute per API key
   - Bulk operations count as single request

Changelog
---------

.. list-table:: API Changelog
   :widths: 25 75
   :header-rows: 1

   * - Version
     - Description
   * - **v1.0.0** (2025-01-15)
     - Initial release with UI-compatible endpoints
   * - **v1.1.0** (2025-02-01)
     - Added template and discovery endpoints
   * - **v1.2.0** (2025-03-01)
     - Added test and validation endpoints

Migration Notes
---------------

From previous version (if any):

1. **Rule ID format changed** to "RULE-YYYY-NNN"
2. **Simplified response structure** - removed nested "data" and "meta"
3. **Added UI-focused endpoints** for devices, tags, alerts discovery
4. **Removed complex execution history** - simplified to "last_triggered" field
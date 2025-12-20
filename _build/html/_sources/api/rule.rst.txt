Rules Engine API
================

Purpose
-------

The Rules Engine API manages conditional rules that monitor device data and trigger alerts when specific conditions are met. It provides CRUD operations for rule definitions, testing, and management.

Base URL
--------

.. code-block:: text

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

Get All Rules
^^^^^^^^^^^^^

.. http:get:: /rules

   Retrieve all rules with optional filtering.

   **Query Parameters:**

   * **status** (optional): Filter by status - ``enabled``, ``disabled``
   * **type** (optional): Filter by rule type - ``tag``, ``device``, ``schedule``
   * **device** (optional): Filter by device name
   * **page** (optional): Page number (default: ``1``)
   * **limit** (optional): Items per page (default: ``20``)
   * **sort** (optional): Sort field - ``name``, ``created_date``, ``last_triggered``
   * **order** (optional): Sort order - ``asc``, ``desc`` (default: ``asc``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": [
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
            "trigger_count": 12,
            "last_triggered": "2024-01-15T10:18:00Z",
            "created_date": "2024-01-10T08:30:00Z",
            "last_modified": "2024-01-15T09:45:00Z",
            "selected_alerts": ["alert-001", "alert-002"]
          }
        ],
        "meta": {
          "count": 2,
          "total": 45,
          "page": 1,
          "total_pages": 3
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **401 Unauthorized**: Invalid or missing API key
   * **500 Internal Server Error**: Server error

Get Single Rule
^^^^^^^^^^^^^^^

.. http:get:: /rules/{id}

   Retrieve a specific rule by ID.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
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
          "last_triggered": "2024-01-15T10:18:00Z",
          "created_date": "2024-01-10T08:30:00Z",
          "last_modified": "2024-01-15T09:45:00Z",
          "selected_alerts": [
            {
              "id": "alert-001",
              "name": "High Temperature Alert",
              "channels": ["mqtt", "email", "dashboard"]
            }
          ]
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key

Create Rule
^^^^^^^^^^^

.. http:post:: /rules

   Create a new rule.

   **Request:**

   .. sourcecode:: http

      POST /rules HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "High-Pressure-Alert",
        "enabled": true,
        "type": "tag",
        "device": "Compressor_Unit_03",
        "tag": "pressure_bar",
        "condition": ">",
        "value": 10.5,
        "unit": "bar",
        "duration": 20,
        "scaled_value": false,
        "rising_edge_only": true,
        "selected_alerts": ["alert-003", "alert-004"]
      }

   **Required Fields:**

   * **name** (string): Name of the rule
   * **type** (string): Rule type - ``tag``, ``device``, ``schedule``
   * **device** (string): Device name or "All Devices"
   * **tag** (string): Tag to monitor
   * **condition** (string): Condition operator - ``>``, ``<``, ``>=``, ``<=``, ``==``, ``!=``
   * **value** (string/number): Threshold value
   * **duration** (integer): Duration in seconds

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "RULE-2025-045",
          "name": "High-Pressure-Alert",
          "enabled": true,
          "type": "tag",
          "description": "Compressor_Unit_03.pressure_bar > 10.5bar for 20s",
          "device": "Compressor_Unit_03",
          "tag": "pressure_bar",
          "condition": ">",
          "value": 10.5,
          "unit": "bar",
          "duration": 20,
          "scaled_value": false,
          "rising_edge_only": true,
          "trigger_count": 0,
          "last_triggered": null,
          "created_date": "2024-01-15T11:30:00Z",
          "last_modified": "2024-01-15T11:30:00Z",
          "selected_alerts": ["alert-003", "alert-004"]
        },
        "message": "Rule created successfully"
      }

   **Status Codes:**

   * **201 Created**: Rule created successfully
   * **400 Bad Request**: Missing or invalid fields
   * **401 Unauthorized**: Invalid or missing API key
   * **404 Not Found**: One or more alerts not found
   * **409 Conflict**: Rule with same name already exists

Update Rule
^^^^^^^^^^^

.. http:put:: /rules/{id}

   Update an existing rule.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Request:**

   .. sourcecode:: http

      PUT /rules/RULE-2025-001 HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Overtemp-Motor-Updated",
        "enabled": true,
        "type": "tag",
        "device": "Motor_Drive_01",
        "tag": "temperature_celsius",
        "condition": ">=",
        "value": 85,
        "unit": "°C",
        "duration": 25,
        "scaled_value": true,
        "rising_edge_only": false,
        "selected_alerts": ["alert-001", "alert-002", "alert-006"]
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "RULE-2025-001",
          "name": "Overtemp-Motor-Updated",
          "enabled": true,
          "type": "tag",
          "description": "Motor_Drive_01.temperature_celsius >= 85°C for 25s",
          "device": "Motor_Drive_01",
          "tag": "temperature_celsius",
          "condition": ">=",
          "value": 85,
          "unit": "°C",
          "duration": 25,
          "scaled_value": true,
          "rising_edge_only": false,
          "trigger_count": 12,
          "last_triggered": "2024-01-15T10:18:00Z",
          "created_date": "2024-01-10T08:30:00Z",
          "last_modified": "2024-01-15T11:45:00Z",
          "selected_alerts": ["alert-001", "alert-002", "alert-006"]
        },
        "message": "Rule updated successfully"
      }

   **Status Codes:**

   * **200 OK**: Rule updated successfully
   * **400 Bad Request**: Invalid fields
   * **404 Not Found**: Rule or alerts not found
   * **401 Unauthorized**: Invalid or missing API key

Update Rule Status
^^^^^^^^^^^^^^^^^^

.. http:patch:: /rules/{id}/status

   Update rule enabled status.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Request:**

   .. sourcecode:: http

      PATCH /rules/RULE-2025-001/status HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "enabled": false
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "RULE-2025-001",
          "enabled": false,
          "previous_status": true
        },
        "message": "Rule disabled successfully"
      }

   **Status Codes:**

   * **200 OK**: Status updated successfully
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key

Delete Rule
^^^^^^^^^^^

.. http:delete:: /rules/{id}

   Delete a rule.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "message": "Rule deleted successfully"
      }

   **Status Codes:**

   * **200 OK**: Rule deleted successfully
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key

Rule Templates API
~~~~~~~~~~~~~~~~~~

Get Rule Templates
^^^^^^^^^^^^^^^^^^

.. http:get:: /rules/templates

   Get available rule templates for quick start.

   **Query Parameters:**

   * **category** (optional): Filter by category - ``temperature``, ``device``, ``vibration``, ``pressure``, ``current``, ``schedule``

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": [
          {
            "id": "temp-template",
            "name": "Temperature Monitor",
            "icon": "fa-thermometer-half",
            "description": "Monitor temperature thresholds",
            "category": "temperature",
            "default_config": {
              "type": "tag",
              "device": "Motor_Drive_01",
              "tag": "temperature_celsius",
              "condition": ">",
              "value": 80,
              "unit": "°C",
              "duration": 30,
              "recommended_alerts": ["alert-001"]
            }
          }
        ]
      }

   **Status Codes:**

   * **200 OK**: Templates retrieved successfully
   * **401 Unauthorized**: Invalid or missing API key

Create Rule from Template
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /rules/templates/{id}/create

   Create a rule from a template.

   **Path Parameters:**

   * **id** (string): Template ID

   **Request:**

   .. sourcecode:: http

      POST /rules/templates/temp-template/create HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Custom Temperature Rule",
        "device": "Motor_02",
        "value": 75,
        "duration": 25
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
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
          "last_triggered": null,
          "created_date": "2024-01-15T11:45:00Z",
          "last_modified": "2024-01-15T11:45:00Z",
          "selected_alerts": ["alert-001"]
        },
        "message": "Rule created from template successfully"
      }

   **Status Codes:**

   * **201 Created**: Rule created successfully
   * **404 Not Found**: Template not found
   * **401 Unauthorized**: Invalid or missing API key

Rule Testing API
~~~~~~~~~~~~~~~~

Test Rule Configuration
^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /rules/test

   Test a rule configuration without saving.

   **Request:**

   .. sourcecode:: http

      POST /rules/test HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Test Temperature Rule",
        "type": "tag",
        "device": "Motor_Drive_01",
        "tag": "temperature_celsius",
        "condition": ">",
        "value": 80,
        "unit": "°C",
        "duration": 30,
        "selected_alerts": ["alert-001", "alert-002"],
        "test_data": {
          "current_value": 85.5,
          "timestamp": "2024-01-15T11:50:00Z"
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "rule_name": "Test Temperature Rule",
          "test_conditions": {
            "condition": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
            "test_value": 85.5,
            "result": "condition_met",
            "message": "Test value exceeds threshold"
          },
          "alerts_to_trigger": [
            {
              "id": "alert-001",
              "name": "High Temperature Alert",
              "status": "would_trigger",
              "channels": ["mqtt", "email", "dashboard"]
            }
          ],
          "simulation_result": "Rule would trigger 2 alerts",
          "timestamp": "2024-01-15T11:50:30Z"
        }
      }

   **Status Codes:**

   * **200 OK**: Test completed successfully
   * **400 Bad Request**: Invalid rule configuration
   * **401 Unauthorized**: Invalid or missing API key

Test Existing Rule
^^^^^^^^^^^^^^^^^^

.. http:post:: /rules/{id}/test

   Test an existing rule with simulated data.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Request:**

   .. sourcecode:: http

      POST /rules/RULE-2025-001/test HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "test_data": {
          "current_value": 90,
          "timestamp": "2024-01-15T11:55:00Z",
          "duration_met": 35
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "rule_id": "RULE-2025-001",
          "rule_name": "Overtemp-Motor",
          "test_conditions": {
            "condition": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
            "test_value": 90,
            "duration_met": 35,
            "result": "condition_met",
            "message": "Both threshold and duration conditions met"
          },
          "alerts_to_trigger": [
            {
              "id": "alert-001",
              "name": "High Temperature Alert",
              "status": "would_trigger"
            }
          ],
          "trigger_summary": "Rule would trigger 2 alerts through configured channels",
          "timestamp": "2024-01-15T11:55:30Z"
        }
      }

   **Status Codes:**

   * **200 OK**: Test completed successfully
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key

Rule Execution API
~~~~~~~~~~~~~~~~~~

Get Rule Executions
^^^^^^^^^^^^^^^^^^^

.. http:get:: /rules/{id}/executions

   Get execution history for a rule.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Query Parameters:**

   * **start_date** (optional): Start date for filtering (ISO format)
   * **end_date** (optional): End date for filtering (ISO format)
   * **limit** (optional): Maximum executions to return (default: ``50``)
   * **status** (optional): Filter by status - ``triggered``, ``suppressed``, ``error``

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "rule_id": "RULE-2025-001",
          "rule_name": "Overtemp-Motor",
          "total_executions": 12,
          "executions": [
            {
              "execution_id": "EXEC-001",
              "timestamp": "2024-01-15T10:18:00Z",
              "trigger_value": 82.5,
              "duration_met": 32,
              "status": "triggered",
              "alerts_triggered": ["alert-001", "alert-002"],
              "message": "Temperature threshold exceeded"
            }
          ],
          "statistics": {
            "last_24h": 1,
            "last_7d": 4,
            "last_30d": 12,
            "success_rate": 100
          }
        },
        "meta": {
          "count": 2,
          "total": 12,
          "page": 1,
          "total_pages": 1
        }
      }

   **Status Codes:**

   * **200 OK**: Executions retrieved successfully
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key

Execute Rule Manually
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /rules/{id}/execute

   Manually execute/trigger a rule.

   **Path Parameters:**

   * **id** (string): Rule ID

   **Request:**

   .. sourcecode:: http

      POST /rules/RULE-2025-001/execute HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "trigger_data": {
          "value": 85.5,
          "timestamp": "2024-01-15T12:00:00Z",
          "duration": 35,
          "force_trigger": true
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "execution_id": "EXEC-013",
          "rule_id": "RULE-2025-001",
          "timestamp": "2024-01-15T12:00:05Z",
          "result": "triggered",
          "alerts_triggered": ["alert-001", "alert-002"],
          "alert_statuses": [
            {
              "alert_id": "alert-001",
              "status": "sent",
              "channels": ["mqtt", "email", "dashboard"]
            }
          ],
          "message": "Rule triggered successfully, 2 alerts sent"
        }
      }

   **Status Codes:**

   * **200 OK**: Rule executed successfully
   * **404 Not Found**: Rule not found
   * **401 Unauthorized**: Invalid or missing API key
   * **400 Bad Request**: Rule is disabled

Rule Configuration API
~~~~~~~~~~~~~~~~~~~~~~

Get Configuration Options
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /rules/configuration/options

   Get available options for rule configuration.

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "rule_types": [
            {
              "id": "tag",
              "name": "Tag Value",
              "description": "Monitor tag values against thresholds",
              "icon": "fa-tags",
              "supported_conditions": [">", "<", ">=", "<=", "==", "!="]
            }
          ],
          "devices": [
            {
              "id": "Motor_Drive_01",
              "name": "Motor Drive 01",
              "type": "motor",
              "tags": ["temperature_celsius", "rpm_speed", "current_amperes"]
            }
          ],
          "tags": [
            {
              "id": "temperature_celsius",
              "name": "Temperature",
              "unit": "°C",
              "data_type": "float",
              "min_value": -40,
              "max_value": 150
            }
          ],
          "duration_units": [
            { "value": "seconds", "label": "Seconds" },
            { "value": "minutes", "label": "Minutes" },
            { "value": "hours", "label": "Hours" }
          ]
        }
      }

   **Status Codes:**

   * **200 OK**: Options retrieved successfully
   * **401 Unauthorized**: Invalid or missing API key

Validate Rule Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /rules/validate

   Validate a rule configuration.

   **Request:**

   .. sourcecode:: http

      POST /rules/validate HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
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
        "status": "success",
        "data": {
          "valid": true,
          "issues": [],
          "suggestions": [
            "Consider adding hysteresis to prevent rapid triggering"
          ],
          "logic_preview": "IF Motor_Drive_01.temperature_celsius > 80°C for 30s THEN trigger alerts"
        }
      }

   **Status Codes:**

   * **200 OK**: Validation completed
   * **400 Bad Request**: Invalid configuration
   * **401 Unauthorized**: Invalid or missing API key

Rule Import/Export API
~~~~~~~~~~~~~~~~~~~~~~

Export Rules
^^^^^^^^^^^^

.. http:get:: /rules/export

   Export rules to JSON file.

   **Query Parameters:**

   * **format** (optional): Export format - ``json`` (default), ``yaml``
   * **include_alerts** (optional): Include alert definitions - ``true``, ``false`` (default: ``true``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "export_url": "https://api.univagateway.com/v1/downloads/rules-export-2025-01-15.json",
          "expires_at": "2024-01-16T11:30:00Z",
          "summary": {
            "rules_exported": 45,
            "alerts_included": 25,
            "file_size": "45.2 KB"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Export prepared successfully
   * **401 Unauthorized**: Invalid or missing API key

Import Rules
^^^^^^^^^^^^

.. http:post:: /rules/import

   Import rules from JSON file.

   **Request:**

   .. sourcecode:: http

      POST /rules/import HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "rules": [
          {
            "name": "Imported Rule",
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
          "enable_rules": true,
          "create_missing_alerts": true
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "imported": 1,
          "skipped": 0,
          "failed": 0,
          "details": {
            "rules_created": 1,
            "rules_updated": 0,
            "alerts_created": 0
          },
          "import_id": "IMP-20250115-001"
        },
        "message": "Rules imported successfully"
      }

   **Status Codes:**

   * **200 OK**: Rules imported successfully
   * **400 Bad Request**: Invalid import data
   * **401 Unauthorized**: Invalid or missing API key

Rule Statistics API
~~~~~~~~~~~~~~~~~~~

Get Statistics
^^^^^^^^^^^^^^

.. http:get:: /rules/statistics

   Get overall statistics for rules.

   **Query Parameters:**

   * **time_range** (optional): Time range - ``today``, ``yesterday``, ``last_7d``, ``last_30d``, ``custom``
   * **start_date** (optional): Custom start date (ISO format)
   * **end_date** (optional): Custom end date (ISO format)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
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
            "triggers_last_7d": 120,
            "triggers_last_30d": 480,
            "most_active_rule": "RULE-2025-001",
            "most_active_count": 145
          },
          "alert_statistics": {
            "total_alerts_triggered": 2850,
            "average_alerts_per_trigger": 2.28,
            "most_common_alert": "alert-001",
            "most_common_count": 850
          },
          "time_based_statistics": {
            "peak_hour": 14,
            "peak_day": "Monday",
            "average_duration": 32.5
          }
        },
        "timestamp": "2024-01-15T12:00:00Z"
      }

   **Status Codes:**

   * **200 OK**: Statistics retrieved successfully
   * **401 Unauthorized**: Invalid or missing API key

Error Responses
---------------

All error responses follow this format:

.. sourcecode:: json

   {
     "status": "error",
     "error": {
       "code": "ERROR_CODE",
       "message": "Human readable error message",
       "details": {
         "field": "specific error details"
       }
     },
     "timestamp": "2024-01-15T12:00:00Z"
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

Webhooks
--------

Rule Triggered Webhook
^^^^^^^^^^^^^^^^^^^^^^

When a rule is triggered, a webhook can be sent:

**Endpoint Configuration:**

.. code-block:: json

   {
     "url": "https://your-webhook-url.com/rules/triggered",
     "event_types": ["rule.triggered", "rule.error"],
     "secret": "webhook_secret",
     "enabled": true
   }

**Webhook Payload:**

.. code-block:: json

   {
     "event": "rule.triggered",
     "timestamp": "2024-01-15T10:18:00Z",
     "data": {
       "rule_id": "RULE-2025-001",
       "rule_name": "Overtemp-Motor",
       "trigger_value": 82.5,
       "condition": "Motor_Drive_01.temperature_celsius > 80°C for 30s",
       "duration_met": 32,
       "alerts_triggered": ["alert-001", "alert-002"],
       "device": "Motor_Drive_01",
       "tag": "temperature_celsius",
       "gateway": "Univa-GW-01"
     }
   }

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
   
   # Create a new rule
   rule_data = {
       "name": "High-Temperature-Alert",
       "type": "tag",
       "device": "Motor_Drive_01",
       "tag": "temperature_celsius",
       "condition": ">",
       "value": 80,
       "duration": 30,
       "selected_alerts": ["alert-001"]
   }
   
   response = requests.post(
       f"{base_url}/rules",
       headers=headers,
       json=rule_data
   )
   
   if response.status_code == 201:
       rule = response.json()["data"]
       print(f"Rule created: {rule['id']}")

JavaScript Example
^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   const apiKey = "your_api_key";
   const baseUrl = "https://api.univagateway.com/v1";
   
   const headers = {
       "Authorization": `Bearer ${apiKey}`,
       "Content-Type": "application/json"
   };
   
   // Test a rule
   fetch(`${baseUrl}/rules/test`, {
       method: "POST",
       headers: headers,
       body: JSON.stringify({
           name: "Test Rule",
           type: "tag",
           device: "Motor_Drive_01",
           tag: "temperature_celsius",
           condition: ">",
           value: 80,
           duration: 30,
           selected_alerts: ["alert-001"],
           test_data: {
               current_value: 85.5,
               timestamp: new Date().toISOString()
           }
       })
   })
   .then(response => response.json())
   .then(data => console.log(data.data.simulation_result));

Support
-------

For API support, contact:

* **Email:** api-support@univagateway.com
* **Documentation:** https://docs.univagateway.com/api/rules
* **Status Page:** https://status.univagateway.com

Data Types and Constraints
--------------------------

Rule Types
^^^^^^^^^^

.. list-table:: Rule Types
   :widths: 30 50 20
   :header-rows: 1

   * - Type
     - Description
     - Supported Conditions
   * - **tag**
     - Monitor tag values against thresholds
     - ``>``, ``<``, ``>=``, ``<=``, ``==``, ``!=``
   * - **device**
     - Monitor device status changes
     - ``==``, ``!=``
   * - **schedule**
     - Time-based triggers
     - ``at``, ``every``, ``between``

Condition Operators
^^^^^^^^^^^^^^^^^^^

.. list-table:: Condition Operators
   :widths: 20 50 30
   :header-rows: 1

   * - Operator
     - Description
     - Data Types
   * - **>**
     - Greater than
     - Number
   * - **<**
     - Less than
     - Number
   * - **>=**
     - Greater than or equal
     - Number
   * - **<=**
     - Less than or equal
     - Number
   * - **==**
     - Equal to
     - Number, String, Boolean
   * - **!=**
     - Not equal to
     - Number, String, Boolean

Validation Rules
^^^^^^^^^^^^^^^^

1. **Name Validation:**
   - Must be unique
   - 3-50 characters
   - Alphanumeric, hyphen, underscore only

2. **Value Validation:**
   - Numbers must be within tag's min/max range
   - Strings must match tag's valid_values if defined
   - Duration must be 1-86400 seconds (1-24 hours)

3. **Alert Validation:**
   - Selected alerts must exist
   - Maximum 10 alerts per rule

Performance Considerations
--------------------------

1. **Rule Evaluation Frequency:**
   - Rules are evaluated every second
   - Consider total number of active rules
   - Complex conditions may impact performance

2. **Database Optimization:**
   - Index on rule_id for executions table
   - Partition execution history by date
   - Cache frequently accessed rule configurations

3. **Alert Throttling:**
   - Minimum 30 seconds between identical alerts
   - Configurable alert cooldown periods
   - Alert batching for high-frequency triggers

Best Practices
--------------

1. **Rule Design:**
   - Use descriptive rule names
   - Include hysteresis to prevent rapid triggering
   - Set realistic duration thresholds
   - Group related rules by device or category

2. **Alert Configuration:**
   - Configure appropriate alert channels
   - Set up escalation policies
   - Include contextual information in alert messages
   - Test alerts before production deployment

3. **Monitoring:**
   - Monitor rule trigger rates
   - Set up alerts for rule execution failures
   - Regularly review and optimize rule logic
   - Archive old execution data periodically

Troubleshooting
---------------

Common Issues
^^^^^^^^^^^^^

1. **Rule Not Triggering:**
   - Verify rule is enabled
   - Check tag data is being received
   - Validate condition logic
   - Confirm duration threshold is met

2. **Alerts Not Sending:**
   - Verify alert configuration
   - Check alert channel connectivity
   - Review alert throttling settings
   - Check alert cooldown periods

3. **Performance Issues:**
   - Reduce number of active rules
   - Increase evaluation intervals
   - Optimize database queries
   - Scale hardware resources

Debug Tips
^^^^^^^^^^

1. **Enable Debug Logging:**
   - Set log level to DEBUG for rules engine
   - Capture rule evaluation details
   - Log alert delivery attempts

2. **Use Test Endpoints:**
   - Test rule logic before deployment
   - Simulate various input scenarios
   - Validate alert configurations

3. **Monitor Metrics:**
   - Track rule evaluation times
   - Monitor alert delivery success rates
   - Measure database query performance
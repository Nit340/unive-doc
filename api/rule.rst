Rules Engine API
================

Purpose
-------

The Rules Engine API manages conditional rules that monitor device data and trigger alerts when specific conditions are met. It provides CRUD operations for rule definitions with a simplified interface matching the Univa Gateway UI.

Page Route (Frontend)
---------------------

.. http:get:: /rules-engine

   **Description**: Renders the complete rules engine management page with all rules, templates, and statistics embedded in the HTML.
   
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
        <title>Rules Engine - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-rules='[...]' data-templates='[...]' data-statistics='{...}'>
          <!-- Rules engine page with:
               - Rules list sidebar
               - Rule editor wizard (3-step)
               - Quick start templates
               - Test functionality
               - Import/export options
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - All rules, templates, and statistics are embedded in the page
   - JavaScript reads this data and renders the rules management interface
   - No separate API call needed on initial page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all rule operations triggered from the page.

Rule Operations
~~~~~~~~~~~~~~~

.. http:post:: /api/rules-engine/rules

   **Description**: Add new rule (when user clicks "Create New Rule" or finishes wizard).
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/rules HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "rule": {
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
          "trigger_count": 0,
          "last_triggered": "Never",
          "created_date": "2025-01-15",
          "last_modified": "2025-01-15",
          "selected_alerts": ["alert-001"]
        }
      }

.. http:put:: /api/rules-engine/rules/{rule_id}

   **Description**: Edit rule (when user edits an existing rule).
   
   **Path Parameters**:
   
   * **rule_id** (string): Rule identifier (e.g., "RULE-2025-001")
   
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
        "rule": {
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
          "trigger_count": 12,
          "last_triggered": "12m ago",
          "selected_alerts": ["alert-001", "alert-002"]
        }
      }

.. http:delete:: /api/rules-engine/rules/{rule_id}

   **Description**: Remove rule (when user clicks "Delete" on a rule).
   
   **Path Parameters**:
   
   * **rule_id** (string): Rule identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rule_id": "RULE-2025-001",
        "name": "Overtemp-Motor-Updated"
      }

Rule Status Operations
~~~~~~~~~~~~~~~~~~~~~~

.. http:patch:: /api/rules-engine/rules/{rule_id}/status

   **Description**: Toggle rule enabled status (UI toggle switch).
   
   **Path Parameters**:
   
   * **rule_id** (string): Rule identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      PATCH /api/rules-engine/rules/RULE-2025-001/status HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "enabled": false
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rule_id": "RULE-2025-001",
        "name": "Overtemp-Motor-Updated",
        "enabled": false,
        "previous_status": true,
        "status_changed": true
      }

Rule Template Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/rules-engine/templates

   **Description**: Get available rule templates for quick start (matching UI templates).
   
   **Success Response**:
   
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
        }
      ]

.. http:post:: /api/rules-engine/templates/{template_id}/create

   **Description**: Create a rule from template (when user clicks template in UI).
   
   **Path Parameters**:
   
   * **template_id** (string): Template identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/templates/temp-template/create HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "Custom Temperature Rule",
        "device": "Motor_02",
        "value": 75,
        "duration": 25
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "rule": {
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
          "trigger_count": 0,
          "last_triggered": "Never",
          "selected_alerts": ["alert-001"]
        }
      }

Rule Testing Operations
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/rules-engine/rules/test

   **Description**: Test rule configuration without saving (UI "Test Rule" button).
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/rules/test HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
        "simulation_result": "Rule would trigger 1 alert"
      }

.. http:post:: /api/rules-engine/rules/{rule_id}/test

   **Description**: Test an existing rule with specific values.
   
   **Path Parameters**:
   
   * **rule_id** (string): Rule identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/rules/RULE-2025-001/test HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "test_value": 90,
        "duration_met": 35
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
        ]
      }

Configuration Discovery Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/rules-engine/devices

   **Description**: Get available devices for dropdown (UI device selector).
   
   **Success Response**:
   
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
        }
      ]

.. http:get:: /api/rules-engine/tags

   **Description**: Get available tags for dropdown (UI tag selector).
   
   **Query Parameters**:
   
   * **device** (optional): Filter tags by device ID
   
   **Success Response**:
   
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
        }
      ]

.. http:get:: /api/rules-engine/alerts/available

   **Description**: Get available alerts for selection (UI alert selector).
   
   **Success Response**:
   
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
        }
      ]

Rule Import/Export Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/rules-engine/import

   **Description**: Import rules from JSON (UI import button).
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: multipart/form-data
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/import HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: multipart/form-data
      
      --boundary
      Content-Disposition: form-data; name="file"; filename="rules.json"
      Content-Type: application/json
      
      <JSON file content>
      --boundary--
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "imported": 1,
        "skipped": 0,
        "failed": 0,
        "details": {
          "rules_created": 1,
          "rules_updated": 0
        },
        "import_id": "IMP-20250115-001"
      }

.. http:post:: /api/rules-engine/export

   **Description**: Export rules to JSON file (when user exports rules).
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      Content-Disposition: attachment; filename="rules_export_20250115.json"
      
      {
        "rules": [
          {
            "id": "RULE-2025-001",
            "name": "Overtemp-Motor",
            "enabled": true,
            "type": "tag",
            "device": "Motor_Drive_01",
            "tag": "temperature_celsius",
            "condition": ">",
            "value": 80,
            "duration": 30,
            "selected_alerts": ["alert-001", "alert-002"]
          }
        ],
        "export_timestamp": "2025-01-15T10:30:00Z",
        "version": "1.0.0"
      }

Statistics Operations
~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/rules-engine/statistics

   **Description**: Get rule statistics for UI display.
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

.. http:get:: /api/rules-engine/rules/{rule_id}/statistics

   **Description**: Get statistics for a specific rule.
   
   **Path Parameters**:
   
   * **rule_id** (string): Rule identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "rule_id": "RULE-2025-001",
        "rule_name": "Overtemp-Motor",
        "trigger_count": 12,
        "last_triggered": "12m ago",
        "last_triggered_timestamp": "2025-01-15T10:18:00Z",
        "created_date": "2025-01-15",
        "status": "enabled"
      }

Validation Operations
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/rules-engine/validate

   **Description**: Validate rule configuration.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/validate HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "Test Rule",
        "type": "tag",
        "device": "Motor_Drive_01",
        "tag": "temperature_celsius",
        "condition": ">",
        "value": 80,
        "duration": 30
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "valid": true,
        "issues": [],
        "suggestions": [
          "Consider adding hysteresis to prevent rapid triggering"
        ],
        "logic_preview": "IF Motor_Drive_01.temperature_celsius > 80°C for 30s THEN trigger alerts"
      }

Utility Operations
~~~~~~~~~~~~~~~~~~

.. http:get:: /api/rules-engine/conditions

   **Description**: Get available condition operators.
   
   **Success Response**:
   
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

.. http:get:: /api/rules-engine/units

   **Description**: Get available units for tags.
   
   **Success Response**:
   
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

Bulk Operations
~~~~~~~~~~~~~~~

.. http:post:: /api/rules-engine/rules/bulk-delete

   **Description**: Delete multiple rules in a single request.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/rules/bulk-delete HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "ids": ["RULE-2025-001", "RULE-2025-002", "RULE-2025-003"]
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "deleted_count": 3,
        "failed_count": 0,
        "failed_ids": [],
        "errors": []
      }

.. http:post:: /api/rules-engine/rules/bulk-status

   **Description**: Update status for multiple rules.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/rules-engine/rules/bulk-status HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "ids": ["RULE-2025-001", "RULE-2025-002"],
        "enabled": false
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "updated_count": 2,
        "failed_count": 0,
        "failed_ids": []
      }

Route Summary
-------------

.. list-table:: Rules Engine Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/rules-engine`` - Main rules engine page
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/rules`` - Add rule
     - Yes
   * - API
     - PUT
     - ``/api/rules-engine/rules/{id}`` - Edit rule
     - Yes
   * - API
     - DELETE
     - ``/api/rules-engine/rules/{id}`` - Delete rule
     - Yes
   * - API
     - PATCH
     - ``/api/rules-engine/rules/{id}/status`` - Toggle rule status
     - Yes
   * - API
     - GET
     - ``/api/rules-engine/templates`` - Get templates
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/templates/{id}/create`` - Create from template
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/rules/test`` - Test rule config
     - Yes
   * - API
     - GET
     - ``/api/rules-engine/devices`` - Get devices
     - Yes
   * - API
     - GET
     - ``/api/rules-engine/tags`` - Get tags
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/import`` - Import rules
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/export`` - Export rules
     - Yes
   * - API
     - GET
     - ``/api/rules-engine/statistics`` - Get statistics
     - Yes
   * - API
     - POST
     - ``/api/rules-engine/validate`` - Validate rule
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /rules-engine``
   - Server renders HTML with embedded rules data
   - Page shows rules sidebar, empty editor wizard

2. **User creates new rule**:
   - Clicks "Create New Rule" → shows wizard Step 1 (templates)
   - Selects template → ``POST /api/rules-engine/templates/{id}/create``
   - Or builds from scratch → wizard progresses through steps

3. **User configures trigger** (Step 2):
   - Selects device, tag, condition, value, duration
   - UI updates logic preview in real-time

4. **User selects alerts** (Step 3):
   - Chooses alerts to trigger → ``GET /api/rules-engine/alerts/available``
   - Can test rule → ``POST /api/rules-engine/rules/test``
   - Saves rule → ``POST /api/rules-engine/rules``
   - New rule appears in sidebar

5. **User edits existing rule**:
   - Clicks rule in sidebar → loads into editor
   - Makes changes → ``PUT /api/rules-engine/rules/{id}``
   - Toggles status → ``PATCH /api/rules-engine/rules/{id}/status``

6. **User imports/export rules**:
   - Clicks "Import Rules" → ``POST /api/rules-engine/import``
   - Exports configuration → ``POST /api/rules-engine/export``

7. **User monitors statistics**:
   - Stats displayed in UI from ``GET /api/rules-engine/statistics``
   - Individual rule stats from ``GET /api/rules-engine/rules/{id}/statistics``

Error Codes
-----------

.. list-table:: Rules Engine Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - RULES_001
     - Rule not found
   * - RULES_002
     - Invalid rule configuration
   * - RULES_003
     - Rule already exists
   * - RULES_004
     - Cannot delete - rule in use
   * - RULES_005
     - Invalid condition for rule type
   * - RULES_006
     - Device not found
   * - RULES_007
     - Tag not found on device
   * - RULES_008
     - Alert not found
   * - RULES_009
     - Rule validation failed
   * - RULES_010
     - Import/export error
   * - RULES_011
     - Test execution failed

Key Features
------------

- **Wizard-based interface** - 3-step rule creation (Trigger → Conditions → Alerts)
- **Quick start templates** - Pre-configured rule templates
- **Real-time logic preview** - Shows rule logic as user configures
- **Device/tag discovery** - Dynamic dropdowns from device management
- **Alert integration** - Seamless connection with alert messages
- **Test functionality** - Test rules before saving
- **Import/export** - JSON-based configuration sharing
- **Statistics tracking** - Rule trigger counts and activity
- **Bulk operations** - Mass enable/disable/delete
- **Validation** - Comprehensive rule validation

Wizard Steps Detail
-------------------

**Step 1: Quick Start**
   - Template selection (temperature, device status, vibration, pressure, current, schedule)
   - Start from scratch
   - Copy existing rule

**Step 2: Trigger Configuration**
   - Rule type selection (tag, device, schedule)
   - Device selection (from available devices)
   - Tag selection (device-specific tags)
   - Condition operators (>, <, >=, <=, ==, !=)
   - Threshold value entry
   - Duration/hysteresis setting
   - Advanced options (scaled value, rising edge only)

**Step 3: Alert Selection**
   - Available alerts listing (from alert messages)
   - Multi-select for alerts to trigger
   - Alert status indication (enabled/disabled)
   - Test rule functionality
   - Final save

Data Types and Constraints
--------------------------

Rule Object Structure
~~~~~~~~~~~~~~~~~~~~~

.. list-table:: Rule Object Fields
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
~~~~~~~~~~~~~~~~

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

Template Structure
~~~~~~~~~~~~~~~~~~

.. list-table:: Template Object
   :widths: 25 75
   :header-rows: 1

   * - Field
     - Description
   * - **id**
     - Template identifier
   * - **name**
     - Template name
   * - **icon**
     - FontAwesome icon class
   * - **description**
     - Template description
   * - **device**
     - Default device
   * - **tag**
     - Default tag
   * - **condition**
     - Default condition
   * - **value**
     - Default value
   * - **duration**
     - Default duration
   * - **unit**
     - Default unit
   * - **default_alerts**
     - Default alerts to trigger

Integration Points
------------------

Device Management Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Device list**: Pulls from ``/api/device-management/devices``
- **Device tags**: Uses device-specific tags from device config
- **Device status**: Monitors device online/offline status

Alert Messages Integration
~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Alert list**: Pulls from ``/api/alert-messages/alert-messages``
- **Alert triggering**: Uses alert IDs to trigger notifications
- **Alert configuration**: Respects alert channel settings

Performance Considerations
--------------------------


Testing Strategy
----------------

1. **Unit Tests:**
   - Rule validation logic
   - Template creation
   - Condition evaluation

2. **Integration Tests:**
   - Device/Rule integration
   - Alert triggering
   - Import/export workflows

3. **UI Tests:**
   - Wizard navigation
   - Form validation
   - Real-time updates

4. **Performance Tests:**
   - Rule evaluation under load
   - Concurrent user access
   - Large rule set management


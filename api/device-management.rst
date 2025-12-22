Device Management API
=====================

This document describes the device management page and its related API endpoints for managing industrial devices, groups, and operations.

Page Route (Frontend)
---------------------

.. http:get:: /device-management

   **Description**: Renders the complete device management page with all devices, groups, and statistics embedded in the HTML.
   
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
        <title>Device Management</title>
      </head>
      <body>
        <div id="app" data-devices='[...]' data-groups='[...]' data-statistics='{...}'>
          <!-- Device management page with:
               - Device list table
               - Group management panel
               - Device actions (add, edit, delete, test)
               - Scanning interface
               - Import/export options
               - Wireless device pairing
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - All device data, groups, and statistics are embedded in the page
   - JavaScript reads this data and renders the device management interface
   - No separate API call needed on initial page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all device operations triggered from the page.

Device Operations
~~~~~~~~~~~~~~~~~

.. http:post:: /api/device-management/devices

   **Description**: Add new device (when user clicks "Add Device").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/devices HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "New Load Cell",
        "type": "modbus-rtu",
        "group_id": 1,
        "config": {
          "slave_address": 1,
          "baud_rate": 9600,
          "parity": "None",
          "stop_bits": 1,
          "polling_interval": 500
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "device": {
          "id": "device-123",
          "name": "New Load Cell",
          "type": "Modbus RTU",
          "address": "Slave 01",
          "status": "Online"
        }
      }

.. http:put:: /api/device-management/devices/{device_id}

   **Description**: Edit device configuration (when user clicks "Edit" on a device).
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
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
        "device": {
          "id": "loadcell-001",
          "name": "Updated Device",
          "type": "Modbus RTU"
        }
      }

.. http:delete:: /api/device-management/devices/{device_id}

   **Description**: Remove device (when user clicks "Delete" on a device).
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "device_id": "loadcell-001"
      }

Device Actions
~~~~~~~~~~~~~~

.. http:get:: /api/device-management/devices/{device_id}/details

   **Description**: Get detailed device information (when user clicks "View Details").
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "device_id": "loadcell-001",
        "name": "LoadCell-Front",
        "type": "Modbus RTU",
        "status": "Online",
        "last_response": "2 sec ago"
      }

.. http:post:: /api/device-management/devices/{device_id}/test

   **Description**: Ping device to test connectivity (when user clicks "Ping Device").
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "ping_time_ms": 2,
        "status": "Online"
      }

.. http:post:: /api/device-management/devices/{device_id}/disable

   **Description**: Disable or enable a device (when user toggles device status).
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/devices/loadcell-001/disable HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "disabled": true
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "status": "Disabled"
      }

.. http:post:: /api/device-management/devices/{device_id}/duplicate

   **Description**: Create a copy of device configuration (when user clicks "Duplicate").
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "new_device_id": "device-copy-123",
        "new_device": {
          "id": "device-copy-123",
          "name": "LoadCell-Front (Copy)",
          "type": "Modbus RTU"
        }
      }

Scanning Operations
~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/device-management/scan

   **Description**: Scan network for devices (when user clicks "Scan Networks").
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/scan HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "scan_type": "modbus-rtu",
        "auto_add": true
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-123",
        "devices_found": [
          {
            "address": "Slave 01",
            "type": "Modbus RTU"
          }
        ]
      }

.. http:get:: /api/device-management/scan/{scan_id}/status

   **Description**: Monitor scan progress (polled by JavaScript while scanning).
   
   **Path Parameters**:
   
   * **scan_id** (string): Scan job identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-123",
        "status": "completed",
        "progress": 100,
        "devices_found": 5
      }

.. http:post:: /api/device-management/wireless/scan

   **Description**: Scan for wireless devices (when user clicks "Scan" in Wireless section).
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "devices_found": [
          {
            "rf_address": "RF:0x09",
            "signal_strength": -70,
            "device_type": "ACS Sensor"
          }
        ]
      }

Group Management
~~~~~~~~~~~~~~~~

.. http:get:: /api/device-management/groups

   **Description**: Get all device groups (for dropdowns and group management).
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "groups": [
          {
            "id": 1,
            "name": "Crane-01",
            "device_count": 3,
            "color": "blue"
          }
        ]
      }

.. http:post:: /api/device-management/groups

   **Description**: Add new device group (when user clicks "Add New Group").
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/groups HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "Safety Sensors",
        "color": "red"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "group_id": 3,
        "group": {
          "id": 3,
          "name": "Safety Sensors",
          "device_count": 0,
          "color": "red"
        }
      }

.. http:put:: /api/device-management/groups/{group_id}

   **Description**: Edit device group.
   
   **Path Parameters**:
   
   * **group_id** (integer): Group identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Group updated"
      }

.. http:delete:: /api/device-management/groups/{group_id}

   **Description**: Remove device group.
   
   **Path Parameters**:
   
   * **group_id** (integer): Group identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Group deleted"
      }

.. http:post:: /api/device-management/groups/{group_id}/assign-devices

   **Description**: Assign multiple devices to a group (when user uses "Assign Devices" modal).
   
   **Path Parameters**:
   
   * **group_id** (integer): Group identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/groups/1/assign-devices HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "device_ids": ["device-123", "device-456"]
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "assigned_count": 2,
        "group_device_count": 5
      }

Import/Export Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/device-management/import

   **Description**: Import devices from CSV file (when user uploads CSV).
   
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
        "devices_added": 5
      }

.. http:post:: /api/device-management/export

   **Description**: Export devices to CSV file (when user clicks "Export to CSV").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="devices_export.csv"
      
      id,name,type,address,status
      loadcell-001,LoadCell-Front,Modbus RTU,Slave 01,Online

Wireless Operations
~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/device-management/wireless/pair

   **Description**: Pair with wireless device (when user clicks "Start Pairing").
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/device-management/wireless/pair HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "rf_address": "RF:0x09"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "pairing_code": "ABC123",
        "timeout_seconds": 30
      }

Diagnostics
~~~~~~~~~~~

.. http:get:: /api/device-management/devices/{device_id}/packets

   **Description**: View last 10 communication packets (when user clicks "View Last 10 Packets").
   
   **Path Parameters**:
   
   * **device_id** (string): Device identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "packets": [
          {
            "timestamp": "2025-03-12T10:02:01Z",
            "direction": "tx",
            "data_hex": "01 03 00 10 00 02 C4 0B"
          }
        ]
      }

Route Summary
-------------

.. list-table:: Device Management Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/device-management`` - Main device management page
     - Yes
   * - API
     - POST
     - ``/api/device-management/devices`` - Add device
     - Yes
   * - API
     - PUT
     - ``/api/device-management/devices/{id}`` - Edit device
     - Yes
   * - API
     - DELETE
     - ``/api/device-management/devices/{id}`` - Delete device
     - Yes
   * - API
     - GET
     - ``/api/device-management/devices/{id}/details`` - View details
     - Yes
   * - API
     - POST
     - ``/api/device-management/devices/{id}/test`` - Ping device
     - Yes
   * - API
     - POST
     - ``/api/device-management/scan`` - Scan for devices
     - Yes
   * - API
     - POST
     - ``/api/device-management/groups`` - Add group
     - Yes
   * - API
     - POST
     - ``/api/device-management/import`` - Import CSV
     - Yes
   * - API
     - POST
     - ``/api/device-management/export`` - Export CSV
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /device-management``
   - Server renders HTML with embedded device data
   - Page shows device table, groups panel, action buttons

2. **User adds device**: 
   - Clicks "Add Device" → opens modal
   - Fills form → ``POST /api/device-management/devices``
   - New device appears in table

3. **User scans for devices**:
   - Clicks "Scan Networks" → ``POST /api/device-management/scan``
   - Page polls ``GET /api/device-management/scan/{id}/status``
   - Shows found devices in results panel

4. **User manages groups**:
   - Clicks "Add Group" → ``POST /api/device-management/groups``
   - Drags devices to group → ``POST /api/device-management/groups/{id}/assign-devices``

5. **User exports data**:
   - Clicks "Export CSV" → ``POST /api/device-management/export``
   - Browser downloads CSV file

Error Codes
-----------

.. list-table:: Device Management Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - DEVICE_001
     - Device not found
   * - DEVICE_002
     - Invalid device configuration
   * - DEVICE_003
     - Device already exists
   * - DEVICE_004
     - Device communication timeout
   * - DEVICE_005
     - Group not found
   * - DEVICE_006
     - Invalid CSV format
   * - DEVICE_007
     - Scan already in progress
   * - DEVICE_008
     - Wireless pairing failed

Key Features
------------

- **Single page** with embedded initial data
- **Real-time updates** via WebSocket for device status
- **Batch operations** for device grouping
- **File operations** for import/export
- **Wireless support** for RF devices
- **Diagnostic tools** for troubleshooting
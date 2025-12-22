Tag Mapping API
===============

This document describes the tag mapping page and its related API endpoints for managing mappings between physical device addresses and logical tag names.

Page Route (Frontend)
---------------------

.. http:get:: /tag-mapping

   **Description**: Renders the complete tag mapping management page with all mappings, devices, and categories embedded in the HTML.
   
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
        <title>Tag Mapping - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-mappings='[...]' data-devices='[...]' data-categories='[...]'>
          <!-- Tag mapping page with:
               - Tag mapping table with search/filter
               - Tag browser grid view
               - Add/Edit/Delete mapping buttons
               - Import/export functionality
               - Protocol-specific configuration forms
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - All tag mappings, available devices, and categories are embedded in the page
   - JavaScript reads this data and renders the tag management interface
   - No separate API call needed on initial page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all tag mapping operations triggered from the page.

Tag Mapping CRUD Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/tag-mapping

   **Description**: Create new tag mapping (when user clicks "Add Mapping").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/tag-mapping HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "deviceId": 1,
        "deviceName": "LoadCell_1",
        "protocol": "Modbus RTU",
        "protocolConfig": {
          "registerType": "holding",
          "address": "40003",
          "registerCount": 1,
          "byteOrder": "12"
        },
        "tagName": "Hoist_Height",
        "dataType": "INT32",
        "scale": 0.01,
        "offset": 0,
        "unit": "meters",
        "pollInterval": 200,
        "category": "Position Tracking",
        "description": "Hoist height measurement"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "tag": {
          "id": 3,
          "deviceId": 1,
          "tagName": "Hoist_Height",
          "protocol": "Modbus RTU",
          "address": "40003",
          "dataType": "INT32",
          "unit": "meters"
        }
      }

.. http:put:: /api/tag-mapping/{id}

   **Description**: Update existing tag mapping (when user clicks "Edit" on a tag).
   
   **Path Parameters**:
   
   * **id** (integer): Tag mapping ID
   
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
        "message": "Tag mapping updated successfully",
        "tag": {
          "id": 3,
          "tagName": "Hoist_Height_Updated"
        }
      }

.. http:delete:: /api/tag-mapping/{id}

   **Description**: Delete tag mapping (when user clicks "Delete" on a tag).
   
   **Path Parameters**:
   
   * **id** (integer): Tag mapping ID
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Tag mapping deleted successfully",
        "deleted_id": 3
      }

.. http:get:: /api/tag-mapping/{id}

   **Description**: Get detailed information for a single tag mapping (when user clicks "View Details").
   
   **Path Parameters**:
   
   * **id** (integer): Tag mapping ID
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": 1,
        "deviceId": 1,
        "deviceName": "LoadCell_1",
        "protocol": "Modbus RTU",
        "tagName": "Hoist_Load",
        "dataType": "INT16",
        "unit": "tons",
        "lastValue": 145.2,
        "quality": "Good"
      }

Device and Protocol Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/tag-mapping/devices

   **Description**: Get list of all available devices for dropdown selection (populates device dropdown).
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "devices": [
          {
            "id": 1,
            "name": "LoadCell_1",
            "protocol": "Modbus RTU",
            "type": "Load Cell",
            "status": "online"
          }
        ],
        "total": 5
      }

.. http:get:: /api/tag-mapping/protocol-forms/{protocol}

   **Description**: Get form configuration for a specific protocol (when user selects a device).
   
   **Path Parameters**:
   
   * **protocol** (string): Protocol name (modbus-rtu, modbus-tcp, canbus, wireless-io, acs-sensor)
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "protocol": "Modbus RTU",
        "formFields": [
          {
            "name": "registerType",
            "label": "Register Type",
            "type": "select",
            "options": [
              {"value": "holding", "label": "Holding Register"}
            ]
          }
        ],
        "supportedDataTypes": ["INT16", "UINT16", "INT32"]
      }

Import/Export Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/tag-mapping/import-csv

   **Description**: Import tag mappings from CSV file (when user clicks "Import CSV").
   
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
        "imported": 15,
        "failed": 2,
        "total": 17
      }

.. http:get:: /api/tag-mapping/export-csv

   **Description**: Export tag mappings to CSV file (when user clicks "Export CSV").
   
   **Query Parameters**:
   
   * **category** (optional): Filter by category
   * **device_id** (optional): Filter by device ID
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="tag_mappings_export.csv"
      
      id,deviceName,tagName,protocol,address,dataType,unit,category
      1,LoadCell_1,Hoist_Load,Modbus RTU,40001,INT16,tons,Load Monitoring

Filtering and Search
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/tag-mapping/filter

   **Description**: Filter tag mappings (when user uses search/filter on the page).
   
   **Query Parameters**:
   
   * **category** (optional): Filter by category
   * **device_id** (optional): Filter by device ID
   * **protocol** (optional): Filter by protocol
   * **search** (optional): Search in tag names and descriptions
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "mappings": [
          {
            "id": 1,
            "tagName": "Hoist_Load",
            "deviceName": "LoadCell_1",
            "protocol": "Modbus RTU"
          }
        ],
        "count": 1
      }

Device Testing and Validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/tag-mapping/devices/{device_id}/test

   **Description**: Test device connection (when user clicks "Test Device" in footer).
   
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

.. http:post:: /api/tag-mapping/validate-all

   **Description**: Validate all tag mappings (when user clicks "Validate All" in footer).
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "validated_count": 25,
        "errors": [],
        "message": "All mappings validated successfully"
      }

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /api/tag-mapping/save-config

   **Description**: Save all tag mapping configuration (when user clicks "Save Configuration").
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Configuration saved successfully"
      }

Route Summary
-------------

.. list-table:: Tag Mapping Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/tag-mapping`` - Main tag mapping page
     - Yes
   * - API
     - POST
     - ``/api/tag-mapping`` - Create tag mapping
     - Yes
   * - API
     - PUT
     - ``/api/tag-mapping/{id}`` - Update tag mapping
     - Yes
   * - API
     - DELETE
     - ``/api/tag-mapping/{id}`` - Delete tag mapping
     - Yes
   * - API
     - GET
     - ``/api/tag-mapping/{id}`` - Get tag details
     - Yes
   * - API
     - GET
     - ``/api/tag-mapping/devices`` - Get available devices
     - Yes
   * - API
     - GET
     - ``/api/tag-mapping/protocol-forms/{protocol}`` - Get protocol form
     - Yes
   * - API
     - POST
     - ``/api/tag-mapping/import-csv`` - Import CSV
     - Yes
   * - API
     - GET
     - ``/api/tag-mapping/export-csv`` - Export CSV
     - Yes
   * - API
     - GET
     - ``/api/tag-mapping/filter`` - Filter mappings
     - Yes
   * - API
     - POST
     - ``/api/tag-mapping/devices/{id}/test`` - Test device
     - Yes
   * - API
     - POST
     - ``/api/tag-mapping/validate-all`` - Validate all
     - Yes
   * - API
     - PUT
     - ``/api/tag-mapping/save-config`` - Save config
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /tag-mapping``
   - Server renders HTML with embedded tag data
   - Page shows tag table, tag browser grid, action buttons

2. **User adds new tag**:
   - Clicks "Add Mapping" → opens 3-step modal
   - Selects device → dynamically loads protocol form via ``GET /api/tag-mapping/protocol-forms/{protocol}``
   - Fills form → ``POST /api/tag-mapping``
   - New tag appears in table and browser grid

3. **User edits tag**:
   - Clicks "Edit" on a tag → loads existing data
   - Makes changes → ``PUT /api/tag-mapping/{id}``
   - Tag updates in both table and grid

4. **User imports tags**:
   - Clicks "Import CSV" → selects file
   - Uploads → ``POST /api/tag-mapping/import-csv``
   - Shows import results

5. **User exports tags**:
   - Applies filters if needed
   - Clicks "Export CSV" → ``GET /api/tag-mapping/export-csv``
   - Browser downloads CSV file

6. **User searches/filters**:
   - Enters search text or selects filters
   - Page calls ``GET /api/tag-mapping/filter``
   - Updates table and grid with filtered results

7. **User tests device**:
   - Selects device from table
   - Clicks "Test Device" in footer → ``POST /api/tag-mapping/devices/{id}/test``
   - Shows connection status

8. **User saves configuration**:
   - Makes multiple changes
   - Clicks "Save Configuration" → ``PUT /api/tag-mapping/save-config``
   - Shows success message

Error Codes
-----------

.. list-table:: Tag Mapping Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - TAG_001
     - Tag name already exists
   * - TAG_002
     - Invalid device ID
   * - TAG_003
     - Unsupported data type for protocol
   * - TAG_004
     - Invalid address format
   * - TAG_005
     - Tag mapping not found
   * - TAG_006
     - CSV import format error
   * - TAG_007
     - Invalid scale/offset values

Key Features
------------

- **Single page** with embedded initial data
- **Two views**: Table view for management, Grid view for browsing
- **Dynamic forms** based on protocol selection
- **Real-time validation** of tag names and addresses
- **Bulk operations** via CSV import/export
- **Advanced filtering** and search capabilities
- **Protocol-specific configuration** for each device type

Data Types and Protocols
------------------------

### Supported Data Types

.. list-table:: Supported Data Types
   :widths: 30 30 40
   :header-rows: 1

   * - Data Type
     - Size (bits)
     - Description
   * - **BOOL**
     - 1
     - Boolean (true/false)
   * - **INT16**
     - 16
     - Signed 16-bit integer
   * - **UINT16**
     - 16
     - Unsigned 16-bit integer
   * - **INT32**
     - 32
     - Signed 32-bit integer
   * - **FLOAT32**
     - 32
     - IEEE 754 single precision float

### Supported Protocols

.. list-table:: Supported Protocols
   :widths: 30 70
   :header-rows: 1

   * - Protocol
     - Key Configuration
   * - **Modbus RTU**
     - registerType, address, registerCount
   * - **Modbus TCP**
     - registerType, address, registerCount
   * - **CANBus**
     - canId, dataLength, byteOrder
   * - **Wireless IO**
     - channel, nodeId, dataFormat
   * - **ACS Sensor**
     - sensorId, parameterId

Validation Rules
----------------

1. **Tag Names**: 
   - Must be unique
   - Alphanumeric and underscore only
   - Max 50 characters

2. **Addresses**:
   - Protocol-specific validation
   - Must be within device address space

3. **Poll Interval**:
   - Min: 50ms
   - Max: 60000ms (1 minute)
   - Default: 1000ms
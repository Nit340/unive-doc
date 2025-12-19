Tag Mapping API
===============

APIs for managing tag mappings between devices and data points.

Overview
--------

The Tag Mapping API allows you to create, read, update, and delete mappings between physical device addresses and logical tag names. This enables data normalization and abstraction across different protocols and devices.

Endpoints
---------

Get All Tag Mappings
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /tag-mapping

   Retrieve all existing tag mappings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **category** (optional): Filter by category
   * **device_id** (optional): Filter by device ID
   * **protocol** (optional): Filter by protocol

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "mappings": [
          {
            "id": 1,
            "deviceId": 1,
            "deviceName": "LoadCell_1",
            "protocol": "Modbus RTU",
            "address": "40001",
            "tagName": "Hoist_Load",
            "dataType": "INT16",
            "scale": 0.01,
            "offset": 0,
            "unit": "tons",
            "pollInterval": 200,
            "category": "Load Monitoring",
            "description": "Main hoist load measurement",
            "minValid": 0,
            "maxValid": 200,
            "endianness": "big-endian"
          },
          {
            "id": 2,
            "deviceId": 2,
            "deviceName": "Inclin_Arm",
            "protocol": "Modbus TCP",
            "address": "40002",
            "tagName": "Boom_Angle",
            "dataType": "FLOAT32",
            "scale": 0.1,
            "offset": 0,
            "unit": "degrees",
            "pollInterval": 500,
            "category": "Position Tracking",
            "description": "Boom angle measurement",
            "minValid": -10,
            "maxValid": 85,
            "endianness": "little-endian"
          }
        ],
        "count": 2,
        "categories": ["Load Monitoring", "Position Tracking"]
      }

Get Available Devices
~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /tag-mapping/devices

   Get list of all devices for dropdown selection.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

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
            "status": "online",
            "address": "01",
            "description": "Main hoist load cell"
          },
          {
            "id": 2,
            "name": "Inclin_Arm",
            "protocol": "Modbus TCP",
            "type": "Inclinometer",
            "status": "online",
            "address": "192.168.1.21:502",
            "description": "Boom angle sensor"
          },
          {
            "id": 3,
            "name": "HoistDrive",
            "protocol": "CANBus",
            "type": "Drive Controller",
            "status": "offline",
            "address": "0x32",
            "description": "Main hoist drive"
          },
          {
            "id": 4,
            "name": "WindSensor",
            "protocol": "Wireless IO",
            "type": "Wind Sensor",
            "status": "online",
            "address": "CH1",
            "description": "Wind speed sensor"
          },
          {
            "id": 5,
            "name": "ACS_Sensor_1",
            "protocol": "ACS Sensor",
            "type": "ACS Sensor",
            "status": "online",
            "address": "01",
            "description": "ACS sensor device"
          }
        ],
        "total": 5,
        "online": 4,
        "protocols": ["Modbus RTU", "Modbus TCP", "CANBus", "Wireless IO", "ACS Sensor"]
      }

Get Protocol Form Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /tag-mapping/protocol-forms/{protocol}

   Get form configuration for a specific protocol.

   **Path Parameters**:

   * **protocol** (string): Protocol name (modbus-rtu, modbus-tcp, canbus, wireless-io, acs-sensor)

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response** (Modbus RTU Example):

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
            "required": true,
            "options": [
              {"value": "holding", "label": "Holding Register (4x)"},
              {"value": "input", "label": "Input Register (3x)"},
              {"value": "coil", "label": "Coil (0x)"},
              {"value": "discrete", "label": "Discrete Input (1x)"}
            ],
            "default": "holding"
          },
          {
            "name": "address",
            "label": "Register Address",
            "type": "text",
            "required": true,
            "placeholder": "e.g., 40001",
            "validation": "^[0-9]+$",
            "default": "40001"
          },
          {
            "name": "registerCount",
            "label": "Register Count",
            "type": "number",
            "required": true,
            "min": 1,
            "max": 125,
            "default": 1
          },
          {
            "name": "byteOrder",
            "label": "Byte Order",
            "type": "select",
            "required": true,
            "options": [
              {"value": "12", "label": "Big Endian (1-2)"},
              {"value": "21", "label": "Little Endian (2-1)"}
            ],
            "default": "12"
          }
        ],
        "defaultDataType": "INT16",
        "supportedDataTypes": ["INT16", "UINT16", "INT32", "UINT32", "FLOAT32", "BOOL"]
      }

Create Tag Mapping
~~~~~~~~~~~~~~~~~~

.. http:post:: /tag-mapping

   Create new tag mapping.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /tag-mapping HTTP/1.1
      Authorization: Bearer <token>
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
        "description": "Hoist height measurement",
        "minValid": 0,
        "maxValid": 100,
        "endianness": "big-endian"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "tag": {
          "id": 3,
          "deviceId": 1,
          "deviceName": "LoadCell_1",
          "protocol": "Modbus RTU",
          "address": "40003",
          "tagName": "Hoist_Height",
          "dataType": "INT32",
          "scale": 0.01,
          "offset": 0,
          "unit": "meters",
          "pollInterval": 200,
          "category": "Position Tracking",
          "description": "Hoist height measurement",
          "minValid": 0,
          "maxValid": 100,
          "endianness": "big-endian"
        }
      }

Update Tag Mapping
~~~~~~~~~~~~~~~~~~

.. http:put:: /tag-mapping/{id}

   Update existing tag mapping.

   **Path Parameters**:

   * **id** (integer): Tag mapping ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /tag-mapping/3 HTTP/1.1
      Authorization: Bearer <token>
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
        "tagName": "Hoist_Height_Updated",
        "dataType": "INT32",
        "scale": 0.01,
        "offset": 0,
        "unit": "meters",
        "pollInterval": 250,
        "category": "Position Tracking",
        "description": "Updated hoist height measurement",
        "minValid": 0,
        "maxValid": 150,
        "endianness": "big-endian"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Tag mapping updated successfully",
        "tag": {
          "id": 3,
          "tagName": "Hoist_Height_Updated",
          "pollInterval": 250,
          "description": "Updated hoist height measurement",
          "maxValid": 150
        }
      }

Delete Tag Mapping
~~~~~~~~~~~~~~~~~~

.. http:delete:: /tag-mapping/{id}

   Delete tag mapping.

   **Path Parameters**:

   * **id** (integer): Tag mapping ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Tag mapping deleted successfully",
        "deleted_id": 3
      }

Import Mappings from CSV
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /tag-mapping/import-csv

   Import tag mappings from CSV file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /tag-mapping/import-csv HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="csvFile"; filename="mappings.csv"
      Content-Type: text/csv

   **Query Parameters**:

   * **overwrite** (optional): Overwrite existing mappings (default: false)
   * **skipErrors** (optional): Skip rows with errors (default: true)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "imported": 15,
        "failed": 2,
        "total": 17,
        "errors": [
          {
            "row": 3,
            "error": "Invalid data type: UNKNOWN",
            "field": "dataType",
            "value": "UNKNOWN"
          },
          {
            "row": 7,
            "error": "Device not found",
            "field": "deviceId",
            "value": "999"
          }
        ],
        "imported_ids": [101, 102, 103, 104, 105]
      }

Export Mappings to CSV
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /tag-mapping/export-csv

   Export all tag mappings to CSV file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **include_config** (optional): Include protocol configuration (default: false)
   * **category** (optional): Filter by category
   * **device_id** (optional): Filter by device ID

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="tag_mappings_export_20250312.csv"
      
      id,deviceName,tagName,protocol,address,dataType,unit,category
      1,LoadCell_1,Hoist_Load,Modbus RTU,40001,INT16,tons,Load Monitoring
      2,Inclin_Arm,Boom_Angle,Modbus TCP,40002,FLOAT32,degrees,Position Tracking

Get Single Tag Mapping
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /tag-mapping/{id}

   Get detailed information for a single tag mapping.

   **Path Parameters**:

   * **id** (integer): Tag mapping ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": 1,
        "deviceId": 1,
        "deviceName": "LoadCell_1",
        "protocol": "Modbus RTU",
        "protocolConfig": {
          "registerType": "holding",
          "address": "40001",
          "registerCount": 1,
          "byteOrder": "12",
          "slaveId": 1,
          "functionCode": 3
        },
        "tagName": "Hoist_Load",
        "dataType": "INT16",
        "scale": 0.01,
        "offset": 0,
        "unit": "tons",
        "pollInterval": 200,
        "category": "Load Monitoring",
        "description": "Main hoist load measurement",
        "minValid": 0,
        "maxValid": 200,
        "endianness": "big-endian",
        "createdAt": "2025-03-12T10:30:00Z",
        "updatedAt": "2025-03-12T10:30:00Z",
        "lastValue": 145.2,
        "lastTimestamp": "2025-03-12T10:29:58Z",
        "quality": "Good"
      }

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
   * - **INT8**
     - 8
     - Signed 8-bit integer
   * - **UINT8**
     - 8
     - Unsigned 8-bit integer
   * - **INT16**
     - 16
     - Signed 16-bit integer
   * - **UINT16**
     - 16
     - Unsigned 16-bit integer
   * - **INT32**
     - 32
     - Signed 32-bit integer
   * - **UINT32**
     - 32
     - Unsigned 32-bit integer
   * - **FLOAT32**
     - 32
     - IEEE 754 single precision float
   * - **FLOAT64**
     - 64
     - IEEE 754 double precision float
   * - **STRING**
     - Variable
     - Null-terminated string

### Supported Protocols

.. list-table:: Supported Protocols
   :widths: 30 70
   :header-rows: 1

   * - Protocol
     - Configuration Fields
   * - **Modbus RTU**
     - registerType, address, registerCount, byteOrder, slaveId, baudRate, parity
   * - **Modbus TCP**
     - registerType, address, registerCount, byteOrder, ipAddress, port, unitId
   * - **CANBus**
     - canId, canFormat (standard/extended), dataLength, byteOrder
   * - **Wireless IO**
     - channel, nodeId, dataFormat, samplingRate
   * - **ACS Sensor**
     - sensorId, parameterId, samplingRate

Error Codes
-----------

.. list-table:: Tag Mapping Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - **TAG_001**
     - Tag name already exists
   * - **TAG_002**
     - Invalid device ID
   * - **TAG_003**
     - Unsupported data type for protocol
   * - **TAG_004**
     - Invalid address format
   * - **TAG_005**
     - Tag mapping not found
   * - **TAG_006**
     - CSV import format error
   * - **TAG_007**
     - Invalid scale/offset values
   * - **TAG_008**
     - Validation range error (min > max)

Examples
--------

Python - Create Tag Mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Get devices for dropdown
   headers = {"Authorization": "Bearer your_token"}
   devices_response = requests.get(
       "https://api.example.com/v1/tag-mapping/devices",
       headers=headers
   )
   
   devices = devices_response.json()["devices"]
   print(f"Available devices: {len(devices)}")
   
   # Get protocol form for Modbus RTU
   protocol_response = requests.get(
       "https://api.example.com/v1/tag-mapping/protocol-forms/modbus-rtu",
       headers=headers
   )
   
   protocol_config = protocol_response.json()
   print(f"Protocol: {protocol_config['protocol']}")
   print(f"Supported data types: {protocol_config['supportedDataTypes']}")
   
   # Create new tag mapping
   new_tag = {
       "deviceId": 1,
       "deviceName": "LoadCell_1",
       "protocol": "Modbus RTU",
       "protocolConfig": {
           "registerType": "holding",
           "address": "40010",
           "registerCount": 2,
           "byteOrder": "12"
       },
       "tagName": "Temperature_Cell",
       "dataType": "FLOAT32",
       "scale": 0.1,
       "offset": -10,
       "unit": "°C",
       "pollInterval": 1000,
       "category": "Temperature",
       "description": "Load cell temperature sensor",
       "minValid": -20,
       "maxValid": 80,
       "endianness": "big-endian"
   }
   
   create_response = requests.post(
       "https://api.example.com/v1/tag-mapping",
       json=new_tag,
       headers=headers
   )
   
   if create_response.status_code == 201:
       tag = create_response.json()["tag"]
       print(f"Created tag: {tag['tagName']} (ID: {tag['id']})")

Python - Import from CSV
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Import tag mappings from CSV
   import csv
   import io
   
   # Create sample CSV data
   csv_data = """deviceId,tagName,protocol,address,dataType,unit,category
   1,Temp_Sensor,Modbus RTU,40010,FLOAT32,°C,Temperature
   2,Pressure_1,Modbus TCP,40020,FLOAT32,bar,Pressure"""
   
   # Convert to file-like object
   csv_file = io.BytesIO(csv_data.encode('utf-8'))
   
   # Prepare multipart form data
   files = {'csvFile': ('mappings.csv', csv_file, 'text/csv')}
   
   # Send import request
   import_response = requests.post(
       "https://api.example.com/v1/tag-mapping/import-csv",
       files=files,
       headers={"Authorization": "Bearer your_token"}
   )
   
   result = import_response.json()
   print(f"Imported: {result['imported']}, Failed: {result['failed']}")
   
   if result['errors']:
       print("Errors:")
       for error in result['errors']:
           print(f"  Row {error['row']}: {error['error']}")

JavaScript - Dynamic Form Generation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Get devices and populate dropdown
   async function loadDevices() {
       const token = localStorage.getItem('token');
       const response = await fetch('/tag-mapping/devices', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       const select = document.getElementById('device-select');
       
       data.devices.forEach(device => {
           const option = document.createElement('option');
           option.value = device.id;
           option.textContent = `${device.name} (${device.protocol})`;
           select.appendChild(option);
       });
       
       // Add change listener
       select.addEventListener('change', async (event) => {
           const deviceId = event.target.value;
           const device = data.devices.find(d => d.id == deviceId);
           
           if (device) {
               await loadProtocolForm(device.protocol);
           }
       });
   }
   
   // Load protocol-specific form
   async function loadProtocolForm(protocol) {
       const token = localStorage.getItem('token');
       const protocolKey = protocol.toLowerCase().replace(/\s+/g, '-');
       
       const response = await fetch(`/tag-mapping/protocol-forms/${protocolKey}`, {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const formConfig = await response.json();
       renderProtocolForm(formConfig);
   }
   
   // Render dynamic form
   function renderProtocolForm(formConfig) {
       const formContainer = document.getElementById('protocol-form');
       formContainer.innerHTML = '';
       
       formConfig.formFields.forEach(field => {
           const div = document.createElement('div');
           div.className = 'form-group';
           
           const label = document.createElement('label');
           label.textContent = field.label;
           label.htmlFor = field.name;
           
           let input;
           
           if (field.type === 'select') {
               input = document.createElement('select');
               input.id = field.name;
               input.name = field.name;
               input.required = field.required;
               
               field.options.forEach(option => {
                   const opt = document.createElement('option');
                   opt.value = option.value;
                   opt.textContent = option.label;
                   if (option.value === field.default) {
                       opt.selected = true;
                   }
                   input.appendChild(opt);
               });
           } else {
               input = document.createElement('input');
               input.type = field.type;
               input.id = field.name;
               input.name = field.name;
               input.required = field.required;
               
               if (field.placeholder) {
                   input.placeholder = field.placeholder;
               }
               
               if (field.default) {
                   input.value = field.default;
               }
           }
           
           div.appendChild(label);
           div.appendChild(input);
           formContainer.appendChild(div);
       });
   }

JavaScript - Create Tag Mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Create new tag mapping
   async function createTagMapping() {
       const token = localStorage.getItem('token');
       
       // Collect form data
       const formData = {
           deviceId: document.getElementById('device-select').value,
           deviceName: document.getElementById('device-select').selectedOptions[0].text,
           protocol: document.getElementById('protocol').value,
           protocolConfig: {},
           tagName: document.getElementById('tag-name').value,
           dataType: document.getElementById('data-type').value,
           scale: parseFloat(document.getElementById('scale').value),
           offset: parseFloat(document.getElementById('offset').value),
           unit: document.getElementById('unit').value,
           pollInterval: parseInt(document.getElementById('poll-interval').value),
           category: document.getElementById('category').value,
           description: document.getElementById('description').value,
           minValid: parseFloat(document.getElementById('min-valid').value),
           maxValid: parseFloat(document.getElementById('max-valid').value)
       };
       
       // Collect protocol-specific config
       const protocolForm = document.getElementById('protocol-form');
       const inputs = protocolForm.querySelectorAll('input, select');
       
       inputs.forEach(input => {
           if (input.name) {
               formData.protocolConfig[input.name] = input.value;
           }
       });
       
       // Send request
       const response = await fetch('/tag-mapping', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify(formData)
       });
       
       const result = await response.json();
       
       if (result.success) {
           alert(`Tag "${result.tag.tagName}" created successfully!`);
           // Refresh the tag list
           loadTagMappings();
       } else {
           alert(`Error: ${result.error}`);
       }
   }

Workflow Example
----------------

### Complete Tag Creation Flow

1. **Load Tag Mappings Page**
   
   .. code-block:: javascript
      
      // Initial page load
      async function loadPage() {
          await loadTagMappings();
          await loadDevices();
      }

2. **User Clicks "Add New Tag"**
   
   .. code-block:: javascript
      
      function showAddTagModal() {
          // Show modal with device dropdown
          document.getElementById('add-tag-modal').style.display = 'block';
      }

3. **User Selects Device**
   
   .. code-block:: javascript
      
      async function onDeviceSelect(deviceId) {
          const device = getDeviceById(deviceId);
          const protocolForm = await getProtocolForm(device.protocol);
          
          // Show protocol-specific form
          renderProtocolForm(protocolForm);
          
          // Set default data types
          populateDataTypes(protocolForm.supportedDataTypes);
      }

4. **User Fills Form and Submits**
   
   .. code-block:: javascript
      
      async function submitTagForm() {
          const tagData = collectFormData();
          
          try {
              const result = await createTagMapping(tagData);
              
              if (result.success) {
                  // Close modal, refresh list
                  closeModal();
                  await loadTagMappings();
              }
          } catch (error) {
              showError(error.message);
          }
      }

Notes
-----

### Validation Rules

1. **Tag Name**
   - Must be unique across all devices
   - Only alphanumeric and underscore allowed
   - Maximum 50 characters

2. **Address Validation**
   - Protocol-specific validation
   - Modbus: 0-65535 for addresses
   - CANBus: Hex format (0x000-0x7FF or 0x00000000-0x1FFFFFFF)

3. **Poll Interval**
   - Minimum: 50ms
   - Maximum: 60000ms (1 minute)
   - Default: 1000ms

4. **Scale/Offset**
   - Scale cannot be 0
   - Offset can be negative

### Performance Considerations

- **Maximum tags per device**: 1000
- **Total tags per gateway**: 10,000
- **Polling overhead**: Consider total polling frequency
- **Memory usage**: Each tag consumes ~1KB of memory

### Best Practices

1. **Naming Convention**
   - Use descriptive tag names
   - Include units in tag names (e.g., "Temperature_C")
   - Group related tags with similar prefixes

2. **Category Organization**
   - Use consistent categories
   - Limit to 10-15 categories maximum
   - Use subcategories if needed

3. **Validation Ranges**
   - Set realistic min/max values
   - Include safety margins
   - Update ranges based on operational data

Troubleshooting
---------------

### Common Issues

1. **"Tag name already exists"**
   - Check for duplicate names
   - Use search to find existing tags

2. **"Invalid address"**
   - Verify protocol-specific address format
   - Check device address space limitations

3. **"Device not responding"**
   - Verify device is online
   - Check network connectivity
   - Verify protocol configuration

4. **CSV Import Errors**
   - Check CSV format and encoding
   - Verify required columns are present
   - Check data type compatibility

### Debug Tips

1. Enable debug logging for tag operations
2. Test with single tag before bulk import
3. Verify device communication before mapping
4. Use the "Test Connection" feature if available
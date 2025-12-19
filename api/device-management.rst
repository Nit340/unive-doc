Device Management API
=====================

APIs for managing devices, groups, and device operations.

Overview
--------

This API handles all device management operations including device CRUD, scanning, grouping, and device diagnostics.

Endpoints
---------

Get Device Management Page
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /device-management

   Load entire device management page with devices, groups, and statistics.

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
            "id": "loadcell-001",
            "name": "LoadCell-Front",
            "type": "Modbus RTU",
            "address": "Slave 01",
            "status": "Online",
            "last_poll": "2 sec ago",
            "firmware": "1.2.4",
            "group_id": 1,
            "group_name": "Crane-01"
          }
        ],
        "groups": [
          {
            "id": 1,
            "name": "Crane-01",
            "device_count": 3,
            "color": "blue",
            "device_ids": ["loadcell-001"]
          }
        ],
        "statistics": {
          "total_devices": 5,
          "online": 4,
          "offline": 1
        }
      }

Create New Device
~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/devices

   Add new device (Add Device button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/devices HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "New Device",
        "type": "modbus-rtu",
        "group_id": 1,
        "config": {
          "slave_address": 1,
          "baud_rate": 9600,
          "parity": "None",
          "stop_bits": 1,
          "polling_interval": 500,
          "timeout": 200
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "device": {
          "id": "device-123",
          "name": "New Device",
          "type": "Modbus RTU",
          "address": "Slave 01",
          "status": "Online",
          "last_poll": "Just now",
          "group_id": 1
        }
      }

Update Device
~~~~~~~~~~~~~

.. http:put:: /device-management/devices/{device_id}

   Edit device configuration (Edit Device button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /device-management/devices/loadcell-001 HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Updated Device",
        "type": "modbus-rtu",
        "group_id": 2,
        "config": {
          "slave_address": 2,
          "baud_rate": 19200
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "device": {
          "id": "loadcell-001",
          "name": "Updated Device",
          "type": "Modbus RTU",
          "address": "Slave 02",
          "status": "Online"
        }
      }

Delete Device
~~~~~~~~~~~~~

.. http:delete:: /device-management/devices/{device_id}

   Remove device (Delete Device button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "device_id": "loadcell-001"
      }

Get Device Details
~~~~~~~~~~~~~~~~~~

.. http:get:: /device-management/devices/{device_id}/details

   View detailed device information (View Details button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "device_id": "loadcell-001",
        "name": "LoadCell-Front",
        "type": "Modbus RTU",
        "address": "Slave 01",
        "status": "Online",
        "last_response": "2 sec ago",
        "retries": 0,
        "signal_strength": "N/A (Wired)",
        "firmware_version": "1.2.4",
        "group": "Crane-01",
        "calibration": {
          "scale_factor": 1.000,
          "zero_offset": 0.00,
          "test_reading": "2.34 tons"
        }
      }

Test Device Connection
~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/devices/{device_id}/test

   Ping device to test connectivity (Ping Device button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "ping_time_ms": 2,
        "status": "Online"
      }

Disable/Enable Device
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/devices/{device_id}/disable

   Disable or enable a device (Disable Device button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/devices/loadcell-001/disable HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "disabled": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "status": "Disabled"
      }

Duplicate Device
~~~~~~~~~~~~~~~~

.. http:post:: /device-management/devices/{device_id}/duplicate

   Create a copy of device configuration (Duplicate button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "new_device_id": "device-copy-123",
        "new_device": {
          "id": "device-copy-123",
          "name": "LoadCell-Front (Copy)",
          "type": "Modbus RTU",
          "address": "Slave 02",
          "status": "Online"
        }
      }

Scan for Devices
~~~~~~~~~~~~~~~~

.. http:post:: /device-management/scan

   Scan network for devices (Scan Networks button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/scan HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "scan_type": "modbus-rtu",
        "auto_add": true,
        "overwrite": false,
        "scan_range": "Default"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-123",
        "devices_found": [
          {
            "address": "Slave 01",
            "type": "Modbus RTU",
            "name_suggestion": "Load Cell"
          }
        ]
      }

Check Scan Status
~~~~~~~~~~~~~~~~~

.. http:get:: /device-management/scan/{scan_id}/status

   Monitor scan progress.

   **Path Parameters**:

   * **scan_id** (string): Scan job identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "scan_id": "scan-123",
        "status": "completed",
        "progress": 100,
        "devices_found": 5,
        "devices_added": 3
      }

Scan Wireless Devices
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/wireless/scan

   Scan for wireless devices (Scan button in Wireless config).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

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

Get Device Groups
~~~~~~~~~~~~~~~~~

.. http:get:: /device-management/groups

   Retrieve all device groups.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

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

Create Device Group
~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/groups

   Add new device group (Add New Group button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/groups HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Safety Sensors",
        "color": "red"
      }

   **Response**:

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

Update Device Group
~~~~~~~~~~~~~~~~~~~

.. http:put:: /device-management/groups/{group_id}

   Edit device group.

   **Path Parameters**:

   * **group_id** (integer): Group identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /device-management/groups/1 HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Crane-01 Updated",
        "color": "darkblue"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Group updated"
      }

Delete Device Group
~~~~~~~~~~~~~~~~~~~

.. http:delete:: /device-management/groups/{group_id}

   Remove device group.

   **Path Parameters**:

   * **group_id** (integer): Group identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Group deleted"
      }

Assign Devices to Group
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/groups/{group_id}/assign-devices

   Assign multiple devices to a group (Assign Devices modal).

   **Path Parameters**:

   * **group_id** (integer): Group identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/groups/1/assign-devices HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "device_ids": ["device-123", "device-456"]
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "assigned_count": 2,
        "group_device_count": 5
      }

Import Devices from CSV
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/import

   Import devices from CSV file.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: multipart/form-data

   **Request**:

   .. sourcecode:: http

      POST /device-management/import HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: multipart/form-data
      Content-Disposition: form-data; name="file"; filename="devices.csv"
      Content-Type: text/csv

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "devices_added": 5,
        "imported_devices": [
          {
            "id": "device-123",
            "name": "Imported Device",
            "type": "Modbus RTU"
          }
        ]
      }

Export Devices to CSV
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/export

   Export devices to CSV file (Export to CSV button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/export HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "include_config": true,
        "include_timestamp": true
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: text/csv
      Content-Disposition: attachment; filename="devices_export_20250312.csv"
      
      id,name,type,address,status
      loadcell-001,LoadCell-Front,Modbus RTU,Slave 01,Online
      ...

Pair Wireless Device
~~~~~~~~~~~~~~~~~~~~

.. http:post:: /device-management/wireless/pair

   Pair with wireless device (Start Pairing button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /device-management/wireless/pair HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "rf_address": "RF:0x09"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "pairing_code": "ABC123",
        "timeout_seconds": 30
      }

Get Device Packets
~~~~~~~~~~~~~~~~~~

.. http:get:: /device-management/devices/{device_id}/packets

   View last 10 communication packets (View Last 10 Packets button).

   **Path Parameters**:

   * **device_id** (string): Device identifier

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "packets": [
          {
            "timestamp": "2025-03-12T10:02:01Z",
            "direction": "tx",
            "data_hex": "01 03 00 10 00 02 C4 0B"
          },
          {
            "timestamp": "2025-03-12T10:02:01Z",
            "direction": "rx",
            "data_hex": "01 03 04 00 12 01 1A 69 85"
          }
        ]
      }

Error Codes
-----------

.. list-table:: Device Management Errors
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

Examples
--------

Python - Create Device
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Create new Modbus device
   headers = {"Authorization": "Bearer your_token"}
   
   device_config = {
       "name": "New Load Cell",
       "type": "modbus-rtu",
       "group_id": 1,
       "config": {
           "slave_address": 3,
           "baud_rate": 9600,
           "parity": "None",
           "stop_bits": 1,
           "polling_interval": 1000
       }
   }
   
   response = requests.post(
       "https://api.example.com/v1/device-management/devices",
       json=device_config,
       headers=headers
   )
   
   if response.status_code == 201:
       device = response.json()["device"]
       print(f"Created device: {device['id']}")

Python - Scan for Devices
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Start device scan
   scan_config = {
       "scan_type": "modbus-rtu",
       "auto_add": True,
       "overwrite": False
   }
   
   response = requests.post(
       "https://api.example.com/v1/device-management/scan",
       json=scan_config,
       headers=headers
   )
   
   scan_id = response.json()["scan_id"]
   print(f"Scan started: {scan_id}")
   
   # Check scan status
   import time
   while True:
       status_response = requests.get(
           f"https://api.example.com/v1/device-management/scan/{scan_id}/status",
           headers=headers
       )
       
       status = status_response.json()
       print(f"Progress: {status['progress']}%")
       
       if status["status"] == "completed":
           print(f"Found {status['devices_found']} devices")
           break
           
       time.sleep(2)

JavaScript - Get Device Details
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Get device details
   async function getDeviceDetails(deviceId) {
       const token = localStorage.getItem('token');
       
       const response = await fetch(`/device-management/devices/${deviceId}/details`, {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       console.log('Device details:', data);
       
       // Display device info
       document.getElementById('device-name').textContent = data.name;
       document.getElementById('device-status').textContent = data.status;
       document.getElementById('device-type').textContent = data.type;
       
       return data;
   }
   
   // Test device connection
   async function pingDevice(deviceId) {
       const token = localStorage.getItem('token');
       
       const response = await fetch(`/device-management/devices/${deviceId}/test`, {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const result = await response.json();
       
       if (result.success) {
           alert(`Ping successful! Response time: ${result.ping_time_ms}ms`);
       } else {
           alert('Device is offline or unreachable');
       }
       
       return result;
   }

JavaScript - Manage Device Groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Create new group
   async function createGroup(groupName, color) {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/device-management/groups', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               name: groupName,
               color: color
           })
       });
       
       const result = await response.json();
       
       if (result.success) {
           console.log(`Created group: ${result.group.name} (ID: ${result.group_id})`);
           return result.group;
       }
   }
   
   // Assign devices to group
   async function assignToGroup(groupId, deviceIds) {
       const token = localStorage.getItem('token');
       
       const response = await fetch(`/device-management/groups/${groupId}/assign-devices`, {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               device_ids: deviceIds
           })
       });
       
       const result = await response.json();
       
       if (result.success) {
           console.log(`Assigned ${result.assigned_count} devices to group`);
       }
       
       return result;
   }

Notes
-----

- **Device Status**: Online, Offline, Disabled, Error
- **Device Types**: Modbus RTU, Modbus TCP, Wireless RF, Ethernet
- **Scan Types**: modbus-rtu, modbus-tcp, wireless, auto
- **File Upload**: CSV import supports .csv files with specific columns
- **Real-time Updates**: Consider WebSocket for real-time device status updates

Troubleshooting
---------------

1. **Device Not Responding**:
   - Check physical connection
   - Verify device address
   - Test with ping function

2. **Scan Not Finding Devices**:
   - Verify network configuration
   - Check firewall settings
   - Ensure correct scan type

3. **Import/Export Issues**:
   - Verify CSV format
   - Check file encoding (UTF-8 recommended)
   - Ensure required columns are present
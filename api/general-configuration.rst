General Configuration API
=========================

APIs for managing gateway configuration settings.

Get Configuration
-----------------

.. http:get:: /general-configuration

   Load entire configuration page with current settings.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "gateway_identity": {
          "name": "Univa-GW-01",
          "serial_number": "GW2025-1190021",
          "deployment_site": "Chennai Port - Zone A",
          "location_mode": "manual",
          "latitude": 12.99123,
          "longitude": 80.12312,
          "asset_id": "CRN-CT-12"
        },
        "date_time": {
          "timezone": "Asia/Kolkata",
          "ntp_server": "pool.ntp.org",
          "current_date": "12/03/2025",
          "current_time": "14:53",
          "date_format": "DD/MM/YYYY",
          "time_format": "24-Hour",
          "language": "English"
        },
        "network": {
          "mode": "ethernet",
          "ethernet": {
            "ip_assignment": "dhcp",
            "static_ip": "192.168.1.50",
            "subnet_mask": "255.255.255.0",
            "gateway": "192.168.1.1",
            "dns1": "8.8.8.8",
            "dns2": "8.8.4.4"
          },
          "wifi": {
            "ssid": "",
            "password": "",
            "signal_strength": 3
          },
          "cellular": {
            "apn": "internet",
            "username": "",
            "password": "",
            "signal_strength": -67
          },
          "mac_address": "00:1A:2B:3C:4D:5E"
        },
        "heartbeat": {
          "interval": 30,
          "offline_threshold": 120
        }
      }

Update Configuration
--------------------

.. http:put:: /general-configuration

   Save all configuration changes.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /general-configuration HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "gateway_identity": {
          "name": "Univa-GW-01",
          "deployment_site": "Chennai Port - Zone A",
          "location_mode": "manual",
          "latitude": 12.99123,
          "longitude": 80.12312,
          "asset_id": "CRN-CT-12"
        },
        "date_time": {
          "timezone": "Asia/Kolkata",
          "ntp_server": "pool.ntp.org",
          "date_format": "DD/MM/YYYY",
          "time_format": "24-Hour",
          "language": "English"
        },
        "network": {
          "mode": "ethernet",
          "ethernet": {
            "ip_assignment": "dhcp",
            "static_ip": "192.168.1.50",
            "subnet_mask": "255.255.255.0",
            "gateway": "192.168.1.1",
            "dns1": "8.8.8.8",
            "dns2": "8.8.4.4"
          },
          "wifi": {
            "ssid": "",
            "password": ""
          },
          "cellular": {
            "apn": "internet",
            "username": "",
            "password": ""
          }
        },
        "heartbeat": {
          "interval": 30,
          "offline_threshold": 120
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Configuration saved successfully"
      }

Scan WiFi Networks
------------------

.. http:post:: /general-configuration/wifi-scan

   Scan for available WiFi networks.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Request**:

   .. sourcecode:: http

      POST /general-configuration/wifi-scan HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {}

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "networks": [
          {"ssid": "Port_WiFi_5G", "signal": 4},
          {"ssid": "Guest_WiFi", "signal": 3}
        ]
      }

Sync Time
---------

.. http:post:: /general-configuration/time-sync

   Sync time with selected NTP server (Sync Now button).

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      POST /general-configuration/time-sync HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "timezone": "Asia/Kolkata",
        "ntp_server": "pool.ntp.org",
        "date_format": "DD/MM/YYYY",
        "time_format": "24-Hour",
        "language": "English"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "date_time": {
          "timezone": "Asia/Kolkata",
          "ntp_server": "pool.ntp.org",
          "current_date": "12/03/2025",
          "current_time": "14:55",
          "date_format": "DD/MM/YYYY",
          "time_format": "24-Hour",
          "language": "English"
        }
      }

Error Codes
-----------

.. list-table:: General Configuration Errors
   :widths: 20 80
   :header-rows: 1

   * - Error Code
     - Description
   * - CONFIG_001
     - Invalid network configuration
   * - CONFIG_002
     - Timezone not supported
   * - CONFIG_003
     - NTP server unreachable
   * - CONFIG_004
     - Invalid location coordinates
   * - CONFIG_005
     - Configuration save failed

Examples
--------

Python Example
~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Get configuration
   headers = {"Authorization": "Bearer your_token"}
   response = requests.get("https://api.example.com/v1/general-configuration", 
                          headers=headers)
   config = response.json()
   print(f"Gateway name: {config['gateway_identity']['name']}")
   
   # Update configuration
   update_data = {
       "gateway_identity": {
           "name": "Updated-GW-01",
           "deployment_site": "Chennai Port - Zone B"
       }
   }
   response = requests.put("https://api.example.com/v1/general-configuration",
                          json=update_data, headers=headers)
   print(response.json()["message"])

JavaScript Example
~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Get configuration
   async function getConfiguration() {
       const token = localStorage.getItem('token');
       const response = await fetch('/general-configuration', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       console.log('Current config:', data);
       return data;
   }
   
   // Scan WiFi
   async function scanWiFi() {
       const token = localStorage.getItem('token');
       const response = await fetch('/general-configuration/wifi-scan', {
           method: 'POST',
           headers: {
               'Authorization': `Bearer ${token}`,
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({})
       });
       
       return response.json();
   }
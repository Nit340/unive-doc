General Configuration API
=========================

This document describes the single configuration page and its related API endpoints.

Page Route (Frontend)
---------------------

.. http:get:: /general-configuration

   **Description**: Renders the complete configuration page WITH all current settings embedded in the HTML.
   
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
        <title>General Configuration</title>
      </head>
      <body>
        <div id="app" data-config='{
          "gateway_identity": {
            "name": "Univa-GW-01",
            "serial_number": "GW2025-1190021",
            "deployment_site": "Chennai Port - Zone A"
          },
          "date_time": {
            "timezone": "Asia/Kolkata",
            "current_time": "14:53"
          }
        }'>
          <!-- Page content with pre-populated form fields -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - Current configuration data is embedded in the page (e.g., in a `data-config` attribute or script tag)
   - JavaScript reads this data and populates the form fields
   - No separate API call needed on page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle actions FROM the configuration page.

.. http:put:: /api/general-configuration

   **Description**: Save all configuration changes (when user clicks "Save").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      PUT /api/general-configuration HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "gateway_identity": {
          "name": "Updated-GW-01",
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
          }
        },
        "heartbeat": {
          "interval": 30,
          "offline_threshold": 120
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Configuration saved successfully"
      }

.. http:post:: /api/general-configuration/wifi-scan

   **Description**: Scan for WiFi networks (when user clicks "Scan Networks").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "networks": [
          {"ssid": "Port_WiFi_5G", "signal": 4, "encryption": "WPA2"},
          {"ssid": "Guest_WiFi", "signal": 3, "encryption": "WPA2"}
        ]
      }

.. http:post:: /api/general-configuration/time-sync

   **Description**: Manual time synchronization (when user clicks "Sync Now").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Time synchronized successfully",
        "current_time": "14:55",
        "current_date": "12/03/2025"
      }

Route Summary
-------------

.. list-table:: Complete Route List
   :header-rows: 1
   :widths: 25 25 40 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/general-configuration`` - Loads page WITH embedded config data
     - Yes
   * - API
     - PUT
     - ``/api/general-configuration`` - Save all changes
     - Yes
   * - API
     - POST
     - ``/api/general-configuration/wifi-scan`` - Scan WiFi
     - Yes
   * - API
     - POST
     - ``/api/general-configuration/time-sync`` - Sync time
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /general-configuration``
   - Server renders HTML
   - Current config is **embedded in the page**
   - Form fields pre-populated from embedded data

2. **User edits settings**: Changes values in form

3. **User clicks "Scan WiFi"**: 
   - JavaScript sends ``POST /api/general-configuration/wifi-scan``
   - Returns network list
   - Updates WiFi dropdown

4. **User clicks "Sync Now"**:
   - JavaScript sends ``POST /api/general-configuration/time-sync``
   - Updates time display

5. **User clicks "Save"**:
   - JavaScript collects all form data
   - Sends ``PUT /api/general-configuration``
   - Shows success message
   - Optionally reloads page to get fresh embedded data

Advantages of This Approach
---------------------------

1. **Faster page load**: No extra API call on initial load
2. **SEO friendly**: Data visible to search engines
3. **Works without JavaScript**: Form can submit traditionally (though enhanced with JS)
4. **Simplicity**: Clear separation - page has data, APIs handle actions


Cloud Integration API
=====================

This document describes the cloud integration page and its related API endpoints for managing cloud service connections and data publishing.

Page Route (Frontend)
---------------------

.. http:get:: /cloud-integration

   **Description**: Renders the complete cloud integration management page with all connections, statistics, and configuration embedded in the HTML.
   
   **UI Layout**:
   
   1. **Left Column**: Active Connections list with real-time status
   2. **Right Column**: Configuration panel with 4 tabs:
      - Connection settings
      - Topics & Format
      - Tag Publishing
      - Advanced settings
   
   **UI Flow**:
   
   1. Page loads with embedded initial data
   2. User sees connection list on left
   3. Clicking any connection shows its configuration on right
   4. "Add Connection" button opens type selection modal
   5. Selecting a type loads that protocol's configuration form
   6. Form filled and saved → new connection appears in list
   
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
        <title>Cloud Integration - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-connections='[...]' data-statistics='{...}' data-templates='{...}'>
          <!-- Embedded data includes:
               - All configured connections with status
               - Overall statistics
               - Connection type templates
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page with embedded data
   - JavaScript renders the two-column layout
   - Left column shows active connections
   - Right column shows configuration when a connection is selected
   - No initial API call needed

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all cloud integration operations triggered from the page.

Connection CRUD Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/cloud-integration/connections

   **UI Trigger**: Clicking "Add Connection" → selecting connection type → filling form → clicking "Add Connection" in modal
   
   **UI Flow**:
   
   1. Click "Add Connection" button in header
   2. Modal opens showing connection types (MQTT, HTTP, AWS, Azure, etc.)
   3. User selects a type (e.g., MQTT)
   4. Modal loads MQTT configuration form
   5. User fills form with broker details
   6. Click "Add Connection" → sends ``POST /api/cloud-integration/connections``
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/cloud-integration/connections HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "type": "mqtt",
        "name": "Production MQTT Broker",
        "enabled": true,
        "connection": {
          "host": "broker.example.com",
          "port": 1883,
          "username": "gateway_user",
          "password": "secure_password"
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "id": "mqtt-456",
        "connection": {
          "id": "mqtt-456",
          "type": "mqtt",
          "name": "Production MQTT Broker",
          "enabled": true,
          "status": "connecting"
        }
      }
   
   **UI Response**: 
   - Modal closes
   - New connection appears in left column list
   - Connection automatically selected and shows in right panel

.. http:put:: /api/cloud-integration/connections/{id}

   **UI Trigger**: Editing connection in right panel and clicking "Save Changes"
   
   **UI Flow**:
   
   1. User clicks on connection in left list
   2. Configuration loads in right panel
   3. User edits fields in any of the 4 tabs
   4. Click "Save Changes" → sends ``PUT /api/cloud-integration/connections/{id}``
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection updated successfully",
        "connection": {
          "id": "mqtt-123",
          "name": "Updated MQTT Broker"
        }
      }
   
   **UI Response**: 
   - Connection name updates in left list
   - Success notification appears
   - Connection status updates if needed

.. http:delete:: /api/cloud-integration/connections/{id}

   **UI Trigger**: Clicking "Delete" button in connection configuration
   
   **UI Flow**:
   
   1. User selects connection
   2. Clicks "Delete" button in configuration panel
   3. Confirmation dialog appears
   4. User confirms → sends ``DELETE /api/cloud-integration/connections/{id}``
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection deleted successfully",
        "deletedId": "mqtt-123"
      }
   
   **UI Response**: 
   - Connection removed from left list
   - Right panel shows "No Connection Selected" state
   - Success notification appears

Connection Information & Selection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/cloud-integration/connections/{id}

   **UI Trigger**: Clicking on any connection in the left list
   
   **UI Flow**:
   
   1. User clicks connection in left list
   2. JavaScript sends ``GET /api/cloud-integration/connections/{id}``
   3. Right panel loads with connection details and 4 configuration tabs
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": "mqtt-123",
        "type": "mqtt",
        "name": "Production MQTT",
        "enabled": true,
        "status": "connected",
        "config": {
          "host": "broker.example.com",
          "port": 1883
        },
        "statistics": {
          "messages": 1250,
          "errors": 3
        }
      }
   
   **UI Response**: 
   - Right panel shows connection name and status
   - Statistics update in quick stats section
   - Configuration tabs populated with saved values

Connection Types & Templates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/cloud-integration/connection-types

   **UI Trigger**: Clicking "Add Connection" button (loads connection type cards)
   
   **UI Flow**:
   
   1. User clicks "Add Connection"
   2. Modal opens and immediately calls ``GET /api/cloud-integration/connection-types``
   3. Modal displays 8 connection type cards (MQTT, HTTP, AWS, Azure, Grafana, WebSocket, FTP, UDP)
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "types": [
          {
            "id": "mqtt",
            "name": "MQTT Broker",
            "description": "Standard IoT messaging protocol",
            "icon": "fa-solid fa-cloud",
            "color": "blue"
          }
        ]
      }
   
   **UI Response**: 
   - Modal shows grid of connection type cards with icons and descriptions
   - User clicks a card to proceed to configuration

.. http:get:: /api/cloud-integration/templates/{type}

   **UI Trigger**: Selecting a connection type card in "Add Connection" modal
   
   **UI Flow**:
   
   1. User clicks on a connection type card (e.g., MQTT)
   2. Modal loads specific form template via ``GET /api/cloud-integration/templates/mqtt``
   3. Modal shows protocol-specific configuration form
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "type": "mqtt",
        "defaults": {
          "host": "",
          "port": 1883,
          "protocol": "mqtt"
        },
        "validation": {
          "host": {"required": true}
        }
      }
   
   **UI Response**: 
   - Modal title changes to "Configure MQTT Broker"
   - Form fields appear with protocol-specific inputs
   - User fills form and clicks "Add Connection"

Connection Testing
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/cloud-integration/connections/{id}/test

   **UI Trigger**: Clicking "Test Connection" button in configuration panel
   
   **UI Flow**:
   
   1. User selects a connection
   2. Clicks "Test Connection" button in configuration panel
   3. Button shows "Testing..." with spinner
   4. API call tests connectivity
   5. Results shown in modal
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "connected": true,
        "latency": 124,
        "message": "Connected successfully"
      }
   
   **UI Response**: 
   - "Test Results" modal opens showing status, latency, authentication
   - Statistics update in quick stats
   - Success notification appears

Connection Status Toggle
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/cloud-integration/connections/{id}/toggle

   **UI Trigger**: Toggling the "Enabled" switch in connection header
   
   **UI Flow**:
   
   1. User toggles the switch next to connection name
   2. Switch changes state immediately
   3. API call updates backend status
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection enabled",
        "connection": {
          "id": "mqtt-123",
          "enabled": true
        }
      }
   
   **UI Response**: 
   - Connection status updates in left list (Active/Inactive badge)
   - Connection status indicator changes (online/offline)

Tag Management
~~~~~~~~~~~~~~

.. http:get:: /api/cloud-integration/available-tags

   **UI Trigger**: Opening tag selection in "Tag Publishing" tab
   
   **UI Flow**:
   
   1. User selects connection
   2. Clicks "Tag Publishing" tab
   3. Clicks "Add Tags" button
   4. Modal opens and loads available tags via API
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "tags": [
          {
            "name": "Hoist_Load",
            "displayName": "Hoist Load",
            "unit": "tons",
            "category": "Load Monitoring",
            "device": "LoadCell_1"
          }
        ],
        "total": 45
      }
   
   **UI Response**: 
   - Modal shows list of available tags with checkboxes
   - User selects tags and clicks "Add Selected Tags"

.. http:post:: /api/cloud-integration/connections/{id}/tags

   **UI Trigger**: Adding tags to connection in "Tag Publishing" tab
   
   **UI Flow**:
   
   1. User selects tags in modal
   2. Clicks "Add Selected Tags"
   3. Tags assigned to connection
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/cloud-integration/connections/mqtt-123/tags HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "tags": ["Hoist_Load", "Boom_Angle"],
        "publishMode": "onChange",
        "changeThreshold": 0.1
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Tags assigned successfully",
        "assignedCount": 2
      }

Statistics and Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/cloud-integration/connections/{id}/statistics

   **UI Trigger**: Loading connection details (automatically fetched when connection selected)
   
   **UI Flow**:
   
   1. User clicks connection in left list
   2. API fetches statistics along with connection details
   3. Statistics displayed in quick stats section
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "connectionId": "mqtt-123",
        "messages": {
          "total": 1250,
          "successful": 1247,
          "failed": 3
        },
        "latency": {
          "average": 124
        }
      }
   
   **UI Response**: 
   - Quick stats update (Messages, Errors, Last Sent, Latency)
   - Diagnostics section updates

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:put:: /api/cloud-integration/save-config

   **UI Trigger**: Clicking "Save" button in header
   
   **UI Flow**:
   
   1. User makes changes to one or more connections
   2. Clicks "Save" button in header
   3. All connection configurations saved
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Configuration saved successfully"
      }
   
   **UI Response**: 
   - Button shows "Saving..." with spinner
   - Success notification appears
   - Button returns to normal state

Route Summary
-------------

.. list-table:: Cloud Integration Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/cloud-integration`` - Main cloud integration page
     - Yes
   * - API
     - POST
     - ``/api/cloud-integration/connections`` - Create connection
     - Yes
   * - API
     - PUT
     - ``/api/cloud-integration/connections/{id}`` - Update connection
     - Yes
   * - API
     - DELETE
     - ``/api/cloud-integration/connections/{id}`` - Delete connection
     - Yes
   * - API
     - GET
     - ``/api/cloud-integration/connections/{id}`` - Get connection details
     - Yes
   * - API
     - GET
     - ``/api/cloud-integration/connection-types`` - Get available types
     - Yes
   * - API
     - GET
     - ``/api/cloud-integration/templates/{type}`` - Get type template
     - Yes
   * - API
     - POST
     - ``/api/cloud-integration/connections/{id}/test`` - Test connection
     - Yes
   * - API
     - POST
     - ``/api/cloud-integration/connections/{id}/toggle`` - Enable/disable
     - Yes
   * - API
     - GET
     - ``/api/cloud-integration/available-tags`` - Get available tags
     - Yes
   * - API
     - POST
     - ``/api/cloud-integration/connections/{id}/tags`` - Assign tags
     - Yes
   * - API
     - GET
     - ``/api/cloud-integration/statistics`` - Get overall statistics
     - Yes
   * - API
     - PUT
     - ``/api/cloud-integration/save-config`` - Save configuration
     - Yes

Complete User Flow
------------------

1. **User visits page**: ``GET /cloud-integration``
   - Page loads with embedded connections data
   - Left column shows active connections
   - Right column shows "No Connection Selected"

2. **User adds new connection**:
   - Clicks "Add Connection" → opens modal
   - Sees 8 connection type cards (MQTT, HTTP, AWS, Azure, Grafana, WebSocket, FTP, UDP)
   - Selects "MQTT Broker" card
   - Modal loads MQTT configuration form
   - Fills form with broker details
   - Clicks "Add Connection" → ``POST /api/cloud-integration/connections``
   - New connection appears in left list and is automatically selected

3. **User edits connection**:
   - Clicks connection in left list → loads in right panel
   - Makes changes in any of 4 tabs:
     * Connection: Basic settings
     * Topics & Format: Protocol-specific formatting
     * Tag Publishing: Which tags to send
     * Advanced: TLS, timeouts, retries
   - Clicks "Save Changes" → ``PUT /api/cloud-integration/connections/{id}``
   - Changes saved, notification appears

4. **User tests connection**:
   - With connection selected, clicks "Test Connection"
   - Button shows spinner, API tests connectivity
   - "Test Results" modal opens with success/failure
   - Statistics update in quick stats

5. **User manages tags**:
   - Selects connection, clicks "Tag Publishing" tab
   - Clicks "Add Tags" → opens modal with available tags
   - Selects tags to publish
   - Clicks "Add Selected Tags" → ``POST /api/cloud-integration/connections/{id}/tags``
   - Tags assigned to connection

6. **User deletes connection**:
   - Selects connection in left list
   - Clicks "Delete" button in configuration panel
   - Confirms deletion → ``DELETE /api/cloud-integration/connections/{id}``
   - Connection removed from list

7. **User saves all changes**:
   - After making multiple edits
   - Clicks "Save" button in header → ``PUT /api/cloud-integration/save-config``
   - All connection configurations saved

UI Component Details
--------------------

### Left Column (Connections List)
- Each connection shows: Name, Type icon, Status, Last active, Message count
- Active connections have green "Active" badge
- Inactive connections have gray "Inactive" badge
- Clicking any connection selects it and loads details in right panel

### Right Column (Configuration Panel)
- **Header**: Connection name, description, enabled toggle
- **Quick Stats**: Messages, Errors, Last Sent, Latency (updates in real-time)
- **Configuration Tabs**:
  1. **Connection**: Protocol-specific settings
  2. **Topics & Format**: Message formatting and topics
  3. **Tag Publishing**: Which data tags to publish
  4. **Advanced**: Advanced protocol settings
- **Diagnostics**: Real-time connection monitoring
- **Downlink Commands**: Recent commands from cloud

### Connection Type Cards (Modal)
- 8 cards displayed in 2x4 grid on desktop
- Each card has: Icon, Name, Description
- Cards are color-coded by protocol type
- Hover effect with slight elevation

### Test Results Modal
- Shows: Connection status, Latency, Authentication result
- Green success message or red error message
- Detailed technical information if needed

Error Codes
-----------

.. list-table:: Cloud Integration Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - CLOUD_001
     - Connection failed - host unreachable
   * - CLOUD_002
     - Authentication failed
   * - CLOUD_003
     - Invalid certificate
   * - CLOUD_004
     - Topic format error
   * - CLOUD_005
     - Payload too large
   * - CLOUD_006
     - Rate limit exceeded
   * - CLOUD_007
     - Connection timeout

Supported Protocols
-------------------

### Connection Types Available:
1. **MQTT Broker** - Standard IoT messaging
2. **HTTP Endpoint** - REST API integration
3. **AWS IoT Core** - Amazon Web Services
4. **Azure IoT Hub** - Microsoft Azure
5. **Grafana Cloud** - Monitoring & dashboards
6. **WebSocket Server** - Real-time bidirectional
7. **FTP Server** - File transfer for logs
8. **UDP Stream** - Low-latency streaming

### Protocol-Specific Features:
- **MQTT**: QoS 0/1/2, Retained messages, Last Will & Testament
- **HTTP**: Multiple methods (POST, PUT), Custom headers, Authentication
- **WebSocket**: Real-time bidirectional, Low latency
- **UDP**: High-speed unidirectional, Connectionless
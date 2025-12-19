Cloud Integration API
=====================

APIs for configuring and managing cloud service connections.

Overview
--------

The Cloud Integration API allows you to connect your IoT gateway to various cloud services and platforms for data publishing, remote monitoring, and analytics. Supports MQTT, HTTP, and other protocols.

Endpoints
---------

Get All Cloud Connections
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections

   List all configured cloud connections.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "connections": [
          {
            "id": "mqtt-123",
            "type": "mqtt",
            "name": "Production MQTT",
            "enabled": true,
            "status": "connected",
            "lastActive": "2 minutes ago",
            "messageCount": 1250,
            "lastError": null
          },
          {
            "id": "http-456",
            "type": "http",
            "name": "Analytics API",
            "enabled": false,
            "status": "disconnected",
            "lastActive": "5 hours ago",
            "messageCount": 340,
            "lastError": "Connection timeout"
          }
        ],
        "summary": {
          "total": 2,
          "enabled": 1,
          "connected": 1,
          "error": 1
        }
      }

Get Connection Types
~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections/types

   Get available connection types with metadata.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "types": [
          {
            "id": "mqtt",
            "name": "MQTT Broker",
            "description": "Publish data to MQTT broker",
            "icon": "fa-cloud",
            "protocols": ["mqtt", "mqtts", "ws", "wss"],
            "supports": ["publish", "subscribe", "retain", "qos"],
            "defaultPort": 1883
          },
          {
            "id": "http",
            "name": "HTTP Endpoint",
            "description": "Send data to HTTP REST API",
            "icon": "fa-globe",
            "protocols": ["http", "https"],
            "supports": ["post", "put", "get", "delete"],
            "defaultPort": 443
          },
          {
            "id": "aws-iot",
            "name": "AWS IoT Core",
            "description": "Amazon Web Services IoT Core",
            "icon": "fa-aws",
            "protocols": ["mqtts"],
            "supports": ["publish", "subscribe", "shadows"],
            "defaultPort": 8883
          },
          {
            "id": "azure-iot",
            "name": "Azure IoT Hub",
            "description": "Microsoft Azure IoT Hub",
            "icon": "fa-microsoft",
            "protocols": ["mqtts", "amqps"],
            "supports": ["publish", "deviceTwins", "directMethods"],
            "defaultPort": 8883
          }
        ]
      }

Get Connection Template
~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections/templates/{type}

   Get form template and defaults for a connection type.

   **Path Parameters**:

   * **type** (string): Connection type (mqtt, http, aws-iot, azure-iot)

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response** (MQTT Example):

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "type": "mqtt",
        "defaults": {
          "name": "New MQTT Connection",
          "enabled": true,
          "connection": {
            "protocol": "mqtts",
            "host": "",
            "port": 8883,
            "username": "",
            "password": "",
            "clientId": "gateway_${timestamp}",
            "cleanSession": true
          },
          "topics": {
            "baseTopic": "factory/device/${deviceId}",
            "dataTopic": "${baseTopic}/data",
            "eventTopic": "${baseTopic}/events",
            "commandTopic": "${baseTopic}/commands",
            "retainMessages": false
          },
          "format": {
            "payloadFormat": "json",
            "jsonTemplate": "{\"timestamp\":\"${timestamp}\",\"device\":\"${deviceId}\",\"data\":${data}}",
            "timestampFormat": "iso8601",
            "includeMetadata": true
          },
          "publishing": {
            "tags": [],
            "publishMode": "onChange",
            "changeThreshold": 0.1,
            "maxPublishRate": 1000,
            "batchSize": 10,
            "batchInterval": 1000
          },
          "advanced": {
            "autoReconnect": true,
            "reconnectInterval": 5000,
            "keepAlive": 60,
            "qos": 1,
            "timeout": 30
          }
        },
        "validation": {
          "host": {
            "required": true,
            "pattern": "^([a-zA-Z0-9\\-\\.]+)$"
          },
          "port": {
            "required": true,
            "min": 1,
            "max": 65535
          },
          "clientId": {
            "required": false,
            "maxLength": 23
          }
        },
        "variables": {
          "deviceId": "Gateway device ID",
          "timestamp": "Current timestamp",
          "random": "Random number"
        }
      }

Create Cloud Connection
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-connections

   Create a new cloud connection.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request** (MQTT Example):

   .. sourcecode:: http

      POST /cloud-connections HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "type": "mqtt",
        "name": "My MQTT Broker",
        "enabled": true,
        "connection": {
          "host": "broker.example.com",
          "port": 1883,
          "protocol": "mqtt",
          "username": "gateway_user",
          "password": "secure_password_123",
          "clientId": "gateway_001",
          "cleanSession": true
        },
        "topics": {
          "baseTopic": "factory/line1/gateway001",
          "dataTopic": "${baseTopic}/telemetry",
          "eventTopic": "${baseTopic}/events",
          "retainMessages": false
        },
        "format": {
          "payloadFormat": "json",
          "jsonTemplate": "{\"timestamp\":\"${timestamp}\",\"device\":\"${deviceId}\",\"data\":${data}}",
          "timestampFormat": "iso8601"
        },
        "publishing": {
          "tags": ["Hoist_Load", "Boom_Angle", "Temperature_Cell"],
          "publishMode": "onChange",
          "changeThreshold": 0.5,
          "maxPublishRate": 100
        },
        "advanced": {
          "autoReconnect": true,
          "reconnectInterval": 5000,
          "keepAlive": 60,
          "qos": 1,
          "timeout": 30
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "id": "mqtt-456",
        "connection": {
          "id": "mqtt-456",
          "type": "mqtt",
          "name": "My MQTT Broker",
          "enabled": true,
          "status": "connecting",
          "createdAt": "2025-03-12T10:30:00Z"
        },
        "message": "Connection created successfully"
      }

Get Connection Details
~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections/{id}

   Get detailed information for a specific connection.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "id": "mqtt-123",
        "type": "mqtt",
        "name": "Production MQTT",
        "enabled": true,
        "status": "connected",
        "connection": {
          "host": "broker.example.com",
          "port": 1883,
          "protocol": "mqtt",
          "username": "gateway_user",
          "clientId": "gateway_001",
          "cleanSession": true
        },
        "topics": {
          "baseTopic": "factory/line1/gateway001",
          "dataTopic": "factory/line1/gateway001/telemetry",
          "eventTopic": "factory/line1/gateway001/events",
          "commandTopic": "factory/line1/gateway001/commands",
          "retainMessages": false
        },
        "format": {
          "payloadFormat": "json",
          "jsonTemplate": "{\"timestamp\":\"${timestamp}\",\"device\":\"${deviceId}\",\"data\":${data}}",
          "timestampFormat": "iso8601",
          "includeMetadata": true
        },
        "publishing": {
          "tags": [
            {
              "name": "Hoist_Load",
              "published": true,
              "lastValue": 145.2,
              "lastPublished": "2025-03-12T10:29:58Z"
            },
            {
              "name": "Boom_Angle",
              "published": true,
              "lastValue": 45.6,
              "lastPublished": "2025-03-12T10:29:58Z"
            }
          ],
          "publishMode": "onChange",
          "changeThreshold": 0.5,
          "maxPublishRate": 100,
          "batchSize": 10,
          "totalMessages": 1250,
          "failedMessages": 3
        },
        "advanced": {
          "autoReconnect": true,
          "reconnectInterval": 5000,
          "keepAlive": 60,
          "qos": 1,
          "timeout": 30,
          "ssl": false,
          "certificate": null
        },
        "statistics": {
          "createdAt": "2025-03-01T09:00:00Z",
          "lastConnected": "2025-03-12T10:29:00Z",
          "lastDisconnected": null,
          "uptimeSeconds": 95040,
          "totalMessages": 1250,
          "failedMessages": 3,
          "averageLatency": 124
        }
      }

Test Connection
~~~~~~~~~~~~~~~

.. http:post:: /cloud-connections/{id}/test

   Test connectivity to cloud service.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Request** (Optional):

   .. sourcecode:: http

      POST /cloud-connections/mqtt-123/test HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "testType": "connectivity",
        "testMessage": "Test message from gateway",
        "timeout": 5000
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "connected": true,
        "latency": 124,
        "message": "Connected successfully to broker.example.com:1883",
        "details": {
          "brokerVersion": "2.0.0",
          "sessionPresent": false,
          "testTimestamp": "2025-03-12T10:30:45Z",
          "testMessageId": "test_123456"
        }
      }

Update Connection
~~~~~~~~~~~~~~~~~

.. http:put:: /cloud-connections/{id}

   Update an existing cloud connection.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>
      Content-Type: application/json

   **Request**:

   .. sourcecode:: http

      PUT /cloud-connections/mqtt-123 HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "name": "Updated MQTT Broker",
        "enabled": true,
        "connection": {
          "host": "new-broker.example.com",
          "port": 8883,
          "protocol": "mqtts",
          "username": "new_user",
          "password": "new_password"
        },
        "topics": {
          "baseTopic": "factory/updated/gateway001"
        },
        "publishing": {
          "publishMode": "interval",
          "interval": 1000
        }
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection updated successfully",
        "restartRequired": true,
        "connection": {
          "id": "mqtt-123",
          "name": "Updated MQTT Broker",
          "status": "restarting"
        }
      }

Get Available Tags
~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections/available-tags

   Get list of available tags for publishing.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **search** (optional): Search term for tag names
   * **limit** (optional): Maximum results (default: 100)
   * **category** (optional): Filter by tag category

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "tags": [
          {
            "name": "Hoist_Load",
            "displayName": "Hoist Load",
            "unit": "tons",
            "dataType": "FLOAT32",
            "category": "Load Monitoring",
            "device": "LoadCell_1",
            "description": "Main hoist load measurement",
            "currentValue": 145.2,
            "lastUpdate": "2025-03-12T10:29:58Z"
          },
          {
            "name": "Boom_Angle",
            "displayName": "Boom Angle",
            "unit": "degrees",
            "dataType": "FLOAT32",
            "category": "Position Tracking",
            "device": "Inclin_Arm",
            "description": "Boom angle measurement",
            "currentValue": 45.6,
            "lastUpdate": "2025-03-12T10:29:58Z"
          },
          {
            "name": "Temperature_Cell",
            "displayName": "Cell Temperature",
            "unit": "°C",
            "dataType": "FLOAT32",
            "category": "Temperature",
            "device": "LoadCell_1",
            "description": "Load cell temperature",
            "currentValue": 32.1,
            "lastUpdate": "2025-03-12T10:29:58Z"
          }
        ],
        "total": 45,
        "categories": ["Load Monitoring", "Position Tracking", "Temperature", "Pressure", "Safety"],
        "selectedCount": 0
      }

Delete Connection
~~~~~~~~~~~~~~~~~

.. http:delete:: /cloud-connections/{id}

   Delete a cloud connection.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **keepData** (optional): Keep historical data (default: false)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection deleted successfully",
        "deletedId": "mqtt-123",
        "statistics": {
          "totalMessages": 1250,
          "dataRetained": false,
          "cleanupComplete": true
        }
      }

Additional Endpoints
--------------------

Get Connection Statistics
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /cloud-connections/{id}/statistics

   Get detailed statistics for a connection.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Query Parameters**:

   * **timeRange** (optional): Time range (1h, 24h, 7d, 30d)

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "connectionId": "mqtt-123",
        "timeRange": "24h",
        "messages": {
          "total": 1250,
          "successful": 1247,
          "failed": 3,
          "ratePerMinute": 0.87
        },
        "latency": {
          "average": 124,
          "min": 89,
          "max": 245,
          "percentile95": 189
        },
        "bandwidth": {
          "totalBytes": 1250000,
          "averageBytesPerMessage": 1000,
          "bytesPerSecond": 14.5
        },
        "uptime": {
          "percentage": 99.8,
          "downtimeSeconds": 172,
          "lastDowntime": "2025-03-11T15:30:00Z"
        },
        "topTags": [
          {"tag": "Hoist_Load", "count": 450},
          {"tag": "Boom_Angle", "count": 400},
          {"tag": "Temperature_Cell", "count": 200}
        ]
      }

Toggle Connection State
~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /cloud-connections/{id}/toggle

   Enable or disable a connection.

   **Path Parameters**:

   * **id** (string): Connection ID

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Request**:

   .. sourcecode:: http

      POST /cloud-connections/mqtt-123/toggle HTTP/1.1
      Authorization: Bearer <token>
      Content-Type: application/json
      
      {
        "enabled": false
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Connection disabled",
        "connection": {
          "id": "mqtt-123",
          "enabled": false,
          "status": "disconnected"
        }
      }

Connection Types Reference
--------------------------

### MQTT Configuration

.. list-table:: MQTT Connection Options
   :widths: 30 70
   :header-rows: 1

   * - Field
     - Description
   * - **host**
     - MQTT broker hostname or IP
   * - **port**
     - Port number (1883 for MQTT, 8883 for MQTTS)
   * - **protocol**
     - mqtt, mqtts, ws, wss
   * - **username/password**
     - Authentication credentials
   * - **clientId**
     - Unique client identifier
   * - **cleanSession**
     - Start with clean session
   * - **qos**
     - Quality of Service (0, 1, 2)
   * - **keepAlive**
     - Keep-alive interval in seconds

### HTTP Configuration

.. list-table:: HTTP Connection Options
   :widths: 30 70
   :header-rows: 1

   * - Field
     - Description
   * - **url**
     - Complete endpoint URL
   * - **method**
     - HTTP method (POST, PUT, GET)
   * - **headers**
     - Custom HTTP headers
   * - **authType**
     - none, basic, bearer, apiKey
   * - **contentType**
     - application/json, application/xml
   * - **timeout**
     - Request timeout in milliseconds

### AWS IoT Core

.. list-table:: AWS IoT Configuration
   :widths: 30 70
   :header-rows: 1

   * - Field
     - Description
   * - **endpoint**
     - AWS IoT endpoint
   * - **thingName**
     - IoT Thing name
   * - **certificate**
     - Client certificate
   * - **privateKey**
     - Private key
   * - **rootCA**
     - Root CA certificate
   * - **region**
     - AWS region

Publishing Modes
----------------

.. list-table:: Publishing Modes
   :widths: 40 60
   :header-rows: 1

   * - Mode
     - Description
   * - **onChange**
     - Publish when value changes beyond threshold
   * - **interval**
     - Publish at fixed intervals
   * - **onEvent**
     - Publish on specific events
   * - **conditional**
     - Publish based on conditions
   * - **manual**
     - Manual publishing only

Error Codes
-----------

.. list-table:: Cloud Integration Errors
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - **CLOUD_001**
     - Connection failed - host unreachable
   * - **CLOUD_002**
     - Authentication failed
   * - **CLOUD_003**
     - Invalid certificate
   * - **CLOUD_004**
     - Topic format error
   * - **CLOUD_005**
     - Paytoo large
   * - **CLOUD_006**
     - Rate limit exceeded
   * - **CLOUD_007**
     - Connection timeout
   * - **CLOUD_008**
     - SSL/TLS handshake failed
   * - **CLOUD_009**
     - Invalid JSON template

Examples
--------

Python - Create MQTT Connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import requests
   
   # Create MQTT connection
   headers = {"Authorization": "Bearer your_token"}
   
   mqtt_config = {
       "type": "mqtt",
       "name": "Production MQTT Broker",
       "enabled": True,
       "connection": {
           "host": "mqtt.example.com",
           "port": 1883,
           "protocol": "mqtt",
           "username": "gateway",
           "password": "password123",
           "clientId": "gateway_001",
           "cleanSession": True
       },
       "topics": {
           "baseTopic": "factory/area1/gateway001",
           "dataTopic": "${baseTopic}/data",
           "eventTopic": "${baseTopic}/events"
       },
       "format": {
           "payloadFormat": "json",
           "jsonTemplate": '''{
               "timestamp": "${timestamp}",
               "device": "${deviceId}",
               "values": ${data},
               "metadata": {
                   "gateway": "Univa-GW-01",
                   "location": "Chennai Port"
               }
           }'''
       },
       "publishing": {
           "tags": ["Hoist_Load", "Boom_Angle"],
           "publishMode": "onChange",
           "changeThreshold": 0.1,
           "maxPublishRate": 100
       }
   }
   
   response = requests.post(
       "https://api.example.com/v1/cloud-connections",
       json=mqtt_config,
       headers=headers
   )
   
   if response.status_code == 201:
       connection = response.json()["connection"]
       print(f"Created connection: {connection['name']} (ID: {connection['id']})")

Python - Test Connection
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Test existing connection
   connection_id = "mqtt-123"
   
   test_response = requests.post(
       f"https://api.example.com/v1/cloud-connections/{connection_id}/test",
       headers=headers
   )
   
   test_result = test_response.json()
   
   if test_result["success"]:
       print(f"Connection successful! Latency: {test_result['latency']}ms")
   else:
       print(f"Connection failed: {test_result.get('message', 'Unknown error')}")

JavaScript - Dynamic Connection Form
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Load connection types
   async function loadConnectionTypes() {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-connections/types', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       const typeSelect = document.getElementById('connection-type');
       
       data.types.forEach(type => {
           const option = document.createElement('option');
           option.value = type.id;
           option.textContent = type.name;
           typeSelect.appendChild(option);
       });
       
       // Add change listener
       typeSelect.addEventListener('change', async (event) => {
           const type = event.target.value;
           await loadConnectionTemplate(type);
       });
   }
   
   // Load template for selected type
   async function loadConnectionTemplate(type) {
       const token = localStorage.getItem('token');
       
       const response = await fetch(`/cloud-connections/templates/${type}`, {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const template = await response.json();
       renderConnectionForm(template);
   }
   
   // Render dynamic form
   function renderConnectionForm(template) {
       const formContainer = document.getElementById('connection-form');
       formContainer.innerHTML = '';
       
       // Render form based on template
       renderSection('Connection', template.defaults.connection);
       renderSection('Topics', template.defaults.topics);
       renderSection('Format', template.defaults.format);
       renderSection('Publishing', template.defaults.publishing);
       renderSection('Advanced', template.defaults.advanced);
   }
   
   function renderSection(title, fields) {
       const section = document.createElement('div');
       section.className = 'form-section';
       
       const heading = document.createElement('h3');
       heading.textContent = title;
       section.appendChild(heading);
       
       Object.entries(fields).forEach(([key, value]) => {
           const fieldDiv = document.createElement('div');
           fieldDiv.className = 'form-field';
           
           const label = document.createElement('label');
           label.textContent = key;
           label.htmlFor = key;
           
           let input;
           
           if (typeof value === 'boolean') {
               input = document.createElement('input');
               input.type = 'checkbox';
               input.id = key;
               input.name = key;
               input.checked = value;
           } else if (typeof value === 'number') {
               input = document.createElement('input');
               input.type = 'number';
               input.id = key;
               input.name = key;
               input.value = value;
           } else {
               input = document.createElement('input');
               input.type = 'text';
               input.id = key;
               input.name = key;
               input.value = value || '';
               input.placeholder = `Enter ${key}`;
           }
           
           fieldDiv.appendChild(label);
           fieldDiv.appendChild(input);
           section.appendChild(fieldDiv);
       });
       
       document.getElementById('connection-form').appendChild(section);
   }

JavaScript - Manage Tags for Publishing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript

   // Load available tags
   async function loadAvailableTags() {
       const token = localStorage.getItem('token');
       
       const response = await fetch('/cloud-connections/available-tags', {
           headers: {
               'Authorization': `Bearer ${token}`
           }
       });
       
       const data = await response.json();
       const tagList = document.getElementById('available-tags');
       
       data.tags.forEach(tag => {
           const li = document.createElement('li');
           li.className = 'tag-item';
           
           const checkbox = document.createElement('input');
           checkbox.type = 'checkbox';
           checkbox.id = `tag-${tag.name}`;
           checkbox.value = tag.name;
           
           const label = document.createElement('label');
           label.htmlFor = `tag-${tag.name}`;
           label.innerHTML = `
               <strong>${tag.displayName}</strong>
               <span class="tag-info">${tag.device} • ${tag.unit}</span>
               <br>
               <small>${tag.description}</small>
           `;
           
           li.appendChild(checkbox);
           li.appendChild(label);
           tagList.appendChild(li);
       });
   }
   
   // Get selected tags
   function getSelectedTags() {
       const checkboxes = document.querySelectorAll('#available-tags input[type="checkbox"]:checked');
       return Array.from(checkboxes).map(cb => cb.value);
   }

Workflow Example
----------------

### Complete Cloud Connection Setup

1. **Page Load** - List existing connections
   
   .. code-block:: javascript
   
      // Initial load
      async function loadCloudConnections() {
          const connections = await getConnections();
          renderConnectionList(connections);
      }

2. **Add New Connection** - Show type selection
   
   .. code-block:: javascript
   
      function showAddConnection() {
          document.getElementById('connection-modal').style.display = 'block';
          loadConnectionTypes();
      }

3. **Select Connection Type** - Load template
   
   .. code-block:: javascript
   
      async function onTypeSelect(type) {
          const template = await getConnectionTemplate(type);
          renderConnectionForm(template);
      }

4. **Configure Connection** - Fill form
   
   .. code-block:: javascript
   
      async function configureConnection() {
          const config = collectFormData();
          
          // Test before saving
          const testResult = await testConnectionConfig(config);
          
          if (testResult.success) {
              await saveConnection(config);
          }
      }

5. **Save and Enable** - Create connection
   
   .. code-block:: javascript
   
      async function saveConnection(config) {
          const result = await createConnection(config);
          
          if (result.success) {
              // Enable publishing
              await enableConnection(result.id);
              
              // Load tags for publishing
              const tags = await getAvailableTags();
              await selectTagsForPublishing(result.id, tags);
          }
      }

Security Considerations
-----------------------

### Credential Management

1. **Password Storage**
   - Encrypted at rest
   - Never logged in plaintext
   - Rotate regularly

2. **Certificate Handling**
   - Secure certificate upload
   - Private key protection
   - Certificate expiration monitoring

3. **Network Security**
   - Use TLS/SSL where available
   - Validate certificates
   - Use VPN for sensitive connections

### Data Privacy

1. **Data Minimization**
   - Only publish necessary tags
   - Consider data aggregation
   - Implement data masking if needed

2. **Compliance**
   - GDPR considerations
   - Industry-specific regulations
   - Data retention policies

Monitoring and Alerts
---------------------

### Connection Health Monitoring

- **Heartbeat**: Regular connectivity checks
- **Latency Monitoring**: Track response times
- **Error Rate**: Monitor failed messages
- **Bandwidth Usage**: Track data volume

### Alert Configuration

.. code-block:: json

   {
     "alerts": {
       "connectionLost": {
         "enabled": true,
         "timeout": 300,
         "notifications": ["email", "sms"]
       },
       "highLatency": {
         "enabled": true,
         "threshold": 1000,
         "duration": 60
       },
       "errorRate": {
         "enabled": true,
         "threshold": 0.05,
         "window": 300
       }
     }
   }

Best Practices
--------------

1. **Connection Configuration**
   - Use descriptive names
   - Document connection purposes
   - Test before production use

2. **Data Publishing**
   - Start with few tags, add gradually
   - Use appropriate publish modes
   - Monitor bandwidth usage

3. **Error Handling**
   - Implement retry logic
   - Log connection issues
   - Set up alerts for critical failures

4. **Performance**
   - Batch messages where possible
   - Use compression for large payloads
   - Optimize publish intervals

Troubleshooting
---------------

### Common Issues

1. **Connection Fails**
   - Check network connectivity
   - Verify credentials
   - Check firewall settings

2. **Authentication Errors**
   - Verify username/password
   - Check certificate validity
   - Ensure proper permissions

3. **High Latency**
   - Check network bandwidth
   - Reduce publish frequency
   - Consider message batching

4. **Message Loss**
   - Increase QoS level
   - Check broker configuration
   - Verify topic permissions

### Debug Tools

1. **Test Connection** - Verify connectivity
2. **Message Logs** - View sent messages
3. **Network Capture** - Analyze traffic
4. **Broker Tools** - Use MQTT client for testing
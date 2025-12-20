Notification Channels API
=========================

Purpose
-------

The Notification Channels API manages delivery channels for alert messages, including MQTT, email, SMS, webhook, and push notifications. It provides CRUD operations for channel configurations, testing, and status monitoring.

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

Channels Management API
~~~~~~~~~~~~~~~~~~~~~~~

Get All Channels
^^^^^^^^^^^^^^^^

.. http:get:: /channels

   Retrieve all notification channels.

   **Query Parameters:**

   * **type** (optional): Filter by channel type - ``mqtt``, ``email``, ``sms``, ``webhook``, ``push``
   * **status** (optional): Filter by status - ``active``, ``inactive``, ``testing``
   * **page** (optional): Page number (default: ``1``)
   * **limit** (optional): Items per page (default: ``20``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": [
          {
            "id": "mqtt-primary",
            "name": "MQTT Alert Broker",
            "type": "mqtt",
            "status": "active",
            "enabled": true,
            "created_at": "2024-01-10T08:30:00Z",
            "last_updated": "2024-01-15T09:45:00Z",
            "last_test": "2024-01-15T10:30:00Z",
            "last_success": "2024-01-15T10:30:00Z",
            "config": {
              "broker_url": "mqtt.factory.com:1883",
              "topic": "factory/alerts",
              "username": "univa_gateway"
            }
          }
        ],
        "meta": {
          "count": 2,
          "total": 5,
          "active": 4,
          "inactive": 1,
          "page": 1,
          "total_pages": 1
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **401 Unauthorized**: Invalid or missing API key
   * **500 Internal Server Error**: Server error

Get Single Channel
^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/{id}

   Retrieve a specific channel by ID.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "mqtt-primary",
          "name": "MQTT Alert Broker",
          "type": "mqtt",
          "status": "active",
          "enabled": true,
          "created_at": "2024-01-10T08:30:00Z",
          "last_updated": "2024-01-15T09:45:00Z",
          "last_test": "2024-01-15T10:30:00Z",
          "last_success": "2024-01-15T10:30:00Z",
          "config": {
            "broker_url": "mqtt.factory.com:1883",
            "topic": "factory/alerts",
            "username": "univa_gateway",
            "password": "********",
            "qos": 1,
            "retain": false,
            "client_id": "univa_gateway_01"
          },
          "statistics": {
            "messages_sent": 1250,
            "success_rate": 99.8,
            "last_24h": 45,
            "last_7d": 320
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key

Create Channel
^^^^^^^^^^^^^^

.. http:post:: /channels

   Create a new notification channel.

   **Request:**

   .. sourcecode:: http

      POST /channels HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "SMS Emergency Channel",
        "type": "sms",
        "enabled": true,
        "config": {
          "provider": "Twilio",
          "api_key": "SK-XXXXXXXXXXXXXXXX",
          "account_sid": "AC-XXXXXXXXXXXXXXXX",
          "auth_token": "********",
          "from_number": "+1234567890",
          "to_numbers": ["+919876543210", "+971501234567"]
        }
      }

   **Required Fields:**

   * **name** (string): Name of the channel
   * **type** (string): Channel type - ``mqtt``, ``email``, ``sms``, ``webhook``, ``push``
   * **config** (object): Configuration specific to channel type

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "sms-emergency",
          "name": "SMS Emergency Channel",
          "type": "sms",
          "status": "active",
          "enabled": true,
          "created_at": "2024-01-15T11:30:00Z",
          "last_updated": "2024-01-15T11:30:00Z",
          "config": {
            "provider": "Twilio",
            "account_sid": "AC-XXXXXXXXXXXXXXXX",
            "from_number": "+1234567890",
            "to_numbers": ["+919876543210", "+971501234567"]
          }
        },
        "message": "Channel created successfully"
      }

   **Status Codes:**

   * **201 Created**: Channel created successfully
   * **400 Bad Request**: Missing or invalid fields
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Channel with same name already exists

Update Channel
^^^^^^^^^^^^^^

.. http:put:: /channels/{id}

   Update an existing channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Request:**

   .. sourcecode:: http

      PUT /channels/mqtt-primary HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "MQTT Primary Updated",
        "enabled": true,
        "config": {
          "broker_url": "mqtt.factory.local:1883",
          "topic": "factory/alerts/v2",
          "username": "univa_gateway_v2",
          "password": "********",
          "qos": 2
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "mqtt-primary",
          "name": "MQTT Primary Updated",
          "type": "mqtt",
          "status": "active",
          "enabled": true,
          "created_at": "2024-01-10T08:30:00Z",
          "last_updated": "2024-01-15T11:45:00Z",
          "config": {
            "broker_url": "mqtt.factory.local:1883",
            "topic": "factory/alerts/v2",
            "username": "univa_gateway_v2",
            "qos": 2,
            "retain": false
          }
        },
        "message": "Channel updated successfully"
      }

   **Status Codes:**

   * **200 OK**: Channel updated successfully
   * **400 Bad Request**: Invalid fields
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key

Update Channel Status
^^^^^^^^^^^^^^^^^^^^^

.. http:patch:: /channels/{id}/status

   Update channel enabled status.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Request:**

   .. sourcecode:: http

      PATCH /channels/mqtt-primary/status HTTP/1.1
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
          "id": "mqtt-primary",
          "enabled": false,
          "previous_status": true
        },
        "message": "Channel disabled successfully"
      }

   **Status Codes:**

   * **200 OK**: Status updated successfully
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key

Delete Channel
^^^^^^^^^^^^^^

.. http:delete:: /channels/{id}

   Delete a channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "message": "Channel deleted successfully"
      }

   **Status Codes:**

   * **200 OK**: Channel deleted successfully
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Cannot delete - alerts are using this channel

Channel Configuration Templates API
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get Channel Template
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/templates/{type}

   Get configuration template for a channel type.

   **Path Parameters:**

   * **type** (string): Channel type

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "type": "mqtt",
          "name": "MQTT Broker",
          "description": "Publish alert messages to MQTT broker",
          "icon": "fa-satellite-dish",
          "required_fields": ["broker_url", "topic"],
          "optional_fields": ["username", "password", "qos", "retain", "client_id"],
          "template": {
            "broker_url": "mqtt.server.com:1883",
            "topic": "alerts/topic",
            "username": "",
            "password": "",
            "qos": 1,
            "retain": false,
            "client_id": "univa_gateway"
          },
          "validation_rules": {
            "broker_url": "must match pattern hostname:port",
            "topic": "must be valid MQTT topic"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Template retrieved successfully
   * **404 Not Found**: Channel type not supported
   * **401 Unauthorized**: Invalid or missing API key

Channel Testing API
~~~~~~~~~~~~~~~~~~~

Test Channel Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /channels/{id}/test

   Test a channel configuration.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Request:**

   .. sourcecode:: http

      POST /channels/mqtt-primary/test HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "alert_id": "alert-001",
        "test_data": {
          "device": "Test_Device_01",
          "value": "85.6",
          "unit": "°C",
          "timestamp": "2024-01-15T12:00:00Z",
          "severity": "CRITICAL"
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "channel_id": "mqtt-primary",
          "channel_name": "MQTT Alert Broker",
          "test_time": "2024-01-15T12:00:30Z",
          "result": "success",
          "message": "Test notification sent successfully",
          "details": {
            "destination": "mqtt.factory.com:1883/factory/alerts",
            "message_sent": "🔥 CRITICAL: Test_Device_01 temperature is 85.6°C at 2024-01-15T12:00:00Z",
            "response_code": "PUBACK",
            "response_time": "0.125s"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Test completed successfully
   * **404 Not Found**: Channel or alert not found
   * **400 Bad Request**: Test data invalid
   * **401 Unauthorized**: Invalid or missing API key
   * **503 Service Unavailable**: Channel not reachable

Test Multiple Channels
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /channels/test-bulk

   Test multiple channels simultaneously.

   **Request:**

   .. sourcecode:: http

      POST /channels/test-bulk HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "channel_ids": ["mqtt-primary", "email-primary"],
        "alert_id": "alert-002",
        "test_data": {
          "device": "Pump_Station_A",
          "value": "5.8",
          "unit": "mm/s",
          "timestamp": "2024-01-15T12:05:00Z",
          "severity": "WARNING"
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "total_channels": 2,
          "tested_channels": 2,
          "successful": 2,
          "failed": 0,
          "results": [
            {
              "channel_id": "mqtt-primary",
              "result": "success",
              "message": "Test successful",
              "response_time": "0.120s"
            },
            {
              "channel_id": "email-primary",
              "result": "success",
              "message": "Test email queued for delivery",
              "response_time": "0.850s"
            }
          ],
          "summary": "All 2 channels tested successfully"
        }
      }

   **Status Codes:**

   * **200 OK**: Tests completed
   * **400 Bad Request**: Invalid channel IDs or test data
   * **401 Unauthorized**: Invalid or missing API key

Channel Statistics API
~~~~~~~~~~~~~~~~~~~~~~

Get Channel Statistics
^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/{id}/statistics

   Get statistics for a channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Query Parameters:**

   * **start_date** (optional): Start date for statistics (ISO format)
   * **end_date** (optional): End date for statistics (ISO format)
   * **granularity** (optional): Time granularity - ``hour``, ``day``, ``week``, ``month``

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "channel_id": "mqtt-primary",
          "channel_name": "MQTT Alert Broker",
          "period": {
            "start": "2024-01-01T00:00:00Z",
            "end": "2024-01-15T23:59:59Z"
          },
          "overall": {
            "messages_sent": 1250,
            "messages_failed": 5,
            "success_rate": 99.6,
            "average_response_time": 0.125,
            "total_data_transferred": "2.4 MB"
          },
          "by_hour": [
            {
              "hour": "00:00",
              "messages_sent": 15,
              "messages_failed": 0,
              "success_rate": 100.0
            }
          ]
        }
      }

   **Status Codes:**

   * **200 OK**: Statistics retrieved successfully
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key

Get Overview Statistics
^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/statistics/overview

   Get overview statistics for all channels.

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
          "total_channels": 5,
          "active_channels": 4,
          "inactive_channels": 1,
          "channels_by_type": {
            "mqtt": 2,
            "email": 1,
            "sms": 1,
            "webhook": 1
          },
          "performance": {
            "total_messages_sent": 2850,
            "total_messages_failed": 12,
            "overall_success_rate": 99.58,
            "peak_throughput": "125 msg/hour",
            "average_response_time": 0.342
          }
        },
        "timestamp": "2024-01-15T12:00:00Z"
      }

   **Status Codes:**

   * **200 OK**: Overview retrieved successfully
   * **401 Unauthorized**: Invalid or missing API key

Channel Alert Delivery API
~~~~~~~~~~~~~~~~~~~~~~~~~~

Deliver Alert
^^^^^^^^^^^^^

.. http:post:: /channels/{id}/deliver

   Deliver an alert through a specific channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Request:**

   .. sourcecode:: http

      POST /channels/mqtt-primary/deliver HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "alert_id": "alert-001",
        "alert_data": {
          "device": "Crane_01",
          "value": "85.6",
          "unit": "°C",
          "timestamp": "2024-01-15T12:00:00Z",
          "location": "Factory Floor - Zone B",
          "severity": "CRITICAL",
          "threshold": "80"
        },
        "options": {
          "priority": "high",
          "retry_on_failure": true,
          "max_retries": 3,
          "timeout": 5000
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "delivery_id": "DEL-20250115-001",
          "channel_id": "mqtt-primary",
          "alert_id": "alert-001",
          "status": "sent",
          "timestamp": "2024-01-15T12:00:05Z",
          "message": "Alert delivered successfully",
          "details": {
            "destination": "mqtt.factory.com:1883/factory/alerts",
            "message": "🔥 CRITICAL: Crane_01 temperature is 85.6°C (Threshold: 80°C) at 2024-01-15T12:00:00Z",
            "response": "PUBACK",
            "delivery_time": "0.120s",
            "message_id": "MQTT-001"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Alert delivered successfully
   * **202 Accepted**: Alert queued for delivery
   * **404 Not Found**: Channel or alert not found
   * **400 Bad Request**: Invalid alert data
   * **401 Unauthorized**: Invalid or missing API key
   * **503 Service Unavailable**: Channel not reachable

Get Delivery History
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/deliveries

   Get delivery history for all channels.

   **Query Parameters:**

   * **channel_id** (optional): Filter by channel ID
   * **alert_id** (optional): Filter by alert ID
   * **status** (optional): Filter by status - ``sent``, ``delivered``, ``failed``, ``pending``
   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **page** (optional): Page number (default: ``1``)
   * **limit** (optional): Items per page (default: ``50``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "deliveries": [
            {
              "delivery_id": "DEL-20250115-001",
              "channel_id": "mqtt-primary",
              "channel_name": "MQTT Alert Broker",
              "alert_id": "alert-001",
              "alert_name": "High Temperature Alert",
              "timestamp": "2024-01-15T10:18:00Z",
              "status": "sent",
              "message": "🔥 CRITICAL: Motor_Drive_01 temperature is 82.5°C",
              "destination": "mqtt.factory.com:1883/factory/alerts",
              "response_time": "0.125s",
              "error": null
            }
          ],
          "summary": {
            "total": 1250,
            "sent": 1245,
            "delivered": 1240,
            "failed": 5,
            "success_rate": 99.6
          }
        },
        "meta": {
          "count": 2,
          "total": 1250,
          "page": 1,
          "total_pages": 63
        }
      }

   **Status Codes:**

   * **200 OK**: Deliveries retrieved successfully
   * **401 Unauthorized**: Invalid or missing API key

Channel Configuration Validation API
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Validate Configuration
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /channels/validate

   Validate channel configuration.

   **Request:**

   .. sourcecode:: http

      POST /channels/validate HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "type": "mqtt",
        "config": {
          "broker_url": "mqtt.factory.com:1883",
          "topic": "factory/alerts",
          "username": "univa_gateway",
          "password": "********"
        }
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
            "Consider using TLS for secure connection",
            "Add client ID for better tracking"
          ],
          "test_connection": "optional",
          "config_preview": {
            "broker_url": "mqtt.factory.com:1883",
            "topic": "factory/alerts",
            "username": "********",
            "password": "********"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Validation completed
   * **400 Bad Request**: Invalid configuration
   * **401 Unauthorized**: Invalid or missing API key

Test Connection
^^^^^^^^^^^^^^^

.. http:post:: /channels/{id}/validate-connection

   Test connection to channel endpoint.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "channel_id": "mqtt-primary",
          "connection_test": "success",
          "timestamp": "2024-01-15T12:00:30Z",
          "response_time": "0.125s",
          "details": {
            "broker": "mqtt.factory.com:1883",
            "status": "reachable",
            "authentication": "successful",
            "permissions": "read/write",
            "qos_support": [0, 1, 2]
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Connection test completed
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key
   * **503 Service Unavailable**: Connection failed

Channel Import/Export API
~~~~~~~~~~~~~~~~~~~~~~~~~

Export Channels
^^^^^^^^^^^^^^^

.. http:get:: /channels/export

   Export channels to JSON configuration.

   **Query Parameters:**

   * **format** (optional): Export format - ``json`` (default), ``yaml``
   * **include_secrets** (optional): Include passwords/API keys - ``true``, ``false`` (default: ``false``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "export_url": "https://api.univagateway.com/v1/downloads/channels-export-2025-01-15.json",
          "expires_at": "2024-01-16T12:00:00Z",
          "summary": {
            "channels_exported": 5,
            "secrets_included": false,
            "file_size": "15.8 KB"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Export prepared successfully
   * **401 Unauthorized**: Invalid or missing API key

Import Channels
^^^^^^^^^^^^^^^

.. http:post:: /channels/import

   Import channels from JSON configuration.

   **Request:**

   .. sourcecode:: http

      POST /channels/import HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "channels": [
          {
            "name": "Imported MQTT Channel",
            "type": "mqtt",
            "enabled": true,
            "config": {
              "broker_url": "mqtt.new-server.com:1883",
              "topic": "factory/alerts/imported"
            }
          }
        ],
        "options": {
          "overwrite_existing": false,
          "enable_channels": true,
          "update_secrets": false
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
            "channels_created": 1,
            "channels_updated": 0,
            "secrets_updated": 0
          },
          "import_id": "IMP-20250115-002"
        },
        "message": "Channels imported successfully"
      }

   **Status Codes:**

   * **200 OK**: Channels imported successfully
   * **400 Bad Request**: Invalid import data
   * **401 Unauthorized**: Invalid or missing API key

Channel Monitoring API
~~~~~~~~~~~~~~~~~~~~~~

Get Real-time Monitoring
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /channels/{id}/monitor

   Get real-time monitoring data for a channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Query Parameters:**

   * **duration** (optional): Monitoring duration in seconds (default: ``60``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "channel_id": "mqtt-primary",
          "channel_name": "MQTT Alert Broker",
          "monitoring_start": "2024-01-15T12:00:00Z",
          "monitoring_end": "2024-01-15T12:01:00Z",
          "current_status": "active",
          "connection_uptime": "99.95%",
          "message_queue": {
            "pending": 0,
            "processing": 2,
            "completed": 15
          },
          "throughput": {
            "messages_per_second": 0.25,
            "bytes_per_second": 512,
            "peak_throughput": 2.5
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Monitoring data retrieved
   * **404 Not Found**: Channel not found
   * **401 Unauthorized**: Invalid or missing API key

Webhook Configuration API
~~~~~~~~~~~~~~~~~~~~~~~~~

Configure Webhook Endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /channels/webhook/{id}/configure

   Configure webhook endpoints for a webhook channel.

   **Path Parameters:**

   * **id** (string): Channel ID

   **Request:**

   .. sourcecode:: http

      POST /channels/webhook-primary/configure HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "endpoints": [
          {
            "url": "https://webhook.site/abcdef",
            "method": "POST",
            "headers": {
              "Content-Type": "application/json",
              "Authorization": "Bearer token-123"
            },
            "timeout": 5000,
            "retry_policy": {
              "max_retries": 3,
              "retry_delay": 1000
            }
          }
        ]
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "channel_id": "webhook-primary",
          "endpoints_configured": 1,
          "test_results": [
            {
              "url": "https://webhook.site/abcdef",
              "status": "success",
              "response_code": 200,
              "response_time": "0.320s"
            }
          ]
        },
        "message": "Webhook endpoints configured successfully"
      }

   **Status Codes:**

   * **200 OK**: Webhook configured successfully
   * **404 Not Found**: Channel not found or not a webhook channel
   * **401 Unauthorized**: Invalid or missing API key
   * **400 Bad Request**: Invalid webhook configuration

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

.. list-table:: Notification Channels Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **CHANNELS_001**
     - Invalid channel configuration
   * - **CHANNELS_002**
     - Channel not found
   * - **CHANNELS_003**
     - Channel already exists
   * - **CHANNELS_004**
     - Cannot delete - in use by alerts
   * - **CHANNELS_005**
     - Connection test failed
   * - **CHANNELS_006**
     - Delivery failed
   * - **CHANNELS_007**
     - Invalid channel type
   * - **CHANNELS_008**
     - Missing required configuration
   * - **CHANNELS_009**
     - Authentication failed
   * - **CHANNELS_010**
     - Rate limit exceeded
   * - **CHANNELS_011**
     - Import/export error

Webhooks
--------

Channel Status Change Webhook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a channel status changes:

.. code-block:: json

   {
     "event": "channel.status_changed",
     "timestamp": "2024-01-15T10:18:00Z",
     "data": {
       "channel_id": "mqtt-primary",
       "channel_name": "MQTT Alert Broker",
       "previous_status": "active",
       "new_status": "inactive",
       "reason": "manual_disable",
       "user": "admin@factory.com",
       "gateway": "Univa-GW-01"
     }
   }

Delivery Status Webhook
^^^^^^^^^^^^^^^^^^^^^^^

When alert delivery status changes:

.. code-block:: json

   {
     "event": "channel.delivery_status",
     "timestamp": "2024-01-15T10:18:00Z",
     "data": {
       "delivery_id": "DEL-20250115-001",
       "channel_id": "mqtt-primary",
       "alert_id": "alert-001",
       "status": "delivered",
       "timestamp": "2024-01-15T10:18:05Z",
       "message": "Alert delivered successfully",
       "response_time": "0.120s"
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
   
   # Create a new MQTT channel
   channel_data = {
       "name": "Primary MQTT Broker",
       "type": "mqtt",
       "enabled": True,
       "config": {
           "broker_url": "mqtt.factory.com:1883",
           "topic": "factory/alerts",
           "username": "univa_gateway",
           "password": "secure_password"
       }
   }
   
   response = requests.post(
       f"{base_url}/channels",
       headers=headers,
       json=channel_data
   )
   
   if response.status_code == 201:
       channel = response.json()["data"]
       print(f"Channel created: {channel['id']}")
   
   # Test the channel
   test_data = {
       "alert_id": "alert-001",
       "test_data": {
           "device": "Test_Device_01",
           "value": "85.6",
           "timestamp": "2024-01-15T12:00:00Z"
       }
   }
   
   test_response = requests.post(
       f"{base_url}/channels/{channel['id']}/test",
       headers=headers,
       json=test_data
   )

JavaScript Example
^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   const apiKey = "your_api_key";
   const baseUrl = "https://api.univagateway.com/v1";
   
   const headers = {
       "Authorization": `Bearer ${apiKey}`,
       "Content-Type": "application/json"
   };
   
   // Deliver an alert through channel
   fetch(`${baseUrl}/channels/mqtt-primary/deliver`, {
       method: "POST",
       headers: headers,
       body: JSON.stringify({
           alert_id: "alert-001",
           alert_data: {
               device: "Crane_01",
               value: "85.6",
               unit: "°C",
               timestamp: new Date().toISOString(),
               severity: "CRITICAL"
           },
           options: {
               priority: "high",
               retry_on_failure: true
           }
       })
   })
   .then(response => response.json())
   .then(data => console.log(data.data.delivery_id));

Support
-------

For API support, contact:

* **Email:** api-support@univagateway.com
* **Documentation:** https://docs.univagateway.com/api/channels
* **Status Page:** https://status.univagateway.com

Channel Types and Configuration
-------------------------------

Supported Channel Types
^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Supported Channel Types
   :widths: 25 50 25
   :header-rows: 1

   * - Type
     - Description
     - Required Fields
   * - **MQTT**
     - Publish alerts to MQTT broker
     - broker_url, topic
   * - **Email**
     - Send email notifications
     - smtp_server, port, from_email, to_emails
   * - **SMS**
     - Send SMS messages
     - provider, from_number, to_numbers
   * - **Webhook**
     - HTTP callbacks to custom endpoints
     - endpoints (array)
   * - **Push**
     - Mobile push notifications
     - provider, app_id, api_key

MQTT Configuration
^^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "broker_url": "mqtt.server.com:1883",
     "topic": "factory/alerts",
     "username": "univa_gateway",
     "password": "secure_password",
     "qos": 1,
     "retain": false,
     "client_id": "univa_gateway_01",
     "clean_session": true,
     "keepalive": 60
   }

Email Configuration
^^^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "smtp_server": "smtp.gmail.com",
     "port": 587,
     "use_tls": true,
     "username": "alerts@factory.com",
     "password": "secure_password",
     "from_email": "alerts@factory.com",
     "from_name": "Factory Alert System",
     "to_emails": ["ops-team@factory.com", "supervisor@factory.com"],
     "subject_template": "ALERT: {device} - {severity}",
     "cc_emails": [],
     "bcc_emails": []
   }

SMS Configuration
^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "provider": "Twilio",
     "api_key": "SK-XXXXXXXXXXXXXXXX",
     "account_sid": "AC-XXXXXXXXXXXXXXXX",
     "auth_token": "********",
     "from_number": "+1234567890",
     "to_numbers": ["+919876543210", "+971501234567"],
     "message_template": "ALERT: {device} {value}{unit} at {timestamp}"
   }

Webhook Configuration
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: json

   {
     "endpoints": [
       {
         "url": "https://webhook.site/abcdef",
         "method": "POST",
         "headers": {
           "Content-Type": "application/json",
           "Authorization": "Bearer token-123"
         },
         "timeout": 5000,
         "retry_policy": {
           "max_retries": 3,
           "retry_delay": 1000,
           "backoff_factor": 2
         },
         "payload_template": {
           "alert_id": "{alert_id}",
           "device": "{device}",
           "value": "{value}",
           "unit": "{unit}",
           "timestamp": "{timestamp}",
           "severity": "{severity}"
         }
       }
     ],
     "concurrent_requests": 5,
     "max_queue_size": 1000
   }

Security Best Practices
-----------------------

1. **Credential Management:**
   - Store API keys and passwords securely
   - Use environment variables for sensitive data
   - Rotate credentials regularly
   - Implement least privilege access

2. **Connection Security:**
   - Use TLS/SSL for all external communications
   - Validate SSL certificates
   - Use VPN for internal network channels
   - Implement firewall rules for channel access

3. **Data Protection:**
   - Encrypt sensitive data at rest and in transit
   - Implement data masking for logs
   - Use secure protocols (MQTT over TLS, SMTP over TLS)
   - Validate input data to prevent injection attacks

Performance Considerations
--------------------------

1. **Channel Capacity:**
   - MQTT: 1000 messages/second per channel
   - Email: 100 messages/minute per channel
   - SMS: 50 messages/second per channel
   - Webhook: 500 requests/second per channel

2. **Queue Management:**
   - Default queue size: 10,000 messages
   - Automatic retry for failed deliveries
   - Priority-based queuing
   - Dead letter queue for undeliverable messages

3. **Monitoring Thresholds:**
   - Response time > 5s: Warning
   - Success rate < 95%: Warning
   - Queue depth > 80%: Warning
   - Connection failures > 3: Critical

Troubleshooting
---------------

Common Issues
^^^^^^^^^^^^^

1. **Channel Connection Failures:**
   - Check network connectivity
   - Verify credentials
   - Test endpoint accessibility
   - Check firewall rules

2. **Delivery Failures:**
   - Review error logs
   - Check message format
   - Verify destination addresses
   - Test with simple message

3. **Performance Issues:**
   - Monitor queue depth
   - Check channel throughput
   - Review retry configurations
   - Scale channel capacity

Debug Checklist
^^^^^^^^^^^^^^^

1. **Connection Testing:**
   - Use /channels/{id}/test endpoint
   - Check network connectivity
   - Validate credentials
   - Test with simple message

2. **Message Delivery:**
   - Verify message format
   - Check destination addresses
   - Review delivery logs
   - Monitor delivery status

3. **Performance Monitoring:**
   - Check queue statistics
   - Monitor response times
   - Review error rates
   - Track throughput metrics

Compliance and Regulations
--------------------------

1. **Data Privacy:**
   - GDPR compliance for EU data
   - CCPA compliance for California
   - PIPEDA compliance for Canada
   - Data retention policies

2. **Communication Regulations:**
   - SMS: TCPA compliance (US)
   - SMS: CTIA best practices
   - Email: CAN-SPAM compliance
   - Email: GDPR consent requirements

3. **Industry Standards:**
   - ISO 27001 for information security
   - NIST cybersecurity framework
   - IEC 62443 for industrial security
   - HIPAA for healthcare data (if applicable)
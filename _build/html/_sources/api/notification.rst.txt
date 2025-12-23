Notification Channels API
=========================

This document describes the notification channels page and its related API endpoints for managing alert delivery channels including MQTT, email, SMS, webhook, and push notifications.

Page Route (Frontend)
---------------------

.. http:get:: /notification-channels

   **Description**: Renders the complete notification channels management page with all channel configurations, status dashboard, and testing interface embedded in the HTML.
   
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
        <title>Notification Channels - Univa Gateway</title>
      </head>
      <body>
        <div id="app" data-channels='[...]' data-gateway='{...}' data-statistics='{...}'>
          <!-- Notification channels page with:
               - Channels list table with status and actions
               - Channel status dashboard
               - Testing interface with alert message selection
               - Gateway selection and switching
               - Real-time monitoring
          -->
        </div>
      </body>
      </html>
   
   **How it works**:
   - Server renders the HTML page
   - All channel configurations, gateway info, and statistics are embedded in the page
   - JavaScript reads this data and renders the channels management interface
   - No separate API call needed on initial page load

   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

API Endpoints (Backend)
-----------------------

These endpoints handle all notification channel operations triggered from the page.

Channel Operations
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/notification-channels/channels

   **Description**: Add new notification channel (when user clicks "Add Channel").
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "name": "Primary MQTT Broker",
        "type": "mqtt",
        "enabled": true,
        "config": {
          "broker_url": "mqtt.factory.com:1883",
          "topic": "factory/alerts",
          "username": "univa_gateway",
          "password": "secure_password",
          "qos": 1,
          "retain": false
        }
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "channel": {
          "id": "mqtt-primary",
          "name": "Primary MQTT Broker",
          "type": "mqtt",
          "status": "active",
          "enabled": true,
          "config": {
            "broker_url": "mqtt.factory.com:1883",
            "topic": "factory/alerts",
            "username": "univa_gateway",
            "qos": 1
          }
        }
      }

.. http:put:: /api/notification-channels/channels/{channel_id}

   **Description**: Edit channel configuration (when user clicks "Edit" on a channel).
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
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
        "channel": {
          "id": "mqtt-primary",
          "name": "MQTT Primary Updated",
          "type": "mqtt",
          "status": "active",
          "config": {
            "broker_url": "mqtt.factory.local:1883",
            "topic": "factory/alerts/v2",
            "username": "univa_gateway_v2"
          }
        }
      }

.. http:delete:: /api/notification-channels/channels/{channel_id}

   **Description**: Remove channel (when user clicks delete icon on a channel).
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "channel_id": "mqtt-primary",
        "message": "Channel deleted successfully"
      }

Channel Status Operations
~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:patch:: /api/notification-channels/channels/{channel_id}/status

   **Description**: Enable or disable a channel (when user toggles channel status).
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      PATCH /api/notification-channels/channels/mqtt-primary/status HTTP/1.1
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
        "channel_id": "mqtt-primary",
        "enabled": false,
        "previous_status": true,
        "message": "Channel disabled successfully"
      }

Testing Operations
~~~~~~~~~~~~~~~~~~

.. http:post:: /api/notification-channels/channels/{channel_id}/test

   **Description**: Test channel with alert message (when user clicks "Test MQTT", "Test Email", etc.).
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/mqtt-primary/test HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "channel_id": "mqtt-primary",
        "channel_name": "MQTT Alert Broker",
        "test_time": "2024-01-15T12:00:30Z",
        "result": "success",
        "message": "Test notification sent successfully",
        "details": {
          "destination": "mqtt.factory.com:1883/factory/alerts",
          "message_sent": "🔥 CRITICAL: Test_Device_01 temperature is 85.6°C at 2024-01-15T12:00:00Z",
          "response_time": "0.125s"
        }
      }

.. http:post:: /api/notification-channels/channels/test-bulk

   **Description**: Test multiple channels simultaneously (when user clicks "Test All").
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/test-bulk HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

Gateway Operations
~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/gateways

   **Description**: Get list of available gateways for channel configuration.
   
   **Query Parameters**:
   
   * **status** (optional): Filter by gateway status - ``online``, ``offline``, ``maintenance``
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateways": [
          {
            "id": "Univa-GW-01",
            "name": "Primary Gateway",
            "status": "online",
            "location": "Factory Floor - Zone A",
            "last_seen": "2024-01-15T12:00:00Z",
            "ip_address": "192.168.1.100",
            "channels_count": 5,
            "active_channels": 4
          }
        ],
        "meta": {
          "total": 3,
          "online": 2,
          "offline": 1
        }
      }

.. http:post:: /api/notification-channels/gateways/{gateway_id}/switch

   **Description**: Switch to a different gateway for channel operations (when user changes gateway in dropdown).
   
   **Path Parameters**:
   
   * **gateway_id** (string): Gateway identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "previous_gateway": "Univa-GW-01",
        "current_gateway": "Univa-GW-02",
        "timestamp": "2024-01-15T12:05:00Z",
        "channels_transferred": 5,
        "session_id": "sess-20250115-001",
        "message": "Switched to gateway Univa-GW-02"
      }

Status Dashboard Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/channels/status/dashboard

   **Description**: Get real-time status dashboard for all channels (displayed in UI).
   
   **Query Parameters**:
   
   * **gateway_id** (optional): Filter by gateway ID
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "gateway_id": "Univa-GW-01",
        "timestamp": "2024-01-15T12:00:00Z",
        "total_channels": 5,
        "online_channels": 3,
        "offline_channels": 2,
        "channels": [
          {
            "id": "mqtt-primary",
            "name": "MQTT Alert Broker",
            "type": "mqtt",
            "status": "online",
            "last_heartbeat": "2024-01-15T11:59:45Z",
            "connection_info": "mqtt.factory.com:1883/factory/alerts",
            "message_rate": "2.5 msg/sec",
            "health_score": 98
          }
        ],
        "summary": {
          "all_online": false,
          "health_status": "degraded",
          "recommendations": [
            "Check webhook-primary connectivity",
            "Review SMS channel configuration"
          ]
        }
      }

.. http:get:: /api/notification-channels/channels/{channel_id}/statistics

   **Description**: Get statistics for a specific channel.
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Query Parameters**:
   
   * **start_date** (optional): Start date for statistics (ISO format)
   * **end_date** (optional): End date for statistics (ISO format)
   * **granularity** (optional): Time granularity - ``hour``, ``day``, ``week``, ``month``
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

Alert Integration Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/alerts/available

   **Description**: Get available alert messages for channel testing (populates dropdown in UI).
   
   **Query Parameters**:
   
   * **status** (optional): Filter by alert status - ``active``, ``draft``, ``archived``
   * **severity** (optional): Filter by alert severity - ``info``, ``warning``, ``critical``
   * **limit** (optional): Maximum number of alerts to return (default: ``50``)
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "alerts": [
          {
            "id": "alert-001",
            "name": "High Temperature Alert",
            "message": "🔥 CRITICAL: {device} temperature is {value}{unit} at {timestamp}",
            "description": "Triggers when temperature exceeds threshold",
            "severity": "critical",
            "variables": ["device", "value", "unit", "timestamp"],
            "created_at": "2024-01-10T08:30:00Z",
            "updated_at": "2024-01-15T09:45:00Z"
          }
        ],
        "total": 12,
        "variables_summary": {
          "total_unique_variables": 8,
          "most_common_variables": ["device", "value", "timestamp", "location"]
        }
      }

.. http:post:: /api/notification-channels/alerts/{alert_id}/test-with-variables

   **Description**: Test an alert message with variable substitution for channel testing.
   
   **Path Parameters**:
   
   * **alert_id** (string): Alert identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/alerts/alert-001/test-with-variables HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "variables": {
          "device": "Crane_01",
          "value": "85.6",
          "unit": "°C",
          "timestamp": "2024-01-15T12:00:00Z",
          "location": "Factory Floor - Zone B",
          "severity": "CRITICAL"
        },
        "channel_types": ["mqtt", "email"]
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "alert_id": "alert-001",
        "original_message": "🔥 CRITICAL: {device} temperature is {value}{unit} at {timestamp}",
        "formatted_message": "🔥 CRITICAL: Crane_01 temperature is 85.6°C at 2024-01-15T12:00:00Z",
        "variables_used": ["device", "value", "unit", "timestamp"],
        "variables_missing": [],
        "channel_results": [
          {
            "channel_type": "mqtt",
            "result": "success",
            "message": "Test message formatted successfully"
          },
          {
            "channel_type": "email",
            "result": "success",
            "message": "Test message formatted successfully"
          }
        ]
      }

Channel Configuration Templates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/channels/templates/{type}

   **Description**: Get configuration template for a channel type (used when adding new channels).
   
   **Path Parameters**:
   
   * **type** (string): Channel type - ``mqtt``, ``email``, ``sms``, ``webhook``, ``push``
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

Configuration Validation Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/notification-channels/channels/validate

   **Description**: Validate channel configuration before saving.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/validate HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
      {
        "type": "mqtt",
        "config": {
          "broker_url": "mqtt.factory.com:1883",
          "topic": "factory/alerts",
          "username": "univa_gateway",
          "password": "********"
        }
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

.. http:post:: /api/notification-channels/channels/{channel_id}/validate-connection

   **Description**: Test connection to channel endpoint.
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

Delivery Operations
~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/notification-channels/channels/{channel_id}/deliver

   **Description**: Deliver an alert through a specific channel.
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/mqtt-primary/deliver HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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

.. http:get:: /api/notification-channels/channels/deliveries

   **Description**: Get delivery history for all channels.
   
   **Query Parameters**:
   
   * **channel_id** (optional): Filter by channel ID
   * **alert_id** (optional): Filter by alert ID
   * **status** (optional): Filter by status - ``sent``, ``delivered``, ``failed``, ``pending``
   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **page** (optional): Page number (default: ``1``)
   * **limit** (optional): Items per page (default: ``50``)
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
        },
        "meta": {
          "count": 2,
          "total": 1250,
          "page": 1,
          "total_pages": 63
        }
      }

Import/Export Operations
~~~~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/channels/export

   **Description**: Export channels to JSON configuration.
   
   **Query Parameters**:
   
   * **format** (optional): Export format - ``json`` (default), ``yaml``
   * **include_secrets** (optional): Include passwords/API keys - ``true``, ``false`` (default: ``false``)
   * **gateway_id** (optional): Filter by gateway ID
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "export_url": "/downloads/channels-export-2025-01-15.json",
        "expires_at": "2024-01-16T12:00:00Z",
        "summary": {
          "channels_exported": 5,
          "secrets_included": false,
          "file_size": "15.8 KB",
          "gateway": "Univa-GW-01"
        }
      }

.. http:post:: /api/notification-channels/channels/import

   **Description**: Import channels from JSON configuration.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
      Content-Type: application/json
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/import HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
          "update_secrets": false,
          "gateway_id": "Univa-GW-01"
        }
      }
   
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
          "channels_created": 1,
          "channels_updated": 0,
          "secrets_updated": 0
        },
        "import_id": "IMP-20250115-002"
      }

Monitoring Operations
~~~~~~~~~~~~~~~~~~~~~

.. http:get:: /api/notification-channels/channels/{channel_id}/monitor

   **Description**: Get real-time monitoring data for a channel.
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Query Parameters**:
   
   * **duration** (optional): Monitoring duration in seconds (default: ``60``)
   * **refresh** (optional): Auto-refresh interval in seconds
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
        },
        "health_metrics": {
          "response_time_avg": "0.125s",
          "success_rate": "99.8%",
          "error_rate": "0.2%",
          "last_error": null
        }
      }

.. http:get:: /api/notification-channels/channels/statistics/overview

   **Description**: Get overview statistics for all channels.
   
   **Query Parameters**:
   
   * **time_range** (optional): Time range - ``today``, ``yesterday``, ``last_7d``, ``last_30d``, ``custom``
   * **start_date** (optional): Custom start date (ISO format)
   * **end_date** (optional): Custom end date (ISO format)
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
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
        },
        "timestamp": "2024-01-15T12:00:00Z"
      }

Webhook Configuration
~~~~~~~~~~~~~~~~~~~~~

.. http:post:: /api/notification-channels/channels/webhook/{channel_id}/configure

   **Description**: Configure webhook endpoints for a webhook channel.
   
   **Path Parameters**:
   
   * **channel_id** (string): Channel identifier
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /api/notification-channels/channels/webhook-primary/configure HTTP/1.1
      Cookie: session_token=<token>
      Content-Type: application/json
      
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
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "channel_id": "webhook-primary",
        "endpoints_configured": 1,
        "test_results": [
          {
            "url": "https://webhook.site/abcdef",
            "status": "success",
            "response_code": 200,
            "response_time": "0.320s"
          }
        ],
        "message": "Webhook endpoints configured successfully"
      }

Route Summary
-------------

.. list-table:: Notification Channels Routes
   :header-rows: 1
   :widths: 20 20 50 10
   
   * - Type
     - Method
     - Route & Purpose
     - Auth
   * - Page
     - GET
     - ``/notification-channels`` - Main channels management page
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/channels`` - Add channel
     - Yes
   * - API
     - PUT
     - ``/api/notification-channels/channels/{id}`` - Edit channel
     - Yes
   * - API
     - DELETE
     - ``/api/notification-channels/channels/{id}`` - Delete channel
     - Yes
   * - API
     - PATCH
     - ``/api/notification-channels/channels/{id}/status`` - Toggle channel status
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/channels/{id}/test`` - Test single channel
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/channels/test-bulk`` - Test multiple channels
     - Yes
   * - API
     - GET
     - ``/api/notification-channels/gateways`` - Get available gateways
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/gateways/{id}/switch`` - Switch gateway
     - Yes
   * - API
     - GET
     - ``/api/notification-channels/channels/status/dashboard`` - Get status dashboard
     - Yes
   * - API
     - GET
     - ``/api/notification-channels/alerts/available`` - Get alerts for testing
     - Yes
   * - API
     - GET
     - ``/api/notification-channels/channels/export`` - Export configuration
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/channels/import`` - Import configuration
     - Yes
   * - API
     - GET
     - ``/api/notification-channels/channels/{id}/monitor`` - Real-time monitoring
     - Yes
   * - API
     - POST
     - ``/api/notification-channels/channels/{id}/deliver`` - Deliver alert through channel
     - Yes

Complete User Flow
------------------

1. **User goes to page**: ``GET /notification-channels``
   - Server renders HTML with embedded channel data
   - Page shows channels table, status dashboard, testing interface

2. **User adds channel**: 
   - Clicks "Add Channel" → opens modal with channel type selection
   - Fills configuration form → ``POST /api/notification-channels/channels``
   - New channel appears in table with status indicator

3. **User tests channel**:
   - Selects alert message from dropdown (populated from ``GET /api/notification-channels/alerts/available``)
   - Clicks "Test MQTT" → ``POST /api/notification-channels/channels/mqtt-primary/test``
   - Shows test results with formatted alert message

4. **User switches gateway**:
   - Selects different gateway from dropdown → ``POST /api/notification-channels/gateways/{id}/switch``
   - Channel configurations switch to selected gateway context

5. **User monitors channels**:
   - Status dashboard auto-updates via ``GET /api/notification-channels/channels/status/dashboard``
   - Real-time statistics show message rates and health scores

6. **User exports configuration**:
   - Clicks export option → ``GET /api/notification-channels/channels/export``
   - Downloads JSON configuration file

7. **User imports configuration**:
   - Uploads JSON file → ``POST /api/notification-channels/channels/import``
   - Channels imported and configured automatically

8. **User delivers alert**:
   - Alert triggered from rules engine → ``POST /api/notification-channels/channels/{id}/deliver``
   - Alert delivered through configured channels with delivery tracking

Error Codes
-----------

.. list-table:: Notification Channels Error Codes
   :widths: 25 75
   :header-rows: 1

   * - Error Code
     - Description
   * - **CHANNEL_001**
     - Channel not found
   * - **CHANNEL_002**
     - Invalid channel configuration
   * - **CHANNEL_003**
     - Channel already exists
   * - **CHANNEL_004**
     - Channel connection test failed
   * - **CHANNEL_005**
     - Channel delivery failed
   * - **CHANNEL_006**
     - Invalid channel type
   * - **CHANNEL_007**
     - Gateway not found
   * - **CHANNEL_008**
     - Cannot switch gateway - channels in use
   * - **CHANNEL_009**
     - Alert not found for testing
   * - **CHANNEL_010**
     - Missing required variables in alert
   * - **CHANNEL_011**
     - Import configuration invalid
   * - **CHANNEL_012**
     - Export generation failed
   * - **CHANNEL_013**
     - Channel offline - cannot deliver
   * - **CHANNEL_014**
     - Rate limit exceeded for channel
   * - **CHANNEL_015**
     - Webhook configuration invalid

Key Features
------------

- **Multi-channel support**: MQTT, email, SMS, webhook, push notifications
- **Gateway switching**: Manage channels across multiple Univa Gateways
- **Real-time dashboard**: Live status monitoring with health scores
- **Alert integration**: Seamless integration with alert messages
- **Variable substitution**: Automatic variable replacement in alert messages
- **Testing interface**: Comprehensive channel testing with sample data
- **Import/export**: Full configuration backup and restore
- **Delivery tracking**: Detailed delivery history and statistics
- **Connection validation**: Pre-flight checks for channel configurations
- **Bulk operations**: Test and manage multiple channels simultaneously
- **Webhook configuration**: Advanced webhook endpoint management
- **Security**: Secure credential storage and transmission

Channel Types Configuration
---------------------------

MQTT Configuration Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

Email Configuration Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

SMS Configuration Template
~~~~~~~~~~~~~~~~~~~~~~~~~~

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

Webhook Configuration Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

Performance Guidelines
----------------------

- **MQTT Channels**: Supports up to 1000 messages/second per channel
- **Email Channels**: Limit to 100 messages/minute to avoid spam filters
- **SMS Channels**: 50 messages/second maximum throughput
- **Webhook Channels**: 500 requests/second with proper queue management
- **Connection Pooling**: Maintain persistent connections for frequent channels
- **Queue Management**: Automatic retry with exponential backoff for failed deliveries
- **Monitoring**: Real-time health checks every 30 seconds for active channels
- **Load Balancing**: Distribute load across multiple gateways for high-volume deployments

Security Considerations
-----------------------

1. **Credential Security**:
   - Passwords and API keys encrypted at rest
   - TLS/SSL required for all external communications
   - Credentials masked in logs and UI displays
   - Regular rotation of authentication tokens

2. **Channel Security**:
   - MQTT: Require TLS with certificate validation
   - Email: Enforce TLS for SMTP connections
   - Webhook: Validate SSL certificates for HTTPS endpoints
   - SMS: Use secure API tokens with limited permissions

3. **Access Control**:
   - Role-based access to channel configurations
   - Audit logging for all configuration changes
   - IP whitelisting for critical channels
   - Rate limiting per channel and per user

Troubleshooting
---------------

Common Issues and Solutions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Channel Connection Failures**:
   - Verify network connectivity to endpoint
   - Check credentials and authentication
   - Validate SSL certificates
   - Test with simple connection test

2. **Message Delivery Failures**:
   - Review message format and size limits
   - Check destination addresses/URLs
   - Verify channel rate limits
   - Examine delivery logs for specific errors

3. **Performance Issues**:
   - Monitor queue depth and processing rates
   - Check channel throughput statistics
   - Review retry configurations
   - Scale channel capacity as needed

4. **Configuration Issues**:
   - Use validation endpoint before saving
   - Test connection with sample data
   - Verify all required fields are populated
   - Check for syntax errors in templates

Debug Checklist
~~~~~~~~~~~~~~~

1. **Connection Testing**:
   - Use ``/channels/{id}/validate-connection`` endpoint
   - Check network connectivity
   - Validate credentials
   - Test with simple message

2. **Message Delivery**:
   - Verify message format
   - Check destination addresses
   - Review delivery logs
   - Monitor delivery status

3. **Performance Monitoring**:
   - Check queue statistics
   - Monitor response times
   - Review error rates
   - Track throughput metrics

Webhooks Integration
--------------------

Channel Status Change Webhook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a channel status changes, the system sends:

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
~~~~~~~~~~~~~~~~~~~~~~~

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

Gateway Switch Webhook
~~~~~~~~~~~~~~~~~~~~~~

When gateway context is switched:

.. code-block:: json

   {
     "event": "gateway.switched",
     "timestamp": "2024-01-15T12:05:00Z",
     "data": {
       "previous_gateway": "Univa-GW-01",
       "current_gateway": "Univa-GW-02",
       "user": "admin@factory.com",
       "channels_transferred": 5,
       "session_id": "sess-20250115-001"
     }
   }


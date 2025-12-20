CraneIQ Pro API
===============

Purpose
-------

The CraneIQ Pro API provides comprehensive management for crane safety systems, including anti-collision systems, zoning, load monitoring, health monitoring, and IoT data buffering. It enables configuration and real-time monitoring of industrial crane safety parameters.

Base URL
--------

.. code-block:: text

   https://api.univagateway.com/v1/craneiq

Authentication
--------------

All API requests require authentication via API key.

**Headers:**

.. code-block:: http

   Authorization: Bearer <api_key>
   Content-Type: application/json

Endpoints
---------

Crane Profile Management
~~~~~~~~~~~~~~~~~~~~~~~~

Get All Cranes
^^^^^^^^^^^^^^

.. http:get:: /cranes

   Retrieve all configured cranes.

   **Query Parameters:**

   * **status** (optional): Filter by status - ``online``, ``offline``, ``maintenance``
   * **type** (optional): Filter by crane type - ``double_girder``, ``single_girder``, ``gantry``, ``jib``, ``tower``, ``rtg``, ``portal``
   * **site** (optional): Filter by site location
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
            "id": "CRANE-01",
            "name": "Double-Girder Crane 01",
            "type": "double_girder",
            "status": "online",
            "capacity": 25000,
            "unit": "kg",
            "dimensions": {
              "span": 15.0,
              "travel": 45.0,
              "height": 12.0,
              "units": "meters"
            },
            "operating_mode": "manual",
            "location": "Factory Floor - Zone B",
            "gateway": "Univa-GW-01",
            "last_seen": "2024-01-15T10:30:00Z",
            "health_score": 95.5,
            "created_at": "2024-01-10T08:30:00Z",
            "updated_at": "2024-01-15T09:45:00Z"
          }
        ],
        "meta": {
          "count": 2,
          "total": 12,
          "online": 10,
          "offline": 2,
          "page": 1,
          "total_pages": 6
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **401 Unauthorized**: Invalid or missing API key
   * **500 Internal Server Error**: Server error

Get Single Crane
^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}

   Retrieve a specific crane by ID.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "CRANE-01",
          "name": "Double-Girder Crane 01",
          "type": "double_girder",
          "status": "online",
          "capacity": 25000,
          "unit": "kg",
          "dimensions": {
            "span": 15.0,
            "travel": 45.0,
            "height": 12.0,
            "units": "meters"
          },
          "operating_mode": "manual",
          "operating_limits": {
            "max_speed_x": 2.5,
            "max_speed_y": 1.8,
            "max_speed_z": 0.8,
            "units": "m/s"
          },
          "configuration": {
            "load_limiter_enabled": true,
            "acs_enabled": true,
            "zoning_enabled": true,
            "health_monitoring_enabled": true
          },
          "location": {
            "site": "Factory Floor - Zone B",
            "coordinates": {
              "x": 125.5,
              "y": 230.8,
              "z": 0.0
            },
            "area": "Manufacturing Area"
          },
          "gateway": "Univa-GW-01",
          "last_seen": "2024-01-15T10:30:00Z",
          "health_score": 95.5,
          "statistics": {
            "uptime_24h": 99.8,
            "load_cycles_today": 45,
            "distance_traveled_today": 1250.5,
            "energy_consumed_today": 245.8
          },
          "created_at": "2024-01-10T08:30:00Z",
          "updated_at": "2024-01-15T09:45:00Z"
        }
      }

   **Status Codes:**

   * **200 OK**: Successful retrieval
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Create Crane Profile
^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes

   Create a new crane profile.

   **Request:**

   .. sourcecode:: http

      POST /cranes HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Gantry Crane 03",
        "type": "gantry",
        "capacity": 32000,
        "unit": "kg",
        "dimensions": {
          "span": 20.0,
          "travel": 50.0,
          "height": 10.0,
          "units": "meters"
        },
        "operating_mode": "manual",
        "location": {
          "site": "Loading Dock A",
          "coordinates": {
            "x": 300.0,
            "y": 150.0,
            "z": 0.0
          }
        },
        "gateway": "Univa-GW-01",
        "configuration": {
          "load_limiter_enabled": true,
          "acs_enabled": true,
          "zoning_enabled": false,
          "health_monitoring_enabled": true
        }
      }

   **Required Fields:**

   * **name** (string): Name of the crane
   * **type** (string): Crane type
   * **capacity** (number): Maximum load capacity
   * **dimensions** (object): Crane dimensions
   * **location** (object): Crane location

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "CRANE-13",
          "name": "Gantry Crane 03",
          "type": "gantry",
          "status": "offline",
          "capacity": 32000,
          "unit": "kg",
          "dimensions": {
            "span": 20.0,
            "travel": 50.0,
            "height": 10.0,
            "units": "meters"
          },
          "operating_mode": "manual",
          "location": {
            "site": "Loading Dock A",
            "coordinates": {
              "x": 300.0,
              "y": 150.0,
              "z": 0.0
            },
            "area": "Loading Dock A"
          },
          "gateway": "Univa-GW-01",
          "last_seen": null,
          "health_score": null,
          "created_at": "2024-01-15T11:30:00Z",
          "updated_at": "2024-01-15T11:30:00Z"
        },
        "message": "Crane profile created successfully"
      }

   **Status Codes:**

   * **201 Created**: Crane created successfully
   * **400 Bad Request**: Missing or invalid fields
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Crane with same name already exists

Update Crane Profile
^^^^^^^^^^^^^^^^^^^^

.. http:put:: /cranes/{id}

   Update an existing crane.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      PUT /cranes/CRANE-01 HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Double-Girder Crane 01 - Updated",
        "capacity": 28000,
        "operating_mode": "semi_auto",
        "configuration": {
          "load_limiter_enabled": true,
          "acs_enabled": true,
          "zoning_enabled": true,
          "health_monitoring_enabled": true
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "CRANE-01",
          "name": "Double-Girder Crane 01 - Updated",
          "type": "double_girder",
          "status": "online",
          "capacity": 28000,
          "unit": "kg",
          "dimensions": {
            "span": 15.0,
            "travel": 45.0,
            "height": 12.0,
            "units": "meters"
          },
          "operating_mode": "semi_auto",
          "configuration": {
            "load_limiter_enabled": true,
            "acs_enabled": true,
            "zoning_enabled": true,
            "health_monitoring_enabled": true
          },
          "location": {
            "site": "Factory Floor - Zone B",
            "coordinates": {
              "x": 125.5,
              "y": 230.8,
              "z": 0.0
            },
            "area": "Manufacturing Area"
          },
          "gateway": "Univa-GW-01",
          "last_seen": "2024-01-15T10:30:00Z",
          "health_score": 95.5,
          "updated_at": "2024-01-15T11:45:00Z"
        },
        "message": "Crane profile updated successfully"
      }

   **Status Codes:**

   * **200 OK**: Crane updated successfully
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid fields
   * **401 Unauthorized**: Invalid or missing API key

Update Operating Mode
^^^^^^^^^^^^^^^^^^^^^

.. http:patch:: /cranes/{id}/operating-mode

   Update crane operating mode.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      PATCH /cranes/CRANE-01/operating-mode HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "operating_mode": "auto",
        "reason": "Scheduled automation test",
        "operator": "admin@factory.com"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "id": "CRANE-01",
          "previous_mode": "semi_auto",
          "new_mode": "auto",
          "timestamp": "2024-01-15T12:00:00Z",
          "operator": "admin@factory.com"
        },
        "message": "Operating mode changed to auto"
      }

   **Status Codes:**

   * **200 OK**: Mode updated successfully
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid mode or missing authorization
   * **401 Unauthorized**: Invalid or missing API key

Delete Crane Profile
^^^^^^^^^^^^^^^^^^^^

.. http:delete:: /cranes/{id}

   Delete a crane profile.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "message": "Crane profile deleted successfully"
      }

   **Status Codes:**

   * **200 OK**: Crane deleted successfully
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Cannot delete - crane has active configurations

Load Limiter Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

Get Load Limiter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/load-limiter

   Get load limiter configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "settings": {
            "rated_load": 25000,
            "unit": "kg",
            "swl_percentage": 100,
            "warning_threshold": 90,
            "cutoff_threshold": 110,
            "warning_threshold_kg": 22500,
            "cutoff_threshold_kg": 27500
          },
          "sensor": {
            "type": "load_cell",
            "model": "LC-4500",
            "signal_type": "4-20mA",
            "calibration_date": "2024-01-10",
            "next_calibration": "2024-07-10"
          },
          "features": {
            "tilt_compensation": true,
            "shock_load_detection": true,
            "swing_detection": true,
            "overload_override": false
          },
          "alarms": {
            "warning_level": 90,
            "critical_level": 110,
            "shock_load_limit": 150,
            "shock_duration": 500
          },
          "statistics": {
            "max_load_recorded": 21850,
            "overload_events_24h": 2,
            "warning_events_24h": 15,
            "last_overload": "2024-01-15T08:30:00Z"
          },
          "last_updated": "2024-01-15T10:45:00Z"
        }
      }

   **Status Codes:**

   * **200 OK**: Configuration retrieved
   * **404 Not Found**: Crane not found or load limiter not configured
   * **401 Unauthorized**: Invalid or missing API key

Update Load Limiter Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:put:: /cranes/{id}/load-limiter

   Update load limiter configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      PUT /cranes/CRANE-01/load-limiter HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "enabled": true,
        "settings": {
          "rated_load": 28000,
          "swl_percentage": 100,
          "warning_threshold": 85,
          "cutoff_threshold": 105
        },
        "features": {
          "tilt_compensation": true,
          "shock_load_detection": true,
          "swing_detection": false,
          "overload_override": false
        },
        "alarms": {
          "warning_level": 85,
          "critical_level": 105,
          "shock_load_limit": 140,
          "shock_duration": 500
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "settings": {
            "rated_load": 28000,
            "unit": "kg",
            "swl_percentage": 100,
            "warning_threshold": 85,
            "cutoff_threshold": 105,
            "warning_threshold_kg": 23800,
            "cutoff_threshold_kg": 29400
          },
          "features": {
            "tilt_compensation": true,
            "shock_load_detection": true,
            "swing_detection": false,
            "overload_override": false
          },
          "alarms": {
            "warning_level": 85,
            "critical_level": 105,
            "shock_load_limit": 140,
            "shock_duration": 500
          },
          "last_updated": "2024-01-15T12:00:00Z"
        },
        "message": "Load limiter configuration updated"
      }

   **Status Codes:**

   * **200 OK**: Configuration updated
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid configuration
   * **401 Unauthorized**: Invalid or missing API key

Calibrate Load Limiter
^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/load-limiter/calibration

   Calibrate load limiter.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/load-limiter/calibration HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "calibration_type": "zero_calibration",
        "reference_value": 0,
        "operator": "tech-001",
        "notes": "Zero point calibration after maintenance"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "calibration_id": "CAL-20250115-001",
          "crane_id": "CRANE-01",
          "type": "zero_calibration",
          "status": "completed",
          "previous_offset": 25.5,
          "new_offset": 0.0,
          "operator": "tech-001",
          "timestamp": "2024-01-15T12:05:00Z",
          "next_calibration": "2024-07-15T12:05:00Z"
        },
        "message": "Load limiter calibrated successfully"
      }

   **Status Codes:**

   * **200 OK**: Calibration completed
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Crane in operation
   * **401 Unauthorized**: Invalid or missing API key

Get Load Limiter History
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/load-limiter/history

   Get load limiter history.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Query Parameters:**

   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **event_type** (optional): Filter by event type - ``overload``, ``warning``, ``calibration``, ``shock_load``
   * **limit** (optional): Maximum records (default: ``50``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "events": [
            {
              "event_id": "LLE-001",
              "timestamp": "2024-01-15T08:30:00Z",
              "event_type": "overload",
              "load_value": 26800,
              "load_percentage": 107.2,
              "duration": 2.5,
              "action_taken": "motion_stopped",
              "operator": "auto"
            }
          ],
          "summary": {
            "total_events": 18,
            "overload_events": 2,
            "warning_events": 15,
            "shock_load_events": 1,
            "avg_load_percentage": 72.5
          }
        }
      }

   **Status Codes:**

   * **200 OK**: History retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Anti-Collision System (ACS)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get ACS Configuration
^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/acs

   Get anti-collision system configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "geometry": {
            "boom_length": 30.0,
            "jib_angle": 45.0,
            "units": "meters"
          },
          "sensor": {
            "type": "laser_lidar",
            "model": "SICK TiM7xx",
            "range": 50.0,
            "accuracy": 0.02,
            "scan_frequency": 15
          },
          "communication": {
            "protocol": "wireless_rf",
            "frequency": 2.4,
            "network_id": "ACS-NET-01"
          },
          "distances": {
            "warning_distance": 6.0,
            "slow_distance": 4.0,
            "stop_distance": 2.0,
            "units": "meters"
          },
          "paired_cranes": [
            {
              "crane_id": "CRANE-02",
              "crane_name": "RTG Crane 01",
              "status": "paired",
              "last_seen": "2024-01-15T10:25:00Z"
            }
          ],
          "statistics": {
            "collision_warnings_24h": 12,
            "slow_downs_24h": 5,
            "emergency_stops_24h": 1,
            "last_warning": "2024-01-15T10:18:00Z"
          },
          "last_updated": "2024-01-15T10:30:00Z"
        }
      }

   **Status Codes:**

   * **200 OK**: Configuration retrieved
   * **404 Not Found**: Crane not found or ACS not configured
   * **401 Unauthorized**: Invalid or missing API key

Update ACS Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:put:: /cranes/{id}/acs

   Update ACS configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      PUT /cranes/CRANE-01/acs HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "enabled": true,
        "geometry": {
          "boom_length": 32.0,
          "jib_angle": 42.5
        },
        "sensor": {
          "type": "laser_lidar",
          "range": 60.0,
          "scan_frequency": 20
        },
        "distances": {
          "warning_distance": 8.0,
          "slow_distance": 5.0,
          "stop_distance": 2.5
        },
        "paired_cranes": ["CRANE-02", "CRANE-03", "CRANE-04"]
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "geometry": {
            "boom_length": 32.0,
            "jib_angle": 42.5,
            "units": "meters"
          },
          "sensor": {
            "type": "laser_lidar",
            "range": 60.0,
            "scan_frequency": 20
          },
          "distances": {
            "warning_distance": 8.0,
            "slow_distance": 5.0,
            "stop_distance": 2.5,
            "units": "meters"
          },
          "paired_cranes": [
            "CRANE-02",
            "CRANE-03",
            "CRANE-04"
          ],
          "last_updated": "2024-01-15T12:15:00Z"
        },
        "message": "ACS configuration updated"
      }

   **Status Codes:**

   * **200 OK**: Configuration updated
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid configuration
   * **401 Unauthorized**: Invalid or missing API key

Pair Cranes for ACS
^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/acs/pair

   Pair with another crane for ACS.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/acs/pair HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "target_crane_id": "CRANE-04",
        "pairing_key": "ACS-PAIR-1234"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "paired_crane": {
            "crane_id": "CRANE-04",
            "crane_name": "Gantry Crane 02",
            "status": "paired",
            "pairing_timestamp": "2024-01-15T12:20:00Z",
            "pairing_id": "PAIR-20250115-001"
          }
        },
        "message": "Cranes paired successfully for ACS"
      }

   **Status Codes:**

   * **200 OK**: Pairing successful
   * **404 Not Found**: Crane or target crane not found
   * **400 Bad Request**: Invalid pairing key or crane incompatible
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Already paired

Get Real-time ACS Status
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/acs/status

   Get real-time ACS status.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "timestamp": "2024-01-15T12:30:00Z",
          "system_status": "operational",
          "sensor_status": "active",
          "nearest_objects": [
            {
              "id": "CRANE-02",
              "name": "RTG Crane 01",
              "distance": 8.5,
              "status": "warning",
              "speed": 0.8,
              "direction": "approaching"
            }
          ],
          "active_alerts": [],
          "safety_zones": {
            "current_zone": "safe",
            "nearest_zone": {
              "id": "ZONE-003",
              "name": "Slow Zone - Mezzanine",
              "distance": 15.8,
              "type": "slow_zone"
            }
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Status retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Zoning Configuration
~~~~~~~~~~~~~~~~~~~~

Get Configured Zones
^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/zones

   Get configured zones for a crane.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "zones": [
            {
              "id": "ZONE-001",
              "name": "Chemical Storage",
              "type": "no_go",
              "priority": "high",
              "coordinates": {
                "x1": 10.0,
                "y1": 20.0,
                "x2": 30.0,
                "y2": 40.0,
                "height": 8.0,
                "units": "meters"
              },
              "actions": {
                "violation_action": "hard_stop",
                "speed_limit": 0.0,
                "override_required": true
              },
              "created_at": "2024-01-10T08:30:00Z",
              "updated_at": "2024-01-15T09:45:00Z"
            }
          ],
          "violation_action": "hard_stop",
          "override_mode": {
            "enabled": true,
            "auth_required": true,
            "max_override_time": 300,
            "audit_logging": true
          },
          "statistics": {
            "total_zones": 3,
            "active_zones": 3,
            "violations_24h": 3,
            "overrides_24h": 1
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Zones retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Create New Zone
^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/zones

   Create a new zone.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/zones HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "name": "Loading Bay Entry",
        "type": "warning_only",
        "priority": "low",
        "coordinates": {
          "x1": 100.0,
          "y1": 150.0,
          "x2": 120.0,
          "y2": 180.0,
          "height": 10.0,
          "units": "meters"
        },
        "actions": {
          "violation_action": "alarm_only",
          "speed_limit": 1.0,
          "override_required": false
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "zone_id": "ZONE-004",
          "crane_id": "CRANE-01",
          "name": "Loading Bay Entry",
          "type": "warning_only",
          "priority": "low",
          "coordinates": {
            "x1": 100.0,
            "y1": 150.0,
            "x2": 120.0,
            "y2": 180.0,
            "height": 10.0,
            "units": "meters"
          },
          "actions": {
            "violation_action": "alarm_only",
            "speed_limit": 1.0,
            "override_required": false
          },
          "created_at": "2024-01-15T12:30:00Z",
          "updated_at": "2024-01-15T12:30:00Z"
        },
        "message": "Zone created successfully"
      }

   **Status Codes:**

   * **201 Created**: Zone created
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid zone data
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Overlapping zone exists

Delete Zone
^^^^^^^^^^^

.. http:delete:: /cranes/{id}/zones/{zoneId}

   Delete a zone.

   **Path Parameters:**

   * **id** (string): Crane ID
   * **zoneId** (string): Zone ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "message": "Zone deleted successfully"
      }

   **Status Codes:**

   * **200 OK**: Zone deleted
   * **404 Not Found**: Crane or zone not found
   * **401 Unauthorized**: Invalid or missing API key

Health Monitoring
~~~~~~~~~~~~~~~~~

Get Health Monitoring Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/health

   Get health monitoring configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "enabled": true,
          "monitoring_systems": {
            "load_monitoring": true,
            "motor_health": true,
            "brake_monitoring": true,
            "structural_health": false
          },
          "load_monitoring": {
            "overload_warning": 90,
            "overload_cutoff": 110,
            "shock_load_detection": true,
            "swing_detection": true,
            "units": "percentage"
          },
          "motor_health": {
            "monitored_motors": [
              {
                "id": "MH-001",
                "name": "Hoist Motor",
                "type": "lt_motor",
                "temperature_limit": 75,
                "current_limit": 120,
                "vibration_limit": 4.2
              }
            ]
          },
          "brake_monitoring": {
            "enabled": true,
            "coil_current_range": {
              "min": 2.5,
              "max": 3.5,
              "unit": "A"
            },
            "wear_threshold": 80,
            "release_time_limit": 120,
            "slippage_detection": true
          },
          "statistics": {
            "health_score": 95.5,
            "motor_health": 98.2,
            "brake_health": 92.8,
            "load_system_health": 96.4,
            "last_updated": "2024-01-15T10:30:00Z"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Configuration retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Get Real-time Health Status
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/health/status

   Get real-time health status.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "timestamp": "2024-01-15T12:30:00Z",
          "overall_health": 95.5,
          "systems": [
            {
              "system": "load_monitoring",
              "status": "normal",
              "value": 45.2,
              "unit": "%",
              "alert": null
            },
            {
              "system": "hoist_motor",
              "status": "normal",
              "value": 65.0,
              "unit": "°C",
              "alert": null
            }
          ],
          "active_alerts": [
            {
              "alert_id": "HAL-001",
              "system": "trolley_motor",
              "severity": "warning",
              "message": "Temperature approaching limit (78.5°C)",
              "timestamp": "2024-01-15T12:25:00Z",
              "acknowledged": false
            }
          ],
          "recommendations": [
            "Monitor trolley motor temperature",
            "Schedule maintenance check for motor cooling system"
          ]
        }
      }

   **Status Codes:**

   * **200 OK**: Status retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

IoT Gateway Data Buffer (Dataloggers)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get Configured Dataloggers
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/dataloggers

   Get configured dataloggers.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Query Parameters:**

   * **status** (optional): Filter by status - ``active``, ``paused``, ``error``
   * **priority** (optional): Filter by priority - ``critical``, ``high``, ``medium``, ``low``
   * **device_type** (optional): Filter by device type

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "dataloggers": [
            {
              "id": "DLG-001",
              "device": {
                "id": "load_cell",
                "name": "Load Cell System",
                "device_id": "LC-001"
              },
              "tag": {
                "id": "load_actual",
                "name": "Load_Actual",
                "address": "%MW100",
                "description": "Actual load weight",
                "type": "Float32",
                "unit": "kg"
              },
              "priority": "critical",
              "condition": "periodic",
              "condition_details": "Every 5 seconds",
              "storage": {
                "size": 100,
                "unit": "MB",
                "total_mb": 100,
                "estimated_duration": "48 hours"
              },
              "sampling": {
                "rate": 5000,
                "display": "5 sec",
                "data_format": "compressed"
              },
              "status": "active",
              "created": "2024-01-15T10:30:00Z",
              "statistics": {
                "samples_collected": 12500,
                "data_volume": "4.8 MB",
                "last_sample": "2024-01-15T12:30:00Z"
              }
            }
          ],
          "summary": {
            "total_loggers": 6,
            "active_loggers": 5,
            "paused_loggers": 1,
            "total_storage": "450 MB",
            "data_rate": "15.2 MB/hour",
            "buffer_health": "good"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Dataloggers retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Create New Datalogger
^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/dataloggers

   Create a new datalogger.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/dataloggers HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "device_id": "hoist_motor",
        "tag_id": "motor_temp",
        "priority": "high",
        "condition": "threshold",
        "condition_config": {
          "type": "greater_than",
          "value": 70,
          "unit": "°C"
        },
        "storage": {
          "size": 200,
          "unit": "MB"
        },
        "sampling": {
          "rate": 10000,
          "data_format": "compressed"
        }
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "logger_id": "DLG-007",
          "crane_id": "CRANE-01",
          "device": {
            "id": "hoist_motor",
            "name": "Hoist Motor Drive",
            "device_id": "HM-001"
          },
          "tag": {
            "id": "motor_temp",
            "name": "Motor_Temperature",
            "address": "%MW300",
            "description": "Motor winding temperature",
            "type": "Float32",
            "unit": "°C"
          },
          "priority": "high",
          "condition": "threshold",
          "condition_details": "> 70°C",
          "storage": {
            "size": 200,
            "unit": "MB",
            "total_mb": 200,
            "estimated_duration": "96 hours"
          },
          "sampling": {
            "rate": 10000,
            "display": "10 sec",
            "data_format": "compressed"
          },
          "status": "active",
          "created": "2024-01-15T12:45:00Z"
        },
        "message": "Datalogger created successfully"
      }

   **Status Codes:**

   * **201 Created**: Datalogger created
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid configuration
   * **401 Unauthorized**: Invalid or missing API key
   * **409 Conflict**: Datalogger for tag already exists

Update Datalogger Status
^^^^^^^^^^^^^^^^^^^^^^^^

.. http:patch:: /cranes/{id}/dataloggers/{loggerId}/status

   Update datalogger status.

   **Path Parameters:**

   * **id** (string): Crane ID
   * **loggerId** (string): Datalogger ID

   **Request:**

   .. sourcecode:: http

      PATCH /cranes/CRANE-01/dataloggers/DLG-001/status HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "status": "paused",
        "reason": "Scheduled maintenance"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "logger_id": "DLG-001",
          "crane_id": "CRANE-01",
          "previous_status": "active",
          "new_status": "paused",
          "reason": "Scheduled maintenance",
          "timestamp": "2024-01-15T12:50:00Z"
        },
        "message": "Datalogger paused"
      }

   **Status Codes:**

   * **200 OK**: Status updated
   * **404 Not Found**: Crane or logger not found
   * **401 Unauthorized**: Invalid or missing API key

Get Logged Data
^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/dataloggers/{loggerId}/data

   Get logged data from datalogger.

   **Path Parameters:**

   * **id** (string): Crane ID
   * **loggerId** (string): Datalogger ID

   **Query Parameters:**

   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **limit** (optional): Maximum records (default: ``1000``)
   * **format** (optional): Output format - ``json`` (default), ``csv``

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "logger_id": "DLG-001",
          "crane_id": "CRANE-01",
          "device": "Load Cell System",
          "tag": "Load_Actual",
          "period": {
            "start": "2024-01-15T00:00:00Z",
            "end": "2024-01-15T12:30:00Z"
          },
          "samples": [
            {
              "timestamp": "2024-01-15T00:00:05Z",
              "value": 12500.5,
              "unit": "kg",
              "quality": "good"
            }
          ],
          "summary": {
            "total_samples": 4500,
            "avg_value": 12850.2,
            "min_value": 1200.5,
            "max_value": 21850.0,
            "data_points_missing": 2
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Data retrieved
   * **404 Not Found**: Crane or logger not found
   * **401 Unauthorized**: Invalid or missing API key

Rule Engine Integration
~~~~~~~~~~~~~~~~~~~~~~~

Get Safety Rules
^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/rules

   Get auto-generated safety rules.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Query Parameters:**

   * **status** (optional): Filter by status - ``active``, ``disabled``
   * **source** (optional): Filter by source - ``load_limiter``, ``acs``, ``zoning``, ``health``
   * **severity** (optional): Filter by severity - ``critical``, ``warning``, ``info``

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "rules": [
            {
              "rule_id": "R-LL-001",
              "name": "Load Limiter Warning",
              "description": "IF Load > 90% THEN Trigger Alarm",
              "source": "load_limiter",
              "trigger_condition": "load_percentage > 90",
              "actions": [
                {
                  "type": "trigger_alarm",
                  "alarm_id": "ALM-LL-001",
                  "severity": "warning"
                }
              ],
              "status": "active",
              "severity": "warning",
              "generated_from": "Load Limiter Configuration",
              "generated_at": "2024-01-15T10:30:00Z",
              "statistics": {
                "trigger_count": 15,
                "last_triggered": "2024-01-15T08:30:00Z"
              }
            }
          ],
          "summary": {
            "total_rules": 6,
            "active_rules": 6,
            "rules_by_source": {
              "load_limiter": 2,
              "acs": 2,
              "zoning": 1,
              "health": 1
            },
            "trigger_count_24h": 19
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Rules retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Get Rule Execution History
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/rules/history

   Get rule execution history.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Query Parameters:**

   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **rule_id** (optional): Filter by rule ID
   * **limit** (optional): Maximum records (default: ``50``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "executions": [
            {
              "execution_id": "EXEC-001",
              "rule_id": "R-LL-001",
              "rule_name": "Load Limiter Warning",
              "timestamp": "2024-01-15T08:30:00Z",
              "trigger_condition": "load_percentage = 107.2",
              "actions_executed": [
                {
                  "type": "trigger_alarm",
                  "alarm_id": "ALM-LL-001",
                  "status": "executed",
                  "timestamp": "2024-01-15T08:30:00Z"
                }
              ],
              "context": {
                "load_value": 26800,
                "load_percentage": 107.2,
                "operator": "auto",
                "location": "x=125.5, y=230.8"
              },
              "acknowledged": true,
              "acknowledged_by": "john_doe",
              "acknowledged_at": "2024-01-15T08:32:00Z"
            }
          ],
          "summary": {
            "total_executions": 19,
            "executions_24h": 4,
            "critical_executions": 1,
            "warning_executions": 3,
            "acknowledged": 3,
            "pending_acknowledgement": 1
          }
        }
      }

   **Status Codes:**

   * **200 OK**: History retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Test and Validation
~~~~~~~~~~~~~~~~~~~

Test Connection
^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/test-connection

   Test connection to crane.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "test_time": "2024-01-15T13:00:00Z",
          "connection_status": "connected",
          "response_time": "0.125s",
          "gateway_status": "online",
          "systems_checked": [
            {
              "system": "load_limiter",
              "status": "responsive",
              "response_time": "0.080s"
            }
          ],
          "overall_status": "healthy",
          "recommendations": [
            "All systems operational",
            "Connection quality: excellent"
          ]
        }
      }

   **Status Codes:**

   * **200 OK**: Test completed
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key
   * **503 Service Unavailable**: Crane offline

Test Safety Systems
^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/test-safety-systems

   Test safety systems.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/test-safety-systems HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "tests": ["load_limiter", "acs", "emergency_stop"],
        "simulate": true,
        "operator": "admin@factory.com"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "test_time": "2024-01-15T13:05:00Z",
          "mode": "simulation",
          "results": [
            {
              "system": "load_limiter",
              "test": "overload_simulation",
              "status": "passed",
              "simulated_value": "105%",
              "expected_action": "motion_stop",
              "actual_action": "motion_stop",
              "response_time": "0.250s"
            }
          ],
          "summary": {
            "total_tests": 3,
            "passed": 3,
            "failed": 0,
            "overall_status": "pass",
            "test_duration": "2.5s"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: Test completed
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Crane in operation (if simulate=false)
   * **401 Unauthorized**: Invalid or missing API key

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

Save Configuration
^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/configuration/save

   Save current configuration.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/configuration/save HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "comment": "Updated load limiter thresholds",
        "backup_existing": true
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "configuration_id": "CONFIG-20250115-001",
          "backup_id": "BACKUP-20250115-001",
          "saved_at": "2024-01-15T13:10:00Z",
          "configuration_summary": {
            "load_limiter": true,
            "acs": true,
            "zoning": true,
            "health_monitoring": true,
            "dataloggers": 6
          },
          "download_url": "https://api.univagateway.com/v1/downloads/CRANE-01-config-20250115.json",
          "expires_at": "2024-01-22T13:10:00Z"
        },
        "message": "Configuration saved successfully"
      }

   **Status Codes:**

   * **200 OK**: Configuration saved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

Reset to Factory Defaults
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:post:: /cranes/{id}/configuration/reset

   Reset to factory defaults.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Request:**

   .. sourcecode:: http

      POST /cranes/CRANE-01/configuration/reset HTTP/1.1
      Content-Type: application/json
      Authorization: Bearer <token>
      
      {
        "reason": "System malfunction",
        "operator": "admin@factory.com",
        "password": "********"
      }

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "reset_time": "2024-01-15T13:15:00Z",
          "reset_by": "admin@factory.com",
          "reason": "System malfunction",
          "backup_created": true,
          "backup_id": "BACKUP-20250115-002",
          "systems_reset": [
            "load_limiter",
            "acs",
            "zoning",
            "health_monitoring",
            "dataloggers"
          ],
          "next_steps": [
            "Reconfigure safety parameters",
            "Calibrate load limiter",
            "Test safety systems"
          ]
        },
        "message": "Configuration reset to factory defaults"
      }

   **Status Codes:**

   * **200 OK**: Reset completed
   * **404 Not Found**: Crane not found
   * **400 Bad Request**: Invalid authorization
   * **401 Unauthorized**: Invalid or missing API key

Get Configuration History
^^^^^^^^^^^^^^^^^^^^^^^^^

.. http:get:: /cranes/{id}/configuration/history

   Get configuration history.

   **Path Parameters:**

   * **id** (string): Crane ID

   **Query Parameters:**

   * **start_date** (optional): Start date (ISO format)
   * **end_date** (optional): End date (ISO format)
   * **limit** (optional): Maximum records (default: ``20``)

   **Response:**

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "status": "success",
        "data": {
          "crane_id": "CRANE-01",
          "configurations": [
            {
              "configuration_id": "CONFIG-20250115-001",
              "timestamp": "2024-01-15T13:10:00Z",
              "type": "user_save",
              "user": "admin@factory.com",
              "comment": "Updated load limiter thresholds",
              "changes": [
                "load_limiter.warning_threshold: 90 -> 85",
                "load_limiter.cutoff_threshold: 110 -> 105"
              ],
              "download_url": "https://api.univagateway.com/v1/downloads/CRANE-01-config-20250115.json"
            }
          ],
          "summary": {
            "total_configurations": 45,
            "user_saves": 12,
            "auto_backups": 33,
            "last_configuration": "2024-01-15T13:10:00Z"
          }
        }
      }

   **Status Codes:**

   * **200 OK**: History retrieved
   * **404 Not Found**: Crane not found
   * **401 Unauthorized**: Invalid or missing API key

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
     "timestamp": "2024-01-15T13:00:00Z"
   }

Common Error Codes
^^^^^^^^^^^^^^^^^^

.. list-table:: CraneIQ Error Codes
   :widths: 30 70
   :header-rows: 1

   * - Error Code
     - Description
   * - **CRANEIQ_001**
     - Invalid crane configuration
   * - **CRANEIQ_002**
     - Crane not found
   * - **CRANEIQ_003**
     - Crane offline
   * - **CRANEIQ_004**
     - Invalid safety parameter
   * - **CRANEIQ_005**
     - Zone overlap detected
   * - **CRANEIQ_006**
     - Configuration validation failed
   * - **CRANEIQ_007**
     - Calibration failed
   * - **CRANEIQ_008**
     - Test failed
   * - **CRANEIQ_009**
     - Crane in operation
   * - **CRANEIQ_010**
     - Authorization required
   * - **CRANEIQ_011**
     - Datalogger configuration error

Webhooks
--------

Crane Alert Webhook
^^^^^^^^^^^^^^^^^^^

When a crane safety alert is triggered:

.. code-block:: json

   {
     "event": "crane.safety_alert",
     "timestamp": "2024-01-15T08:30:00Z",
     "data": {
       "crane_id": "CRANE-01",
       "crane_name": "Double-Girder Crane 01",
       "alert_type": "overload",
       "severity": "critical",
       "value": 26800,
       "unit": "kg",
       "percentage": 107.2,
       "location": {
         "x": 125.5,
         "y": 230.8,
         "z": 0.0
       },
       "action_taken": "motion_stopped",
       "operator": "auto",
       "rule_id": "R-LL-001",
       "description": "Load exceeded cutoff threshold"
     }
   }

Crane Status Change Webhook
^^^^^^^^^^^^^^^^^^^^^^^^^^^

When crane status changes:

.. code-block:: json

   {
     "event": "crane.status_changed",
     "timestamp": "2024-01-15T10:30:00Z",
     "data": {
       "crane_id": "CRANE-01",
       "previous_status": "online",
       "new_status": "maintenance",
       "reason": "Scheduled maintenance",
       "operator": "tech-001",
       "estimated_duration": "2 hours"
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
   base_url = "https://api.univagateway.com/v1/craneiq"
   
   headers = {
       "Authorization": f"Bearer {api_key}",
       "Content-Type": "application/json"
   }
   
   # Get crane details
   response = requests.get(
       f"{base_url}/cranes/CRANE-01",
       headers=headers
   )
   
   if response.status_code == 200:
       crane_data = response.json()["data"]
       print(f"Crane: {crane_data['name']}")
       print(f"Status: {crane_data['status']}")
       print(f"Health Score: {crane_data['health_score']}")
   
   # Update load limiter configuration
   load_limiter_config = {
       "enabled": True,
       "settings": {
           "rated_load": 28000,
           "warning_threshold": 85,
           "cutoff_threshold": 105
       }
   }
   
   update_response = requests.put(
       f"{base_url}/cranes/CRANE-01/load-limiter",
       headers=headers,
       json=load_limiter_config
   )
   
   if update_response.status_code == 200:
       print("Load limiter configuration updated")

JavaScript Example
^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   const apiKey = "your_api_key";
   const baseUrl = "https://api.univagateway.com/v1/craneiq";
   
   const headers = {
       "Authorization": `Bearer ${apiKey}`,
       "Content-Type": "application/json"
   };
   
   // Create a new datalogger
   const loggerConfig = {
       device_id: "hoist_motor",
       tag_id: "motor_temp",
       priority: "high",
       condition: "periodic",
       storage: {
           size: 200,
           unit: "MB"
       },
       sampling: {
           rate: 10000,
           data_format: "compressed"
       }
   };
   
   fetch(`${baseUrl}/cranes/CRANE-01/dataloggers`, {
       method: "POST",
       headers: headers,
       body: JSON.stringify(loggerConfig)
   })
   .then(response => response.json())
   .then(data => console.log(`Datalogger created: ${data.data.logger_id}`))
   .catch(error => console.error("Error:", error));
   
   // Get real-time health status
   fetch(`${baseUrl}/cranes/CRANE-01/health/status`, {
       headers: headers
   })
   .then(response => response.json())
   .then(data => {
       const health = data.data;
       console.log(`Overall Health: ${health.overall_health}`);
       health.systems.forEach(system => {
           console.log(`${system.system}: ${system.value}${system.unit} (${system.status})`);
       });
   });

Support
-------

For API support, contact:

* **Email:** craneiq-support@univagateway.com
* **Documentation:** https://docs.univagateway.com/api/craneiq
* **Status Page:** https://status.univagateway.com
* **Emergency Support:** +1-800-CRANE-IQ (Available 24/7)

Crane Types and Specifications
------------------------------

Crane Types
^^^^^^^^^^^

.. list-table:: Supported Crane Types
   :widths: 25 50 25
   :header-rows: 1

   * - Type
     - Description
     - Typical Capacity
   * - **double_girder**
     - Double girder overhead crane
     - 5-500 tons
   * - **single_girder**
     - Single girder overhead crane
     - 1-20 tons
   * - **gantry**
     - Gantry crane
     - 5-200 tons
   * - **jib**
     - Jib crane
     - 0.5-10 tons
   * - **tower**
     - Tower crane
     - 10-100 tons
   * - **rtg**
     - Rubber-tired gantry
     - 30-60 tons
   * - **portal**
     - Portal crane
     - 10-100 tons

Safety System Specifications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Safety System Parameters
   :widths: 30 40 30
   :header-rows: 1

   * - System
     - Parameters
     - Accuracy
   * - **Load Limiter**
     - Capacity, Warning %, Cutoff %
     - ±0.5% FS
   * - **Anti-Collision**
     - Warning Distance, Stop Distance
     - ±2cm
   * - **Zoning**
     - Coordinates, Height Limits
     - ±5cm
   * - **Health Monitoring**
     - Temperature, Vibration, Current
     - ±1%

Data Logger Specifications
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Datalogger Specifications
   :widths: 30 40 30
   :header-rows: 1

   * - Parameter
     - Range
     - Resolution
   * - **Sampling Rate**
     - 100ms - 10min
     - 100ms increments
   * - **Storage Capacity**
     - 10MB - 10GB
     - Configurable
   * - **Data Retention**
     - 24h - 365 days
     - Based on sampling
   * - **Compression Ratio**
     - 2:1 - 10:1
     - Configurable

Configuration Best Practices
----------------------------

1. **Load Limiter Configuration:**
   - Set warning threshold at 85-90% of SWL
   - Set cutoff threshold at 105-110% of SWL
   - Enable shock load detection for safety
   - Regular calibration every 6 months

2. **ACS Configuration:**
   - Set warning distance based on crane speed
   - Configure stop distance with safety margin
   - Pair all cranes operating in same area
   - Regular sensor calibration

3. **Zoning Strategy:**
   - Define no-go zones for hazardous areas
   - Set slow zones for high-traffic areas
   - Implement override protocols for maintenance
   - Regular zone boundary validation

4. **Health Monitoring:**
   - Set realistic temperature limits
   - Configure current thresholds
   - Enable vibration monitoring
   - Regular system health checks

Safety Compliance Standards
---------------------------

1. **International Standards:**
   - ISO 12482-1:2014 (Crane monitoring)
   - ISO 4309:2017 (Cranes - Wire ropes)
   - ISO 12480-1:2018 (Safe use of cranes)
   - FEM 9.511 (Testing of cranes)

2. **Regional Standards:**
   - ASME B30.2 (Overhead cranes)
   - EN 13001 (Crane safety)
   - OSHA 1910.179 (US regulations)
   - LOLER (UK regulations)

3. **Industry Certifications:**
   - CE Marking
   - UL Certification
   - SIL 2/SIL 3 (Safety Integrity Level)
   - IEC 61508 (Functional safety)

Troubleshooting Guide
---------------------

Common Issues
^^^^^^^^^^^^^

1. **Load Limiter Malfunctions:**
   - Verify sensor calibration
   - Check wiring connections
   - Validate configuration parameters
   - Test with known weights

2. **ACS False Alarms:**
   - Clean sensor lenses
   - Adjust distance thresholds
   - Update crane geometry
   - Test with static objects

3. **Zone Violation Issues:**
   - Verify coordinate accuracy
   - Check height settings
   - Validate override permissions
   - Test zone boundaries

4. **Communication Problems:**
   - Check network connectivity
   - Verify gateway status
   - Test data transmission
   - Review firewall settings

Debug Procedures
^^^^^^^^^^^^^^^^

1. **System Diagnostics:**
   - Run connection test
   - Execute system self-check
   - Review error logs
   - Check sensor status

2. **Safety System Validation:**
   - Test each system individually
   - Verify emergency stop functionality
   - Validate alarm systems
   - Check manual override

3. **Data Integrity Checks:**
   - Verify data logging
   - Check timestamp synchronization
   - Validate data accuracy
   - Review data retention

Maintenance Schedule
--------------------

.. list-table:: Recommended Maintenance Schedule
   :widths: 30 30 40
   :header-rows: 1

   * - Component
     - Frequency
     - Activities
   * - **Load Sensors**
     - Monthly
     - Calibration, Inspection, Cleaning
   * - **ACS Sensors**
     - Quarterly
     - Alignment, Cleaning, Testing
   * - **Gateway**
     - Biannually
     - Firmware Update, Diagnostics
   * - **System Software**
     - Annually
     - Updates, Security Patches
   * - **Safety Systems**
     - Monthly
     - Full Functional Test
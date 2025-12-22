Univa-Gateway API Documentation
===============================

Overview
--------

This is the complete API interface documentation for the **Univa-Gateway** web application. These REST endpoints provide programmatic access to manage IoT gateways, devices, and telemetry data.

API Structure
-------------

All endpoints follow RESTful conventions and are organized into logical categories:

**Authentication API**
- User login, token management, and session handling
- Separate authentication system with JWT tokens

**Device Management API**
- Gateway and device provisioning
- Configuration management
- Real-time monitoring

Base URL
--------

All API calls should be made to::

    https://api.univa-gateway.com/api/v1/

Usage
-----

To use the Univa-Gateway API:

1. **Authenticate** using the Authentication API
2. **Include token** in subsequent requests
3. **Call endpoints** as documented in the full API reference

For complete endpoint details, parameters, and examples, refer to the interactive API documentation at::

    https://docs.univa-gateway.com/api-reference



*This document provides an overview of the Univa-Gateway API interface.*
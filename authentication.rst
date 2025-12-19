Authentication API
==================

Endpoints for user authentication and account management.

Register User
-------------

.. http:post:: /auth/register

   Create a new user account.

   **Request Example**:

   .. sourcecode:: http

      POST /auth/register HTTP/1.1
      Content-Type: application/json
      
      {
        "email": "admin@example.com",
        "password": "secure123",
        "name": "System Admin"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "token": "eyJhbGciOiJIUzI1NiIs...",
        "user": {
          "id": "1",
          "email": "admin@example.com",
          "name": "System Admin",
          "role": "admin"
        }
      }

Login User
----------

.. http:post:: /auth/login

   Authenticate and get access token.

   **Request**:

   .. sourcecode:: http

      POST /auth/login HTTP/1.1
      Content-Type: application/json
      
      {
        "email": "admin@example.com",
        "password": "secure123"
      }

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "token": "eyJhbGciOiJIUzI1NiIs...",
        "user": {
          "id": "1",
          "email": "admin@example.com",
          "name": "System Admin",
          "role": "admin"
        }
      }

Get Current User
----------------

.. http:get:: /auth/me

   Get authenticated user profile.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "user": {
          "id": "1",
          "email": "admin@example.com",
          "name": "System Admin",
          "role": "admin"
        }
      }

Logout
------

.. http:post:: /auth/logout

   Invalidate current token.

   **Headers**:

   .. code-block:: http

      Authorization: Bearer <token>

   **Response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Logged out successfully"
      }
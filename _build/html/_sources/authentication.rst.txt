Authentication System Documentation
===================================

This document describes the authentication routes and API endpoints for the application.

Page Routes (Frontend)
----------------------

These routes render HTML pages to the user.

.. http:get:: /login

   **Description**: Renders the login page.
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!-- Login page HTML -->

.. http:get:: /register

   **Description**: Renders the registration page.
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!-- Registration page HTML -->

.. http:get:: /profile

   **Description**: Renders the user profile page. Requires authentication.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!-- Profile page HTML -->
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

.. http:get:: /change-password

   **Description**: Renders the change password page. Requires authentication.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: text/html
      
      <!-- Change password page HTML -->
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login

.. http:get:: /logout

   **Description**: Logs out user and redirects to login page.
   
   **Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 302 Found
      Location: /login
      Set-Cookie: session_token=; expires=Thu, 01 Jan 1970 00:00:00 GMT

API Endpoints (Backend)
-----------------------

These endpoints handle authentication operations and return JSON responses.

.. http:post:: /auth/register

   **Description**: Creates a new user account.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /auth/register HTTP/1.1
      Content-Type: application/json
      
      {
        "email": "user@example.com",
        "password": "secure123",
        "name": "John Doe"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 201 Created
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Registration successful",
        "user": {
          "id": "123",
          "email": "user@example.com",
          "name": "John Doe"
        }
      }
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 400 Bad Request
      Content-Type: application/json
      
      {
        "success": false,
        "error": "Email already exists"
      }

.. http:post:: /auth/login

   **Description**: Authenticates user and creates session.
   
   **Request**:
   
   .. sourcecode:: http
   
      POST /auth/login HTTP/1.1
      Content-Type: application/json
      
      {
        "email": "user@example.com",
        "password": "secure123"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      Set-Cookie: session_token=abc123; HttpOnly; Secure
      
      {
        "success": true,
        "message": "Login successful",
        "redirect": "/profile"
      }
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 401 Unauthorized
      Content-Type: application/json
      
      {
        "success": false,
        "error": "Invalid credentials"
      }

.. http:get:: /auth/me

   **Description**: Gets current authenticated user data.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "user": {
          "id": "123",
          "email": "user@example.com",
          "name": "John Doe",
          "role": "user"
        }
      }
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 401 Unauthorized
      Content-Type: application/json
      
      {
        "success": false,
        "error": "Not authenticated"
      }

.. http:put:: /auth/change-password

   **Description**: Changes user's password.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Request**:
   
   .. sourcecode:: http
   
      PUT /auth/change-password HTTP/1.1
      Content-Type: application/json
      
      {
        "currentPassword": "old123",
        "newPassword": "new456"
      }
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      
      {
        "success": true,
        "message": "Password updated successfully"
      }
   
   **Error Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 400 Bad Request
      Content-Type: application/json
      
      {
        "success": false,
        "error": "Current password is incorrect"
      }

.. http:post:: /auth/logout

   **Description**: Logs out user from current session.
   
   **Headers**:
   
   .. sourcecode:: http
   
      Cookie: session_token=<token>
   
   **Success Response**:
   
   .. sourcecode:: http
   
      HTTP/1.1 200 OK
      Content-Type: application/json
      Set-Cookie: session_token=; expires=Thu, 01 Jan 1970 00:00:00 GMT
      
      {
        "success": true,
        "message": "Logged out successfully"
      }

Route Summary
-------------

.. list-table:: Page Routes
   :header-rows: 1
   :widths: 20 20 40 20
   
   * - Method
     - Route
     - Description
     - Auth Required
   * - GET
     - ``/login``
     - Login page
     - No
   * - GET
     - ``/register``
     - Registration page
     - No
   * - GET
     - ``/profile``
     - User profile page
     - Yes
   * - GET
     - ``/change-password``
     - Change password page
     - Yes
   * - GET
     - ``/logout``
     - Logout and redirect
     - No

.. list-table:: API Endpoints  
   :header-rows: 1
   :widths: 20 20 40 20
   
   * - Method
     - Route
     - Description
     - Auth Required
   * - POST
     - ``/auth/register``
     - Create user account
     - No
   * - POST
     - ``/auth/login``
     - Authenticate user
     - No
   * - GET
     - ``/auth/me``
     - Get user data
     - Yes
   * - PUT
     - ``/auth/change-password``
     - Update password
     - Yes
   * - POST
     - ``/auth/logout``
     - End session
     - Yes

Flow Examples
-------------

User Registration Flow
~~~~~~~~~~~~~~~~~~~~~~

1. User visits ``GET /register`` (registration page)
2. Fills form and submits to ``POST /auth/register``
3. On success, redirected to ``/profile``
4. Profile page loads user data via ``GET /auth/me``

User Login Flow
~~~~~~~~~~~~~~~

1. User visits ``GET /login`` (login page)
2. Enters credentials and submits to ``POST /auth/login``
3. On success, receives session cookie and redirected to ``/profile``
4. Profile page loads user data via ``GET /auth/me``

Password Change Flow
~~~~~~~~~~~~~~~~~~~~

1. User visits ``GET /change-password`` (password page)
2. Enters old and new passwords, submits to ``PUT /auth/change-password``
3. On success, shows confirmation message
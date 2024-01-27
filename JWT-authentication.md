# JWT Authentication with NGINX Plus API Gateway

## Introduction
This project demonstrates how to authenticate API clients using JSON Web Tokens (JWT) with NGINX Plus as an API gateway. It includes sample client requests, NGINX configurations, and JWT handling mechanisms.

## Technical Overview
- **NGINX Plus**: Configured as an API gateway to validate JWTs.
- **JWT**: Used for client authentication and authorization.
- **Backend API**: Protected resources that clients access via the NGINX gateway.

## Prerequisites
- NGINX Plus
- Identity Provider (IdP) for JWT generation (e.g., Auth0, Okta)
- Backend API server

## Installation and Configuration

### NGINX Plus Setup
Set up NGINX Plus as an API gateway. The configuration enables JWT validation:
```nginx
http {
    server {
        listen 80;

        # JWT validation location
        location /api/ {
            auth_jwt "API access" token=$cookie_auth_token;
            auth_jwt_key_file /etc/nginx/jwt_keys.json;

            proxy_pass http://backend_server;
        }
    }
}


### Obtaining JWT from IdP

Use an Identity Provider to obtain a JWT. Example request to IdP:
```
curl -X POST https://idp.example.com/auth \
     -H "Content-Type: application/json" \
     -d '{"username":"user", "password":"pass"}'
```

 IdP Responds with JWT

Assuming the credentials are correct, the IdP responds with a JWT. For example:


{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}


Usage
### Making an Authenticated API Request

With the JWT, make a request to the API via the NGINX gateway:

bash

curl -X GET http://nginx-gateway/api/resource \
     -H "Authorization: Bearer <JWT>"



Step 4: NGINX Validates the JWT

Upon receiving the request, NGINX validates the JWT. If the token is valid, NGINX forwards the request to the specified backend service. If not, it returns an authentication error.

This flow provides a basic understanding of how JWTs are used in conjunction with NGINX for secure API access. The specifics can vary based on the actual implementation and configuration of the IdP and NGINX server.


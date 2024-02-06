In this scenario: 

![Topology](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Topology.PNG)

For this, step by step, we will configure APM as OAuth Server
You can refer to this link for more: 
This site was built using [Manual Chapter : Using APM as an OAuth 2.0 Authorization Server](https://techdocs.f5.com/en-us/bigip-17-0-0/big-ip-access-policy-manager-oauth-configuration/using-apm-as-an-oauth-2-server.html#concept-9381).

# From APM: We will define the OAuth profiles, scopes, access policies, and client applications in APM, including Postman.

### Registering a client application for OAuth services
### Registering a resource server for OAuth services
### Configuring OAuth scopes of access for client apps 
### Configuring JWT claims
### Configuring JSON web keys (JWKs)
### Creating an OAuth profile
### Configuring support for JWTs in an OAuth profile
### Creating an access profile for F5 as an OAuth authorization server
### Configuring an access policy for F5 as an OAuth authorization server
### Creating a virtual server for OAuth authorization server traffic


# Setup Postman Client 
- Configure Postman with the authorization details provided by APM, such as the client ID and client secret, and set the callback URL to the one specified in APM.

# Obtain Access Token:
- Use Postman to send a request to APM's token endpoint.
- Authenticate using the provided credentials.
- APM validates the credentials and returns an access token.

# Configure NGINX Plus:
- Enable JWT validation and OAuth 2.0 introspection on NGINX Plus.
- Configure NGINX to communicate with APM for token introspection.
- Setup the necessary location blocks in your NGINX configuration to protect the backend application.

# Access Protected Resource:
- Send a request from Postman to the NGINX Plus Resource Server with the access token.
- NGINX Plus validates the token with APM.
- Upon successful validation, NGINX Plus forwards the request to the backend application.

# Backend Application:
- Receives the request from NGINX Plus.
- Processes the request and returns the response.

# Response to Postman:
- NGINX Plus sends the backend application's response back to Postman.

        

![Image Alt Text](URL)

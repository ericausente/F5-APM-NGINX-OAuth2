  For this, step by step, we will configure APM as OAuth Server
  From APM: We will define the OAuth profiles, scopes, access policies, and client applications in APM, including Postman.

In this scenario: 


![Topology](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Topology.PNG)

    
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

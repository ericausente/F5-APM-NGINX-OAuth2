# Overview

This project demonstrates the implementation of OAuth 2.0 using F5 BIG-IP Access Policy Manager (APM) as the Authorization Server and NGINX as the Resource Server. It outlines the setup of OAuth profiles, scopes, access policies, and client applications, including Postman for testing purposes.

# Topology

A brief description of the network topology used in this project, highlighting the role of each component within the OAuth  flow.
![Topology](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Topology_mew.PNG)

For this, step by step, we will configure APM as OAuth Server

You can refer to this link for more: 

This site was built using [Manual Chapter : Using APM as an OAuth 2.0 Authorization Server](https://techdocs.f5.com/en-us/bigip-17-0-0/big-ip-access-policy-manager-oauth-configuration/using-apm-as-an-oauth-2-server.html#concept-9381).


### Configuring OAuth scopes of access for client apps 
BIG-IP UI >> Access >> Federation >> OAuth Authorization Server >> Scope >> Create
![Scope1](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Scope1.PNG)
![Scope2](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Scope2.PNG)

### Registering a client application for OAuth services
BIG-IP UI >> Access >> Federation >> OAuth Authorization Server >> Client Application >> Create
![Client-App](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Oauth%20Client.PNG)

### Registering a resource server for OAuth services (NGINX)
![RS](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/RS.PNG)

### Configuring JWT claims
Access  ››  Federation : OAuth Authorization Server : Claim 
![Claims](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Claims.PNG)

### Configuring JSON web keys (JWKs)
Access  ››  Federation : JSON Web Token : Key Configuration  ››  jwk
![JWKs](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/JWKs.PNG)

Take note of the ID here and and the type as well as the shared secret. 
We will be using that one to create the key.jwk in nginx later on, it has to be the same for purposes of offline validation. 

### Creating an OAuth profile and Configuring support for JWTs in an OAuth profile
BIG-IP UI >> Access >> Federation >> OAuth Authorization Server >> OAuth Profile >> Create
![OAUTH Profile](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Oauth%20Profile.PNG)

### Creating an access profile for F5 as an OAuth authorization server
BIG-IP UI >> Access >> Profiles / Policies >> Access Profiles (Per-Session Policies) >> Create
![Access Profile](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/Access%20Profile.PNG)

### Configuring an access policy for F5 as an OAuth authorization server
![VPE1](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/AccessPolicyVPE1.PNG)
![VPE2](https://github.com/ericausente/JWT-OAUTH-OIDC/blob/main/VPE2.PNG)

### Creating a virtual server for OAuth authorization server traffic

```
ltm virtual oidcp_vs {
    creation-time 2024-02-04:15:42:13
    destination 172.16.4.156:https
    ip-protocol tcp
    last-modified-time 2024-02-04:15:55:14
    mask 255.255.255.255
    profiles {
        clientssl {
            context clientside
        }
        http { }
        oidcp { }
        rba { }
        serverssl {
            context serverside
        }
        tcp { }
        websso { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    translate-address enabled
    translate-port enabled
    vs-index 23
}
```

# Setup Postman Client 
- Configure Postman with the authorization details provided by APM, such as the client ID and client secret, and set the callback URL to the one specified in APM.

# Obtain Access Token:
- Use Postman to send a request to APM's token endpoint.
- Authenticate using the provided credentials.
- APM validates the credentials and returns an access token.

# Configure NGINX Plus:
- Enable JWT validation and OAuth 2.0 introspection on NGINX Plus.
- Configure NGINX to communicate with APM for token introspection
We can Specify the path to the JSON Web Key file that will be used to verify JWT signature or decrypt JWT content, depending on what you are using. 
This can be done with the auth_jwt_key_file and/or auth_jwt_key_request directives. 
Specifying both directives at the same time will allow you to specify more than one source for keys. 

In my case, for JWS signature verification, I only used auth_jwt_key_file directives. 
In this scenario, the keys will be taken from two files: the key.jwk file. 

```
$ pwd
/etc/nginx
```

```
$ cat key.jwk
{"keys":
    [{
        "k":"ZmFudGFzdGljand0",    #  echo -n fantasticjwt | base64 | tr '+/' '-_' | tr -d '='
        "kty":"oct",
        "kid":"0001"
    }]
}
```

nginx.conf
```
...

server {
listen 8086;
location /products/ {
return 200 "accessing products";
}
}


server {
    listen 8006;
    location /products/ {
        proxy_pass           http://127.0.0.1:8086;
        auth_jwt             "API";
        auth_jwt_key_file    /etc/nginx/key.jwk;     ################################ -> !!!
        #auth_jwt_key_request /public_jws_keys;
    }

    location /public_jws_keys {
        internal;
        proxy_pass "https://172.16.4.156/f5-oauth2/v1/jwkeys";
    }
}

```

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

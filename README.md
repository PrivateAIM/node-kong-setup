# node-kong-setup

## Overview

The `node-kong-setup` is a repository to facilitate the deployment and configuration of Kong on the node side. This setup employs Docker and Kong's native capabilities to configure Kong with enhanced features including integration with Keycloak, and JWT support for secure Admin API access. Since Kong's Admin API allows full control of Kong, it is important to secure this API against unwanted access. The solution involves using Kong to protect its own Admin API, leveraging its routing capabilities to proxy access to the Admin API. Typically, the Kong `admin_listen` is configured at `127.0.0.1:8001`, accessible only locally. The goal is to expose the Admin API via `kong_address/admin-api` in a controlled manner. This is achieved by setting up a Service and Route. Additionally, the JWT and ACL plugins from Kong are used to add a layer of security to the Admin API.

## Prerequisites

- Docker
- Docker-Compose

## Configuration

The `docker-compose.yml` file includes:
- **Kong Migrations**: Manages database migrations.
- **Kong Bootstrap Config**: Imports configurations from kong.yaml into the database.
- **Kong Service**: The main API Gateway service processing API requests.
- **PostgreSQL Database**: Stores Kong's configuration data.

The `kong.yaml` file defines a service that can be accessed through `/admin-api` route, as determined by a set of defined HTTP methods. Authentication and access control are enforced through the use of plugins. The ACL (Access Control List) plugin restricts service access to members of a specified group (`kong-admin-group`), while the JWT (JSON Web Token) plugin ensures that requests are authenticated and verified using provided public key and algorithm.

Assume `pod-orchestrator-client` wants to access Kong's Admin API, before making a request to the Kong Gateway, the `pod-orchestrator-client` first authenticates itself with Keycloak. The `pod-orchestrator-client` provides its credentials to Keycloak, Keycloak validates the credentials. If they are correct, Keycloak issues a JWT to the `pod-orchestrator-client`. This token contains claims that identify the client:

```json
{
  "exp": 1706615232,
  "iat": 1706614932,
  "jti": "e79948a8-94d8-4931-afc6-7797671d3f92",
  "iss": "https://auth.flame.uk-koeln.de/auth/realms/flame",
  "aud": "account",
  "sub": "1873533a-bceb-4d72-87a4-d1475146e62e",
  "typ": "Bearer",
  "azp": "pod-orchestrator-client",
  "acr": "1",
  "realm_access": {
    "roles": [
      "offline_access",
      "default-roles-flame",
      "uma_authorization"
    ]
  },
  "resource_access": {
    "account": {
      "roles": [
        "manage-account",
        "manage-account-links",
        "view-profile"
      ]
    }
  },
  "scope": "profile email",
  "email_verified": false,
  "clientHost": "134.95.195.27",
  "clientId": "pod-orchestrator-client",
  "preferred_username": "service-account-pod-orchestrator-client",
  "clientAddress": "134.95.195.27"
}
```

The `kong.yaml` file defines the clients allowed to access the Admin API. Each client is defined under `consumers`, with the `key` value matching the registered client name in Keycloak. The `kong.yaml` file also includes the public key (in PEM format) and algorithm used to verify the JWT claims (`exp` and `clientId`). Public key and algorithm should be obtained from Keycloak. Additionally, the `kong-admin-group` is specified for each consumer to restrict access to the Admin API. Only consumers that are part of the `kong-admin-group` are allowed to access the Admin API.


## Usage

1. Clone the repository
2. Start the database, run migrations and import the configurations from `kong.yaml` into the database by executing the following command in the root directory:

```bash
COMPOSE_PROFILES=database docker-compose up -d
```

3. Next run the following command to restart the Kong Gateway:

```bash
docker-compose restart
```

## Links

- [Kong](https://github.com/Kong/kong)
- [docker-kong](https://github.com/Kong/docker-kong)
- [Kong - Securing the Admin API](https://docs.konghq.com/gateway/latest/production/running-kong/secure-admin-api/)
- [Kong - JWT Plugin](https://docs.konghq.com/hub/kong-inc/jwt/)
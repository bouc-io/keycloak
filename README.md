# Keycloak Installation and Configuration

## Prerequisites

Add the required charts repo to Helm:
```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```


## Local

In the cluster this namespace is created by Flux from `clusters/<env>/config/namespaces.yaml`.
These commands are the manual equivalent.

1. Create the namespace with Istio enabled:
```shell
kubectl create namespace keycloak
kubectl label namespace keycloak istio-injection=enabled
```

2. Install Keycloak with the local configuration:
```shell
helm upgrade -install keycloak-release -f base.values.yaml -f lcl.values.yaml bitnami/keycloak -n keycloak
```

3. Load the proper realm to create the default 'users' realm:
3.1 Access the Keycloak UI at: https://sso.docker.internal
3.2 Click on the Admin link.
3.3 Click on the realm dropdown list and click on 'Create Realm'.
3.4 Click 'Browse' and locate the JSON file 'local.users-realm-export.json'
3.5 Click on 'Create'

4. Create a user
4.1 Click on 'Users' on the left-side menu.
4.2 Click on 'Add Users'
4.3 Fill the required information (i.e. firstname, lastname, etc.)
4.4 Click 'Create'
4.5 Click on 'Credentials' and click on 'Set Password'.
4.6 Enter your preferred password and click on 'Save'.

Tip: Since this is the local environment, rather than temporary resetting the password at the first access, disable the temporary password to avoid the process.



## Sandbox

Please note, the current setup for the sandbox Keycloak installation expect an external database instead of creating its own PostgreSQL database on Kubernetes; as such, plan to create an external database (refer to the terraform project) and create the secret with the username and password created during the instantiation of the datbase.

1. Create the namespace with Istio enabled:
```shell
kubectl create namespace keycloak --context $BOUCIO_CLSTR_FULLNAME
kubectl label namespace keycloak istio-injection=enabled --context $BOUCIO_CLSTR_FULLNAME
```

2. Create the secret with the proper DB credentials:
```shell
kubectl create secret generic keycloak-db-secret -n keycloak \
  --from-literal='postgres-password=<YOUR-DATABASE-PASSWORD>' \
  --from-literal='username=bn_keycloak' \
  --from-literal='password=<YOUR-DATABASE-PASSWORD>' \
  --from-literal='database=bitnami_keycloak' \
  --cluster $BOUCIO_CLSTR_FULLNAME
```

2. Create the secret with the proper login credentials:
```shell
kubectl create secret generic keycloak-login-secret -n keycloak \
  --from-literal='adminUser=<USERNAME>' \
  --from-literal='admin-password=<YOUR-DATABASE-PASSWORD>' \
  --cluster $BOUCIO_CLSTR_FULLNAME
```

4. Install Keycloak with the local configuration:
```shell
helm upgrade -install keycloak-release -f base.values.yaml -f snbx.values.yaml bitnami/keycloak -n keycloak --kube-context $BOUCIO_CLSTR_FULLNAME
```

To login for the first time, go to sso.bouc.io, use the user "user" and password setup from step #2.  **Note**, this is a temporary user and should be deleted.

To create a new user, follow these steps:
1. Click on "Users" on the left side.
2. Click on "Add user".
3. Enter the required information.
4. Click on Create.
5. Click on "Role Mapping"
6. Click on "Assign role".
7. Search for "admin" and click "Add role"


## Notes

access_token=$(curl -s http://keycloak.localtest.me:9080/auth/realms/master/protocol/openid-connect/token \
    -H 'Content-Type: application/x-www-form-urlencoded' \
    -d 'username=madmin@example.com' \
    -d 'password=password' \
    -d 'grant_type=password' \
    -d 'client_id=oauth2-proxy' \
    -d 'client_secret=72341b6d-7065-4518-a0e4-50ee15025608' | jq -r '.access_token')

curl http://haproxy.localtest.me:8080/v2/config \
    -H "Authorization: Bearer $access_token"



## References

Refer to:
- https://www.keycloak.org/getting-started/getting-started-kube
- https://www.keycloak.org/documentation
- https://www.keycloak.org/docs/latest/server_installation/index.html
- https://blog.ramjee.uk/istio-api-security-with-keycloak-in-kubernetes/
- https://stackoverflow.com/questions/57265755/how-to-get-client-secret-from-keycloak-using-curl
- https://www.baeldung.com/postman-keycloak-endpoints
- https://auth0.com/docs/authorization/flows/call-your-api-using-the-authorization-code-flow
- https://labs.consol.de/development/2020/05/07/istio-and-keycloak.html
- https://medium.com/@senthilrch/api-access-control-using-istio-ingress-gateway-44be659a087e
- https://medium.com/@senthilrch/api-authentication-using-istio-ingress-gateway-oauth2-proxy-and-keycloak-a980c996c259
- https://medium.com/@senthilrch/api-authentication-using-istio-ingress-gateway-oauth2-proxy-and-keycloak-part-2-of-2-dbb3fb9cd0d0
- https://www.keycloak.org/server/hostname#_accessing_keycloak_in_production_mode_using_http
- https://medium.com/@rishabhsvats/understanding-authorization-code-flow-3946d746407
- https://www.keycloak.org/server/configuration **IMPORTANT**
- https://www.keycloak.org/server/configuration-production **IMPORTANT**
- https://www.keycloak.org/operator/basic-deployment
- https://www.keycloak.org/operator/realm-import
- https://hub.docker.com/u/keycloak

## License

[Elastic License 2.0](./LICENSE) — source-available; not OSI open source.

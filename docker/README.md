
# KEYCLOAK

```bash
docker-compose up -d polar-keycloak
```

```bash
docker-compose down
```

Check the Keycloak logs with docker:
```bash
docker logs -f polar-keycloak
```

Bash console inside the Keycloak container:
```bash
docker exec -it polar-keycloak bash
```

```
cd /opt/keycloak/bin
```

Start an authenticated session in order to use the Admin CLI:
```
./kcadm.sh config credentials --server http://localhost:8080 --realm master --user user --password password    
```

Create a new security realm where all the policies associated with Polar Bookshop will be stored:
```
./kcadm.sh create realms -s realm=PolarBookshop -s enabled=true
```

Register Edge Service as an OAuth2 Client in the PolarBookshop realm:
```
./kcadm.sh create clients -r PolarBookshop \
    -s clientId=edge-service \
    -s enabled=true \
    -s publicClient=false \
    -s secret=polar-keycloak-secret \
    -s 'redirectUris=["http://localhost:9000","http://localhost:9000/login/oauth2/code/*"]'
```

Create roles:
```
./kcadm.sh create roles -r PolarBookshop -s name=employee
./kcadm.sh create roles -r PolarBookshop -s name=customer
```

Create the user Isabelle Dahl (username: isabelle) which will be both an employee and a customer of the bookshop:
```
./kcadm.sh create users -r PolarBookshop \
    -s username=isabelle \
    -s firstName=Isabelle \
    -s lastName=Dahl \
    -s enabled=true
 
./kcadm.sh add-roles -r PolarBookshop \
    --uusername isabelle \
    --rolename employee \
    --rolename customer

./kcadm.sh set-password -r PolarBookshop \
    --username isabelle --new-password password

```

Bjorn Vinterberg (username: bjorn), a customer of the bookshop:
```
./kcadm.sh create users -r PolarBookshop \
    -s username=bjorn \
    -s firstName=Bjorn \
    -s lastName=Vinterberg \
    -s enabled=true
 
./kcadm.sh add-roles -r PolarBookshop \
    --uusername bjorn \
    --rolename customer

./kcadm.sh set-password -r PolarBookshop \
    --username bjorn --new-password password
```


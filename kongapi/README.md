In this scenario we're going to install a REST api in KONG api management.

Restful Api in scala language using playframework
Documented with Swagger Framework 

Just needed route file to get an idea about about routes in our app
We're gonna create services to access to documentation 
  

Scenario:

ApiRest in PlayFramework 
Swagger

Get the proper installation on bare metal:
https://konghq.com/install/

Running in bare metal:
Start kong: 
kong start 

Checking Kong version:
kong version

Testing if is kong running:
curl -i http://localhost:8001/  

By default Kong listens on the following ports:

8000 on which Kong listens for incoming HTTP traffic from your clients, and forwards it to your upstream services.

8443 on which Kong listens for incoming HTTPS traffic. This port has a similar behavior as the :8000 port, except that it expects HTTPS traffic only. This port can be disabled via the configuration file.
8001 on which the Admin API used to configure Kong listens.

8444 on which the Admin API listens for HTTPS traffic

In our first labs we’re just managing an API without any security and accessing no secure port. 

Basic Step for managing:
Note: We’re gonna work in 1.5 ver you can change to a lower version if you want 

Keeping in mind our route file in PlayFramework:

routes:
….
->      /apirest                        apirest.Routes
….
apirest.routes:
  ..
GET /football/:league/matchs  scadip.core.api.controllers.FootballLeagueController.getMatchGame(league: String)

Example of endPoint Before Kong Service: http://localhost:9000/apirest/football/PRML/matchs

1. Creating a Service:
Have a look to kong api for creating a service ref: https://docs.konghq.com/1.5.x/admin-api/#create-service
We add minimal configuration but can configure:  retries, connect_timeout, write_timeout, read_timeout and few more features. 

curl -v -i -X POST \
     --url https://localhost:8001/services/ \
     --data 'name=football-service' \
     --data 'url=http://localhost:9000/apirest/football'

2. Adding a Route for the previous Service:
Have a look to kong api for creating a route ref: https://docs.konghq.com/1.5.x/admin-api/#create-route-associated-to-a-specific-service

curl -i -X POST \
  --url https://localhost:8001/services/football-service/routes \
  --data 'paths[]=/apirest/football'

Now we have a minimal service configuration with Kong. 
Some aspects to consider in the previous implementation:
Our Service object does not have a path attribute(--data ‘path’). All have been  indicated all at the url attribute.
Our Route object has a path attribute(--data ‘paths’). In this case the path handling algorithm[ https://docs.konghq.com/1.5.x/admin-api/#path-handling-algorithms ] is very easy because we just have added a path and not resolver is needed.

So in this point you already already make the service request :

        curl -i -X GET \
        --header "Accept: text/csv" \
        --header "Content-Type: text/plain" \
        --url https://localhost:8000/apirest/football/PRML/matchs

Let’s add security to our service, so you can have an example at Kong web page [ https://docs.konghq.com/1.5.x/getting-started/enabling-plugins/#1-configure-the-key-auth-plugin ] but in our case we’re gonna use oAuth2.0 plugin. 
You can get the plugin documentation [ https://docs.konghq.com/hub/kong-inc/oauth2/ ]

3. adding consumer 
note: username should be as a primary key
         custom_id should be random value 

curl -i -X POST \
  --url http://localhost:8001/consumers/ \
  --data "username=devuser2" \
  --data "custom_id=devuser2@mail.com"

Before add oAuth2 to the service check if the route has been added properly

4. Adding a oAuth2 Plugin to the Service:

curl -X POST \
  --url https://localhost:8001/services/football-service/plugins/ \
  --data "name=oauth2" \
  --data "config.mandatory_scope=true" \
  --data "config.scopes=test" \
  --data "config.scopes=full" \
  --data "config.token_expiration=0" \
  --data "config.enable_password_grant=true" \
  --data "config.global_credentials=true" \
  --data "config.accept_http_if_already_terminated=true"

Note: 
Config.scopes: Depending on some criterias we can create as many scopes as we want and then use it in our code to provide a distinction considering the services we provide depending on the scope applied by each user.
Config.enable_password_grant: One of the 4 ways to get the authorization at the The OAuth 2.0 Authorization Framework(Resource Owner Password Credentials Grant) but can configure one of the other 3 authorization methods to get the authorization. 
Token expiration time 0 (not recommended ) means that token never expires. Recommended value in at oAuth2 spec[ https://tools.ietf.org/html/rfc6749#section-4.2.2 ] 

response:

{
	"created_at": 1568809828,
	"consumer": null,
	"id": "6a13c553-0ca6-41b8-a14a-2a1f341b2d1f4",
	"service": {
		"id": "b24b298c-2234-4bcc-83b6-3ea7fe28a6f2"
	},
	"name": "oauth2",
	"config": {
		"refresh_token_ttl": 1209600,
		"provision_key": "Z1exYOuhbVD9Kd5pR7rQugVT5n9HEorb",
		"mandatory_scope": true,
		"scopes": ["test", "full"],
		"accept_http_if_already_terminated": true,
		"hide_credentials": false,
		"enable_implicit_grant": false,
		"global_credentials": true,
		"anonymous": null,
		"enable_password_grant": true,
		"enable_client_credentials": false,
		"enable_authorization_code": false,
		"token_expiration": 0,
		"auth_header_name": "authorization"
	},
	"route": null,
	"run_on": "first",
	"tags": null,
	"protocols": ["http", "https"],
	"enabled": true
}

We’ll need the value of our previous json field “provision_key”  to request our tokens to access our api.
 
5. Adding a oAuth2 Plugin to the Consumer:

curl -i -X POST \
    --url https://localhost:8001/consumers/devuser2/oauth2/ \
    --data "name=football-api-app" \
    --data "redirect_uris=http://konghq.com/"

response:

{
	"created_at": 1568810015,
	"consumer": {
		"id": "d70186ab-b291-48d4-ba3e-b860294c6f6f"
	},
	"redirect_uris": ["http:\/\/konghq.com\/"],
	"client_id": "ud7LrBAUhSbaRzqvcSJmDQqcIFnxsQuP",
	"name": "football-api-app",
	"client_secret": "AAvqlHhbSG0IMpuQSiCOXSEDqoB48opP",
	"id": "b1172e5c-1760-4be8-b8cc-5c24bdcb31ee"
}

6. request token for different scopes that we created before when was installed oAuth2 plugin:

- scope test
curl -k -i -X POST \
  --url https://localhost:8443/apirest/football/oauth2/token \
  --data "grant_type=password" \
  --data "client_id=consumer_response_clientid" \
  --data "client_secret=consumer_response_clientsecret" \
  --data "provision_key=service_response_provision_key" \
  --data "scope=test" \
  --data "config.token_expiration=0" \
  --data "authenticated_userid=evuser2"

- scope full access
curl -k -i -X POST \
  --url https://localhost:8443/apirest/football/oauth2/token \
  --data "grant_type=password" \
  --data "client_id=consumer_response_clientid" \
  --data "client_secret=consumer_response_clientsecret" \
  --data "provision_key=service_response_provision_key" \
  --data "scope=full" \
  --data "config.token_expiration=0" \
  --data "authenticated_userid=evuser2"

7. request to the api through Kong
Note: token_auth is from response of curl in previous step (6)

curl -k -i -X GET \
      --header "Accept: text/csv" \
      --header "Content-Type: text/plain" \
      --header "Authorization: token_auth" \
      --url https://$kong_ip_address:8443/apirest/football/PRML/matchs

8. So our APIs are documented with the Swagger documentation framework. adding swagger services[for documentation page]

In this case we’re gonna to create a service for swagger to interact with our api through Kong.

curl -v -i -X POST \
     --url http://localhost:8001/services/ \
     --data 'name=swagger-service' \
     --data "url=http://localhost:9000/apirest/assets"

9. Adding routes to swagger service:

curl -i -X POST \
  --url http://localhost:8001/services/swagger-service/routes \
  --data 'paths[]=/apirest/assets'

Accessing to the api documentation:
https://172.17.0.4:8443/apirest/assets/dist/index.html

The process for Kong under Docker in a development environment is quite similar so you can access to my the implementation on Docker with the Alpine image
















Appendix 

The main objects involved in the managing process of our API:

All of these objects can be audited, managed, deleted, updated

Service Object [ https://docs.konghq.com/1.5.x/admin-api/#service-object ]
Route Object [ https://docs.konghq.com/1.5.x/admin-api/#route-object ]
Plugin Object [ https://docs.konghq.com/1.5.x/admin-api/#plugin-object ]


In our local examples We are going to use HTTP traffic and I am going to explain some weak aspects that in my opinion are quite complex to tackle in Kong.

For check configuration and customize it [ https://docs.konghq.com/1.5.x/configuration/ ]  

Dealing with cache in Kong [ https://docs.konghq.com/1.5.x/clustering/#what-is-being-cached ]  

Dealing with OAuth2.0 plugin[ https://docs.konghq.com/hub/kong-inc/oauth2/ ]

Admin Api: [ https://docs.konghq.com/1.5.x/admin-api/ ]

Adding service:
https://docs.konghq.com/1.5.x/getting-started/configuring-a-service/

https://docs.konghq.com/hub/kong-inc/oauth2/

https://docs.konghq.com/1.5.x/getting-started/configuring-a-service/

https://discuss.konghq.com/t/import-swagger-open-api-document-to-create-api-in-kong/510


 

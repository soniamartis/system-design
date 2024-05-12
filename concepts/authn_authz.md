# Authentication(OpenID Connect) and Authorization(Oauth2.0)

Oauth is alll about granting access to apps w/o sharing passwords with them


## Most basic type of authentication

Simple login or forms authentication

![Screenshot 2023-06-03 at 3 20 28 PM](https://github.com/soniamartis/system-design/assets/12456295/95b2bb6d-424d-454a-9a93-2e24140dbded)


Implementing authn in this homegrown fashion is that u are responsible for maintenance and security

OAuth2 and openID Connect are becoming the industry best practices for solving these problems


Identity use-cases:
- Simple login(forms and cookies)
- Single sign on across sites(uses the SAML protocol which is more dense than Oauth, lol)
- Mobile app login(to be logged in to an app on ur mobile. cookies dont fit the bill well here)
- Delegated authorization(how can i let a webite access my data without giving it my password)

Before oauth came in , how did ppl implement delegated authorization(eg os yelp.com):
![Screenshot 2023-06-03 at 3 35 02 PM](https://github.com/soniamartis/system-design/assets/12456295/62fa0820-2f3a-45fb-ae0e-e292bc108abe)


Basically, yelp would ask the customers to provide their gmail email and gmail pwd. then they would log into their gmail acct on the customer's behalf, in order to get the list of contacts from gmail
Once they got the info, they would delete the client supplied pwd.
This is extremely risky, in case someone went rogue and exploited the fact that they are now in possession of the customer's gmail credentials


## Delegated authorization with OAuth 2.0

OAuth flow:
Better flow compared to above:
- yelp redirects the customer to google.com
- customer can enter their credentials on google.com (and not provide it to yelp)
- once logged in, it will open a dialog whether user is willing to grant access to yelp to get access to user's contact list
- On yes, user will be redirected to a callback url to yelp, and then using the granted access, yelp will be able to get the contact list

![image](https://github.com/soniamartis/system-design/assets/12456295/f76a949f-a07c-4080-8a30-f36e3daacb35)




## Terminology used in OAuth

- Resource owner(u and me, in example above: the customer)
- client (the app that wants to get the data, eg yelp)
- authorisation server (system that user can use to grant/deny permission, eg accounts.google.com)
- resource server (system that has the data that client wants to get, eg : google contacts api aka contacts.google.com)
- authorization grant (grant is something that proves that the user has clicked yes)
- redirect uri (yelp callback url)
- access token (client really needs this access_token once it has the authorization grant)

![Screenshot 2023-06-03 at 4 31 49 PM](https://github.com/soniamartis/system-design/assets/12456295/1ffaad02-e17f-4ab6-b1ab-2bf6473a03b2)


Suppose client gets the access_token for reading contacts, but instead of reading, it tries to delete contacts, the resource server(contacts.google.com) will not allow it to delete contacts

More terminology:
scope : 
The authorisation server has a list of scopes that it understands, like contacts.read, contacts.write etc
The client can request multiple scopes

consent:
Using the scopes provided by the client to the authz server, the authz server prepares the consent screen to ask the user whether they would provide consent. 


Question:
Why do we have an authz code and exchange it with an access_token, we could have as well used the authz code itself?

Answer:
More terminology...
- back channel(highly secure channel)
- front channel(less secure channel)

back channel:
When a request happens from one web server to another
The last step of the flow where exchange of authz_code and access_token and access_token with the actual resource server takes place on back channel for security

front channel:
when a request happens from browser to a web server
Is used to interact with the user to present them the login screen, consent screen , redirect screen

![Screenshot 2023-06-03 at 4 52 33 PM](https://github.com/soniamartis/system-design/assets/12456295/0eb22ef4-d9a4-49a4-95e7-578561414acc)


- All the bold lines mean front channel communications, whereas the dotted lines mean back channel communication
- When the authz server redirects the user to the redirect uri, it provides an authz code as a query param in the redirect url itself
- This means, that anyone can easily grab that authz code and then connect to the contacts api, if the oauth flow was designed to use the authz code intead of access_token.
- So, once the user is redirected to the callback url, the client app makes a call to the authz server with the authz code over a back channel to obtain the access_token. it also passes the secret key which only the client has so that the authz server knows that it is indeed returning the access_token to the client and not anyone else
- Even the access_token is exchanged with the resource server on a back channel and not returned back to the browser, for the same reason, that someone might steal the access_token and do mischief with it

NOTE:
Whoever is in possession of the access_token can actually go fetch the the data from the resource server, so it has be done on a secure channel


More low level request-response details:
When u click on 'Connect with Google' -> browser will get redirected here on authz server

![Screenshot 2023-06-03 at 5 06 34 PM](https://github.com/soniamartis/system-design/assets/12456295/bac7029b-86c6-4034-8301-0caa06c85954)

- yelp will have to go to google(authz server) and do a one-time setup to create a client on google(which will create a client_id and client_secret)
- The clientid and clientSecret identifies the client to the authz server
- The clientid is not a secret and it identifies the yelp application
- ClientSecret is a secret key that is used in the back channel during the token exchange step

On hitting this url, and signing into google and granting access, this is how the redirect url looks like:
![Screenshot 2023-06-03 at 6 40 04 PM](https://github.com/soniamartis/system-design/assets/12456295/20d39590-26c5-4444-b74e-f2bca806a658)

Notice the authz code in the address bar, which means it happens on front-channel
![Screenshot 2023-06-03 at 6 41 57 PM](https://github.com/soniamartis/system-design/assets/12456295/4aeaa12c-eb15-4f91-b7f5-c669a988edc6)


Exchange the code for the access_token with the authz server:

![Screenshot 2023-06-03 at 6 44 16 PM](https://github.com/soniamartis/system-design/assets/12456295/eac4ddc0-d153-41d9-82da-5dcf13ac58fd)


The authz server will return an access_token in response:

![Screenshot 2023-06-03 at 6 47 51 PM](https://github.com/soniamartis/system-design/assets/12456295/5c6a5b50-5775-4b6d-83c7-3f1353abb2bc)


Now the client can connect to the api(resource server) using the access_token
The resource server will validate the token for the following:
- That the access token has been issued by the authorisation server.
- That the token hasn't expired.
- Its scope covers the request.

There is no clear standard for the access_token: it is pure gibberish to the client, and makes sense only for the resource server and authz server
There are 2 types of bearer token:
- Identifier-based
- Self-contained

| Identifier-based | Self-contained|
|------------------|---------------|
|The token can be a record in the authz server's database| token encodes the entire authorisation in itself and is cryptographically protected against tampering like a jwt|
| resource server will have to make a network call to authz server to check for validity , so doesnt scale well in a distributed env | resource server will use a jwt decoder on its side only in order to validate, so is suitable in distributed envs|
| token can be revoked immediately| token is revoked only on its expiry|


  
![Screenshot 2023-06-03 at 6 49 28 PM](https://github.com/soniamartis/system-design/assets/12456295/0d5f167d-26fa-481c-8990-8f404b24aca3)


---
Different types of OAuth code flows:
Based on the kind of response u would like to get from authz server
---
1. Authorization code(front channel + back channel) (response_type=code)
2. Implicit(front channel, suppose u just have a java script app that does not have any backend, in that case, u can skip the entire access_token creation stuff and directly use the authz code) (response_type=token)
3. Resource owner password credentials(back channel only, used for backward compatbility with older apps, not suggested for new services)
4. Client credentials(back channel only, used for service to service communication and skips the authz_code flow, directly gets the access_token) (response_type=token)

![Screenshot 2023-06-03 at 6 58 35 PM](https://github.com/soniamartis/system-design/assets/12456295/b5f01e04-3955-4859-a56c-ffa4bbce1cba)


OAuth was originally built for authz, but started getting used for authn stuff too by companies like FB and Google
![Screenshot 2023-06-03 at 7 05 54 PM](https://github.com/soniamartis/system-design/assets/12456295/f739ded0-b13b-456d-ba67-da1880f49905)


So now, we need to use it like this:
OpenID Connect -> authn
OAuth -> authz

![Screenshot 2023-06-04 at 4 36 05 AM](https://github.com/soniamartis/system-design/assets/12456295/2ad5df9c-f9d2-4f95-88b9-b13bdad8735b)


Some smart ppl decided to add an extension on top of oauth so that it can support authn as well. That extension to oauth is OpenId Connect

Authz servers that implement OpenID connect return an id token instead of an access_token


In this flow, we request the normal ouath scope as well as an openid scope
- The same oauth flow happens, and now, instead of just an accesS_token , the authz server also returns as id_token
which the client can consume instantly to know who has logged in to the system
- This id_token is a jwt(json web token)
- The id_token is encoded in a certain way, in order to see what it means, u can go to https://jwt.io/
- The access_token is used in a similar way like before

NOTE:
The id_token will always be a jwt but the access_token can either be jwt or identifier-based

How the id_token looks like:
![Screenshot 2023-06-04 at 4 43 41 AM](https://github.com/soniamartis/system-design/assets/12456295/eda7982d-f8bf-40a3-8651-359d580e7b2b)


On decoding:
Header, signature and claims
![Screenshot 2023-06-04 at 4 44 17 AM](https://github.com/soniamartis/system-design/assets/12456295/cd02fcfa-0e17-4a63-bb2b-9de9fc9b35e0)


The identity use-cases are now modified to this:
Identity use-cases:
- Simple login(openID connect)
- Single sign on across sites(OpenID Connect)
- Mobile app login(OpenID Connect)
- Delegated authorization(OAuth 2.0)





## Which grant type (flow) should i use
- Web application w/ server backend -> authorization code flow
- Native mobile apps -> authz code flow with PKCE
- Javascript app w/ api backend -> implicit flow
- Microservices and APIs -> client credentials flow

---
## Nice read with illustrations to understand the above concepts:
https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc


## Q&A
How does the resource server validate the access_token(expiry and scope)
A: https://connect2id.com/learn/oauth-2#access-token


What is the client_credentials flow?
- client_credentials flow is used when u do not have a resource_owner(ie a user)
- so we can bypass all the authz code flow , rather use just back-channel flow
- ie, the service A wants to access resource on service B
- in that case, service A will register with the authz server with client_id and client_secret
- Now, before accessing the resource on svc B, it will make a request to the authz server with client_id, client_secret and scopes
the authzserver will return an access_token
- now the svc A can make a request to svc B using the access_token as Authorization: Bearer <token> request header in api call

eg:
Client credentials Flow :
- Shadow app presents the (clientId-clientSecret-scopes) to idfs-qa
- idfs-qa returns an access token
- shadow app makes a request to Secmaster app using the access token in the Authorization header like: Authorization: Bearer <access_token>



What is the flow that we are using to connect to DEV/QA apis?
- We are using the authorization code flow and not the client_credentials flow so that the identity of the person making the call is properly captured in the service and it does not just use the identity of a client app
- We have created a client credential for postman app in login.idfs-qa
- When we click on create new access token:
- As we are using the authorization code flow:
- We are first redirected to authz server ie login.idfa-qa
- there we provide username and password
- then we are redirected to client app's callback url where the back-channel connection is established to get the access_token(jwt token)
- then we use that access_token as part of the request header while making a rest api call



- Request-Response apis (initiated by client)
  - REST
  - RPC
  - Graphql
- Event streaming apis
  - Webhook (initiated by server)
  - Websocket (client initiated)
  - HTTP streaming
    - Transfer-Encoding: chunked
    - Server Sent Events

## Comparison of Request-Response paradigms
| | Rest| RPC| Graphql|
|---|---|---|---|
| What| Exposes data as resources and uses standard http methods for crud operations | Exposes action-based api methods - clients pass method names and arguments| Query language for apis -  clients define structure of response|
| usage| GET /v1/users/<id> | GET /v1/users.get?id=<id> | ```query($id: String!){user(login: $id){name company createdAt}}```|
| HTTP verbs| GET, POST, PUT, DELETE, PATCH | GET, POST | GET, POST sample get: ```GET http://myapi/graphql?query={me{name}}```|
| Pros| standard method names, args, uses http features, easy to maintain| easy to understand, lightweight payloads, high performance |saves multiple round trips, avoids versioning, smaller payload sizes, stronyly typed, built-in introspection(has a graphiql interface for discovering apis)|
| Cons|big payloads, multiple http round trips| discovery is difficult, limited standardisation| too complicated for a simple api, requires addn query parsing|
| When to use| crud operations on resources| to expose actions| query flexibility|

## Webhooks
### How do they work
- The client(the side that needs to be notified of change) needs to expose an http endpoint and the service porvider will then make a GET/POST request for the endpoint once data is available
- eg, in payment service, the payement service will expose an endpoint and the PSP(payment service provider) will send a message on this EP with status of the payment
- One-way communicattion
- Mostly used between services and not in browser-server settings
- Receiving app closes the connection after providing a response

## Websockets
![image](https://github.com/soniamartis/system-design/assets/12456295/45410539-b4ad-4186-9767-bde21a6e0d94)

- Used for two-way communication
- client initiated
- starts as a http request and then is upgraded to a WS protocol
- Client sends an httprequest to the server with the upgrade header set to websocket, and server responds with a upgrade header specifying that it is now swiitching to WS protocol
- Used for real-time web apps like gaming, chat apps, trading apps etc.
- only protocol so far that is built for 2-way communication

## HTTP Streaming
- In request-response apis, client makes an http request and server sends a response of finite length
- In this paradigm, server can continue to push new data in a single long-lived connection opened by client
- Options
  - Server sets the Transfer-encoding: chunked, which indicates to clients that data will arrive in chunks of newline-delimited strings
  - Server sent events(SSE)

## Comparison of event-driven apis
| | Webhooks| Websockets| HTTP Streaming|
| --|---|--|--|
| What| event notfication via http callback| two-way streaming connection over tcp| long-lived connection over http|
| pros| uses http protocol, easy s2s communication|2-way communication, bypasses firewalls, native browser support| can stream over simple http, native browser support, can bypass firewalls|
|cons| doesnt work on browsers and firewalls, handling failure, security is hard| not http, need to maintain persistent connection|bidirectional communication not supported|
| when to use| to trigger server to send real-time events| two-way real-time communication|for one-way communication over http|


Todo this week:
- small pocs on websocket, http streaming, SSE and gRPC in spring
- https://www.baeldung.com/spring-mvc-sse-streams

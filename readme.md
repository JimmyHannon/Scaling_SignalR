# Example of scaling out Application using SignalR 

## Running the example

```
docker-compose up
```

When making changes to the files the images need to be rebuild:

```
docker-compose build
```

or for a specific image for example the load balancer (lb) 

```
docker-compose build lb
```

## Solution 1 (sticky sessions)

Based on the comments in https://github.com/aspnet/SignalR/issues/2002 it is stated that when scaling an application that uses signalR it is necessary to configure:

 - Sticky sessions on the load balancer
 - Redis (or other storage) for the backplane 

 To enable sticky sessions on the load balancer copy this in the *haproxy.cfg* file:

 ```
# this config needs haproxy-1.1.28 or haproxy-1.2.1
global
  log  127.0.0.1  local0
  log  127.0.0.1  local1 notice
  maxconn  4096
  uid  99
  gid  99
  daemon

defaults
  log   global
  mode  http
  option  httplog
  option  dontlognull
  retries  3
  option  redispatch
  option  http-server-close
  maxconn  2000
  timeout connect  5000
  timeout client  50000
  timeout server  50000

frontend public
  bind *:80
  default_backend api

backend api
  timeout server 30s
  balance roundrobin
  cookie SERVERID insert indirect nocache
  server s1 webapp1 cookie s1
  server s2 webapp2 cookie s2
 ``` 

Then rebuild the load balancer image and run.

Select the appropreate connection in file *site.js* located in the *wwwoot* folder of the **Api** project.

```
// Use this connection when not using sticky sessions
var connection = new signalR.HubConnectionBuilder().withUrl("/socket", {
    skipNegotiation: true,
    transport: signalR.HttpTransportType.WebSockets
}).build();

// Use this connection when using sticky sessions
//var connection = new signalR.HubConnectionBuilder().withUrl("/socket").build();
```

## Solution 2 without sticky sessions

It is possible to work without sticky sessions however the restriction is that for the websockets only the *websocket* protocol can be used.
So long polling and other fall backs are not possible.

To disable sticky sessions in the load balancer, copy this in the *haproxy.cfg* file:

```
# this config needs haproxy-1.1.28 or haproxy-1.2.1
global
  log  127.0.0.1  local0
  log  127.0.0.1  local1 notice
  maxconn  4096
  uid  99
  gid  99
  daemon

defaults
  log   global
  mode  http
  option  httplog
  option  dontlognull
  retries  3
  option  redispatch
  option  http-server-close
  maxconn  2000
  timeout connect  5000
  timeout client  50000
  timeout server  50000

frontend public
  bind *:80
  default_backend api

backend api
  timeout server 30s
  balance roundrobin
  server s1 webapp1
  server s2 webapp2
```

Select the appropreate connection in file *site.js* located in the *wwwoot* folder of the **Api** project.

```
// Use this connection when not using sticky sessions
var connection = new signalR.HubConnectionBuilder().withUrl("/socket", {
    skipNegotiation: true,
    transport: signalR.HttpTransportType.WebSockets
}).build();

// Use this connection when using sticky sessions
//var connection = new signalR.HubConnectionBuilder().withUrl("/socket").build();
```
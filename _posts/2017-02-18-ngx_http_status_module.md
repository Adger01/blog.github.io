Module ngx_http_status_module

Example Configuration
Directives
     status
     status_format
     status_zone
Data
Compatibility
The ngx_http_status_module module provides access to various status information.
  此模块可作为我们的商业订阅的一部分。
 配置例子:  
{% highlight nginx%}
http {
    upstream backend {
        zone http_backend 64k;

        server backend1.example.com weight=5;
        server backend2.example.com;
    }

    proxy_cache_path /data/nginx/cache_backend keys_zone=cache_backend:10m;

    server {
        server_name backend.example.com;

        location / {
            proxy_pass  http://backend;
            proxy_cache cache_backend;

            health_check;
        }

        status_zone server_backend;
    }

    server {
        listen 127.0.0.1;

        location /upstream_conf {
            upstream_conf;
        }

        location /status {
            status;
        }

        location = /status.html {
        }
    }
}

stream {
    upstream backend {
        zone stream_backend 64k;

        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
    }

    server {
        listen      127.0.0.1:12345;
        proxy_pass  backend;
        status_zone server_backend;
        health_check;
    }
}
{% endhighlight%}
  
  具有此配置的状态请求的示例：

http://127.0.0.1/status
http://127.0.0.1/status/nginx_version
http://127.0.0.1/status/caches/cache_backend
http://127.0.0.1/status/upstreams
http://127.0.0.1/status/upstreams/backend
http://127.0.0.1/status/upstreams/backend/peers/1
http://127.0.0.1/status/upstreams/backend/peers/1/weight
http://127.0.0.1/status/stream
http://127.0.0.1/status/stream/upstreams
http://127.0.0.1/status/stream/upstreams/backend
http://127.0.0.1/status/stream/upstreams/backend/peers/1
http://127.0.0.1/status/stream/upstreams/backend/peers/1/weight
The simple monitoring page is shipped with this distribution, accessible as “/status.html” in the default configuration. It requires the locations “/status” and “/status.html” to be configured as shown above.

###命令###
{% highlight nginx%}
Syntax:	status;
Default:	—
Context:	location
{% endhighlight %}
The status information will be accessible from the surrounding location. Access to this location should be limited.

{% highlight %}
Syntax:	status_format json;
status_format jsonp [callback];
Default:	
status_format json;
Context:	http, server, location
{% endhighlight %}
  缺省的,输出的状态详情是```json```格式的  
  
Alternatively, data may be output as JSONP. The callback parameter specifies the name of a callback function. The value can contain variables. If parameter is omitted, or the computed value is an empty string, then “ngx_status_jsonp_callback” is used.
{% highlight nginx %}
Syntax:	status_zone zone;
Default:	—
Context:	server
{% endhighlight %}
Enables collection of virtual http or stream (1.7.11) server status information in the specified zone. Several servers may share the same zone.

Data 数据  
The following status information is provided:  
  提供下列状态信息:
  
```version 版本```
> Version of the provided data set. The current version is 7.
> 提供的版本信息,这个版本是7    

```nginx_version nginx版本```
> Version of nginx. nginx的版本

address
The address of the server that accepted status request.
generation
The total number of configuration reloads.
load_timestamp
Time of the last reload of configuration, in milliseconds since Epoch.
timestamp
Current time in milliseconds since Epoch.
pid
The ID of the worker process that handled status request.
processes
respawned
The total number of abnormally terminated and respawned child processes.
connections
accepted
The total number of accepted client connections.
dropped
The total number of dropped client connections.
active
The current number of active client connections.
idle
The current number of idle client connections.
ssl
handshakes
The total number of successful SSL handshakes.
handshakes_failed
The total number of failed SSL handshakes.
session_reuses
The total number of session reuses during SSL handshake.
requests
total
The total number of client requests.
current
The current number of client requests.
server_zones
For each status_zone:
processing
The number of client requests that are currently being processed.
requests
The total number of client requests received from clients.
responses
total
The total number of responses sent to clients.
1xx, 2xx, 3xx, 4xx, 5xx
The number of responses with status codes 1xx, 2xx, 3xx, 4xx, and 5xx.
discarded
The total number of requests completed without sending a response.
received
The total number of bytes received from clients.
sent
The total number of bytes sent to clients.
upstreams
For each dynamically configurable group, the following data are provided:
peers
For each server, the following data are provided:
id
The ID of the server.
server
An address of the server.
backup
A boolean value indicating whether the server is a backup server.
weight
Weight of the server.
state
Current state, which may be one of “up”, “draining”, “down”, “unavail”, or “unhealthy”.
active
The current number of active connections.
max_conns
The max_conns limit for the server.
requests
The total number of client requests forwarded to this server.
responses
total
The total number of responses obtained from this server.
1xx, 2xx, 3xx, 4xx, 5xx
The number of responses with status codes 1xx, 2xx, 3xx, 4xx, and 5xx.
sent
The total number of bytes sent to this server.
received
The total number of bytes received from this server.
fails
The total number of unsuccessful attempts to communicate with the server.
unavail
How many times the server became unavailable for client requests (state “unavail”) due to the number of unsuccessful attempts reaching the max_fails threshold.
health_checks
checks
The total number of health check requests made.
fails
The number of failed health checks.
unhealthy
How many times the server became unhealthy (state “unhealthy”).
last_passed
Boolean indicating if the last health check request was successful and passed tests.
downtime
Total time the server was in the “unavail” and “unhealthy” states.
downstart
The time (in milliseconds since Epoch) when the server became “unavail” or “unhealthy”.
selected
The time (in milliseconds since Epoch) when the server was last selected to process a request (1.7.5).
header_time
The average time to get the response header from the server (1.7.10). The field is available when using the least_time load balancing method.
response_time
The average time to get the full response from the server (1.7.10). The field is available when using the least_time load balancing method.
keepalive
The current number of idle keepalive connections.
zombies
The current number of servers removed from the group but still processing active client requests.
queue
For the requests queue, the following data are provided:
size
The current number of requests in the queue.
max_size
The maximum number of requests that can be in the queue at the same time.
overflows
The total number of requests rejected due to the queue overflow.
caches
For each cache (configured by proxy_cache_path and the likes):
size
The current size of the cache.
max_size
The limit on the maximum size of the cache specified in the configuration.
cold
A boolean value indicating whether the “cache loader” process is still loading data from disk into the cache.
hit, stale, updating, revalidated
responses
The total number of responses read from the cache (hits, or stale responses due to proxy_cache_use_stale and the likes).
bytes
The total number of bytes read from the cache.
miss, expired, bypass
responses
The total number of responses not taken from the cache (misses, expires, or bypasses due to proxy_cache_bypass and the likes).
bytes
The total number of bytes read from the proxied server.
responses_written
The total number of responses written to the cache.
bytes_written
The total number of bytes written to the cache.
stream
server_zones
For each status_zone:
processing
The number of client connections that are currently being processed.
connections
The total number of connections accepted from clients.
sessions
total
The total number of completed client sessions.
2xx, 4xx, 5xx
The number of sessions completed with status codes 2xx, 4xx, or 5xx.
discarded
The total number of connections completed without creating a session.
received
The total number of bytes received from clients.
sent
The total number of bytes sent to clients.
upstreams
For each dynamically configurable group, the following data are provided:
peers
For each server the following data are provided:
id
The ID of the server.
server
An address of the server.
backup
A boolean value indicating whether the server is a backup server.
weight
Weight of the server.
state
Current state, which may be one of “up”, “down”, “unavail”, or “unhealthy”.
active
The current number of connections.
connections
The total number of client connections forwarded to this server.
connect_time
The average time to connect to the upstream server. The field is available when using the least_time load balancing method.
first_byte_time
The average time to receive the first byte of data. The field is available when using the least_time load balancing method.
response_time
The average time to receive the last byte of data. The field is available when using the least_time load balancing method.
sent
The total number of bytes sent to this server.
received
The total number of bytes received from this server.
fails
The total number of unsuccessful attempts to communicate with the server.
unavail
How many times the server became unavailable for client connections (state “unavail”) due to the number of unsuccessful attempts reaching the max_fails threshold.
health_checks
checks
The total number of health check requests made.
fails
The number of failed health checks.
unhealthy
How many times the server became unhealthy (state “unhealthy”).
last_passed
Boolean indicating if the last health check request was successful and passed tests.
downtime
Total time the server was in the “unavail” and “unhealthy” states.
downstart
The time (in milliseconds since Epoch) when the server became “unavail” or “unhealthy”.
selected
The time (in milliseconds since Epoch) when the server was last selected to process a connection.
Compatibility

The sessions status data and the discarded field in stream server_zones were added in version 7.
The zombies field was moved from nginx debug version in version 6.
The ssl status data were added in version 6.
The discarded field in server_zones was added in version 6.
The queue status data were added in version 6.
The pid field was added in version 6.
The list of servers in upstreams was moved into peers in version 6.
The keepalive field of an upstream server was removed in version 5.
The stream status data were added in version 5.
The generation field was added in version 5.
The respawned field in processes was added in version 5.
The header_time and response_time fields in upstreams were added in version 5.
The selected field in upstreams was added in version 4.
The draining state in upstreams was added in version 4.
The id and max_conns fields in upstreams were added in version 3.
The revalidated field in caches was added in version 3.
The server_zones, caches, and load_timestamp status data were added in version 2.

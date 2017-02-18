这个模块缺省没有被安装开启,如果想要开启这个模块,需要在编译的时候使用`--with-mail`编译参数
   
配置例子:  

{% highlight nginx %}
worker_processes 1;

error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

mail {
    server_name       mail.example.com;
    auth_http         localhost:9000/cgi-bin/nginxauth.cgi;

    imap_capabilities IMAP4rev1 UIDPLUS IDLE LITERAL+ QUOTA;

    pop3_auth         plain apop cram-md5;
    pop3_capabilities LAST TOP USER PIPELINING UIDL;

    smtp_auth         login plain cram-md5;
    smtp_capabilities "SIZE 10485760" ENHANCEDSTATUSCODES 8BITMIME DSN;
    xclient           off;

    server {
        listen   25;
        protocol smtp;
    }
    server {
        listen   110;
        protocol pop3;
        proxy_pass_error_message on;
    }
    server {
        listen   143;
        protocol imap;
    }
    server {
        listen   587;
        protocol smtp;
    }
}
{% endhighlight %}
Directives

Syntax:	listen address:port [ssl] [backlog=number] [bind] [ipv6only=on|off] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
Default:	—
Context:	server
Sets the address and port for the socket on which the server will accept requests. It is possible to specify just the port. The address can also be a hostname, for example:

listen 127.0.0.1:110;
listen *:110;
listen 110;     # same as *:110
listen localhost:110;
IPv6 addresses (0.7.58) are specified in square brackets:

listen [::1]:110;
listen [::]:110;
UNIX-domain sockets (1.3.5) are specified with the “unix:” prefix:

listen unix:/var/run/nginx.sock;
Different servers must listen on different address:port pairs.

The ssl parameter allows specifying that all connections accepted on this port should work in SSL mode.

The listen directive can have several additional parameters specific to socket-related system calls.

backlog=number
sets the backlog parameter in the listen() call that limits the maximum length for the queue of pending connections (1.9.2). By default, backlog is set to -1 on FreeBSD, DragonFly BSD, and Mac OS X, and to 511 on other platforms.
bind
this parameter instructs to make a separate bind() call for a given address:port pair. The fact is that if there are several listen directives with the same port but different addresses, and one of the listen directives listens on all addresses for the given port (*:port), nginx will bind() only to *:port. It should be noted that the getsockname() system call will be made in this case to determine the address that accepted the connection. If the ipv6only or so_keepalive parameters are used then for a given address:port pair a separate bind() call will always be made.
ipv6only=on|off
this parameter determines (via the IPV6_V6ONLY socket option) whether an IPv6 socket listening on a wildcard address [::] will accept only IPv6 connections or both IPv6 and IPv4 connections. This parameter is turned on by default. It can only be set once on start.
so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]
this parameter configures the “TCP keepalive” behavior for the listening socket. If this parameter is omitted then the operating system’s settings will be in effect for the socket. If it is set to the value “on”, the SO_KEEPALIVE option is turned on for the socket. If it is set to the value “off”, the SO_KEEPALIVE option is turned off for the socket. Some operating systems support setting of TCP keepalive parameters on a per-socket basis using the TCP_KEEPIDLE, TCP_KEEPINTVL, and TCP_KEEPCNT socket options. On such systems (currently, Linux 2.4+, NetBSD 5+, and FreeBSD 9.0-STABLE), they can be configured using the keepidle, keepintvl, and keepcnt parameters. One or two parameters may be omitted, in which case the system default setting for the corresponding socket option will be in effect. For example,
so_keepalive=30m::10
will set the idle timeout (TCP_KEEPIDLE) to 30 minutes, leave the probe interval (TCP_KEEPINTVL) at its system default, and set the probes count (TCP_KEEPCNT) to 10 probes.
Syntax:	mail { ... }
Default:	—
Context:	main
Provides the configuration file context in which the mail server directives are specified.

Syntax:	protocol imap | pop3 | smtp;
Default:	—
Context:	server
Sets the protocol for a proxied server. Supported protocols are IMAP, POP3, and SMTP.

If the directive is not set, the protocol can be detected automatically based on the well-known port specified in the listen directive:

imap: 143, 993
pop3: 110, 995
smtp: 25, 587, 465
Unnecessary protocols can be disabled using the configuration parameters --without-mail_imap_module, --without-mail_pop3_module, and --without-mail_smtp_module.

Syntax:	resolver address ... [valid=time];
resolver off;
Default:	
resolver off;
Context:	mail, server
Configures name servers used to find the client’s hostname to pass it to the authentication server, and in the XCLIENT command when proxying SMTP. For example:

resolver 127.0.0.1 [::1]:5353;
An address can be specified as a domain name or IP address, and an optional port (1.3.1, 1.2.2). If port is not specified, the port 53 is used. Name servers are queried in a round-robin fashion.

Before version 1.1.7, only a single name server could be configured. Specifying name servers using IPv6 addresses is supported starting from versions 1.3.1 and 1.2.2.
By default, nginx caches answers using the TTL value of a response. An optional valid parameter allows overriding it:

resolver 127.0.0.1 [::1]:5353 valid=30s;
Before version 1.1.9, tuning of caching time was not possible, and nginx always cached answers for the duration of 5 minutes.
The special value off disables resolving.

Syntax:	resolver_timeout time;
Default:	
resolver_timeout 30s;
Context:	mail, server
Sets a timeout for DNS operations, for example:

resolver_timeout 5s;
Syntax:	server { ... }
Default:	—
Context:	mail
Sets the configuration for a server.


{% highlight nginx %}
Syntax:	server_name name;
Default:	
server_name hostname;
Context:	mail, server
{% endhighlight %}  
设置使用的服务名称【域名】

在最初的POP3/SMTP服务器的问候；
in the salt during the SASL CRAM-MD5 authentication;
in the EHLO command when connecting to the SMTP backend, if the passing of the XCLIENT command is enabled.
If the directive is not specified, the machine’s hostname is used.

{% highlight nginx %}
Syntax:	timeout time;
Default:	
timeout 60s;
Context:	mail, server
{% endhighlight %}  
使用代理连接后端之前的超时时间

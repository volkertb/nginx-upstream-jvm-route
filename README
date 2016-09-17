# Available versions: 

## Version 1.6.3
You can obtain the image as follows:

    docker pull volkertb/nginx-upstream-jvm-route:1.6.3

# About the Docker image

This image is built on the Centos 7 base image (centos:7). See https://hub.docker.com/_/centos/

This image contains a ready-to-run build of the Nginx light-weight reverse proxy server, which has been patched to support so-called "sticky-session" load balancing when used in conjunction with Tomcat nodes, or nodes running on Tomcat-derived application servers, such as certain (older) versions of JBoss. This is accomplished through support of the "jvmRoute" attribute, which enables application server nodes to "tag" sessions ID cookies with an additional node-specific identifier, so that the load balancer (in this case the patched version of Nginx) will know how to maintain connections between logged-in users and the specific server nodes to which they are logged in.

The commercial version of Nginx already supports sticky sessions, but in this Docker image, an available unofficial open-source patch has been applied, which enables this feature in the free and open-source version of Nginx as well.

The patch that was applied in this image can be found here:

    https://github.com/nulab/nginx-upstream-jvm-route

As of 2016-09-17, the ginx-upstream-jvm-route patch on GitHub was last updated in 2014, to make it compatible with version 1.6 of Nginx. The last release of Nginx at nginx.org that the patch can still be cleanly applied to is version 1.6.3, so the source code of that specific version was used to build and patch Nginx for this image.

Although the GitHub project page of the nginx-upstream-jvm-route patch contains some brief information and configuration examples, it is not very detailed and I ran into some problems while trying to apply those examples to my project. For further information about what ended up working for me at least, read on.

In this image, the default configuration file is located at /usr/local/nginx/conf/nginx.conf

Below is an example of a customized nginx.conf that configures Nginx to proxy from a public HTTPS/SSL URL to a pool of three Tomcat nodes, each running locally at port 8080 in their respective container instances. You will have to COPY your own nginx.conf to /usr/local/nginx/conf/ in your Dockerfile when you base it on this image.

    worker_processes  1;

    events {
        worker_connections  1024;
    }

    http {
        include       mime.types;
        default_type  application/octet-stream;

        #access_log  logs/access.log  main;

        sendfile        on;

        keepalive_timeout  65;

        upstream backend {
            server tomcat-node1:8080 srun_id=node1; # In server.xml: <Engine name="tomcat1" defaultHost="localhost" jvmRoute="node1">
            server tomcat-node2:8080 srun_id=node2; # In server.xml: <Engine name="tomcat2" defaultHost="localhost" jvmRoute="node2">
            server tomcat-node3:8080 srun_id=node3; # In server.xml: <Engine name="tomcat3" defaultHost="localhost" jvmRoute="node3">

            jvm_route $cookie_JSESSIONID reverse;
        }
        
        server {
            listen       80;
            listen 443 default_server ssl;
            
            # Assuming I'm using Avahi and the Docker machine's hostname is "nginx-loadbalancer".
            # See https://twitter.com/volkertb/status/777120785878151170
            server_name nginx-loadbalancer.local;

            ssl_certificate /usr/local/nginx/ssl/nginx.crt;
            ssl_certificate_key /usr/local/nginx/ssl/nginx.key;

            location / {
                # With thanks to https://serverfault.com/a/782154
                proxy_pass http://backend;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
            }

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

        }

    }

This above configuration expects the SSL-related files nginx.key and nginx.crt to be placed in /usr/local/nginx/ssl/. Take this into consideration when you base your own SSL-specific container image on this image.

You can generate a self-signed certificate for Nginx as follows:

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx.key -out ./nginx.crt

Naturally, you will have to COPY nginx.key and nginx.crt to /usr/local/nginx/ssl/ in your Dockerfile.

If you have any questions or suggested improvements, please don't hesitate to comment below. I'm happy to improve this further in any way. I'm just trying to improve on the hard work of others here. Also, this is my first publicly shared Docker image and I'm always willing to learn how to do things better. Thank you! 😄

=nginx_upstream_jvm_route=
-------

This module achieves session stickiness with the session cookie. If the session is not in the cookie
or URL, the module will be a normal Round-Robin upstream module.

=INSTALLATION=
    
    cd nginx-0.7.59 # or whatever
    patch -p0 < /path/to/this/directory/jvm_route.patch
  
compile nginx with the following addition option:

  --add-module=/path/to/this/directory

=DIRECTIVES=

    ==jvm_route==

    syntax: jvm_route $cookie_SESSION_COOKIE[|session_url] [reverse]
    default: none
    context: upstream
    description: 
    '$cookie_SESSION_COOKIE' specifies the session cookie name(0.7.24+). 'session_url' specifies a
    different session name in the URL when the client does not accept a cookie. The session name is
    case-insensitive. In this module, if it does not find the session_url, it will use the session
    cookie name instead. So if the session name in cookie is the name with its in URL, you don't
    need give the session_url name.  
    With scanning this cookie, the module will send the request to right backend server. As far as I
    know, the resin's srun_id name is in the head of cookie. For example, requests with cookie value
    'a***' are always sent to the server with the srun_id of 'a'. But tomcat's JSESSIONID is
    opposite, which is like '***.a'. The parameter of 'reverse' specifies the cookie scanned from
    tail to head.
    If the request fails to be sent to the chosen backend server, It will try another server with
    the Round-Robin mode until all the upstream servers tried. The directive proxy_next_upstream can
    specify in what cases the request will be transmitted to the next server. If you want to force
    the session sticky, you can set 'proxy_next_upstream off'.


    ==jvm_route_status==

    syntax: jvm_route_status upstream_name
    default: none
    context: location
    example:
        location status {
            jvm_route_status backend;
        }
    description: 
    return the status of the jvm_route peers, like this: 

        upstream backend: total_busy = 10, total_requests = 311, current_peer 15/18

        peer 0: 172.19.0.126:80(g) down: 0, fails: 0/1, busy: 0/0, weight: 4/4, total_req: 60, last_req: 298, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 1: 172.19.0.120:80(a) down: 0, fails: 0/1, busy: 1/1, weight: 1/1, total_req: 15, last_req: 295, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 2: 172.19.0.121:80(b) down: 0, fails: 0/1, busy: 0/1, weight: 0/1, total_req: 16, last_req: 299, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 3: 172.19.0.122:80(c) down: 0, fails: 0/1, busy: 0/1, weight: 0/1, total_req: 16, last_req: 300, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 4: 172.19.0.123:80(d) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 301, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 5: 172.19.0.124:80(e) down: 0, fails: 1/1, busy: 0/1, weight: 1/1, total_req: 2, last_req: 216, total_fails: 2, fail_acc_time: Wed Nov 18 11:19:37 2009
        peer 6: 172.19.0.125:80(f) down: 0, fails: 0/1, busy: 0/0, weight: 0/1, total_req: 16, last_req: 302, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 7: 172.19.0.127:80(h) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 303, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 8: 172.19.0.128:80(i) down: 0, fails: 0/1, busy: 0/1, weight: 0/1, total_req: 16, last_req: 304, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 9: 172.19.0.129:80(j) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 305, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 10: 172.19.0.130:80(k) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 306, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 11: 172.19.0.131:80(l) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 307, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 12: 172.19.0.132:80(m) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 308, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 13: 172.19.0.235:80(n) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 309, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 14: 172.19.0.236:80(o) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 14, last_req: 310, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 15: 172.19.0.237:80(p) down: 0, fails: 0/1, busy: 1/1, weight: 0/1, total_req: 16, last_req: 311, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 16: 172.19.0.238:80(q) down: 0, fails: 0/1, busy: 0/1, weight: 1/1, total_req: 15, last_req: 292, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970
        peer 17: 172.19.0.239:80(r) down: 0, fails: 0/1, busy: 0/1, weight: 1/1, total_req: 15, last_req: 293, total_fails: 0, fail_acc_time: Thu Jan  1 08:00:00 1970

    total_busy is the sum of all the backend servers' active connections.
    total_requests is all the count of requests which had proxied to backend.
    current_peer is meaningful with the Round Robin mode when the session cookie is absent.

    down is the state of backend server whether is configured with 'down'.
    fails is the failure requests count in the interval of 'fail_timeout'.
    busy is the current active connections of the backend server.
    weight is the current weight of server and just meaningful with the Round Robin module.
    total_req is the count of requests which had proxied to this backend server.
    last_req is the last request's id proxied by this server.
    total_fails is the count of failure requests which had proxied to the this backend server.
    fail_acc_time stands for the last failure access time.

    ==server==

    Main syntax is the same as the official directive. 
    This module add these parameters:
    'srun_id': identifies the backend JVM's name by cookie. The default srun_id's value is 'a'. The
    name can be more than one letter.
    'max_busy': the maximum of active connections with the backend server. The default value is 0
    which means unlimited. If the server's active connections is higher than this parameter, it will
    not be chosen until the server is less busier. If all the servers are busy, Nginx will return
    502.
     
    NOTE: This module does not support the parameter of 'backup' yet.
 
=EXAMPLE=

1.For resin with nginx

upstream backend {
    server 192.168.0.100 srun_id=a;
    server 192.168.0.101 srun_id=b;
    server 192.168.0.102 srun_id=c;
    server 192.168.0.103 srun_id=d;

    jvm_route $cookie_JSESSIONID;
}

For all resin servers' configure:

    <server id="a" address="192.168.0.100" port="8080">
    <http id="" port="80"/>
    </server>
    <server id="b" address="192.168.0.101" port="8080">
    <http id="" port="80"/>
    </server>
    <server id="c" address="192.168.0.102" port="8080">
    <http id="" port="80"/>
    </server>
    <server id="d" address="192.168.0.103" port="8080">
    <http id="" port="80"/>
    </server>

And start each resin instances like this:
    server a
    shell $> /usr/local/resin/bin/httpd.sh -server a start

    server b
    shell $> /usr/local/resin/bin/httpd.sh -server b start

    server c
    shell $> /usr/local/resin/bin/httpd.sh -server c start

    server d
    shell $> /usr/local/resin/bin/httpd.sh -server d start

2.For tomcat with nginx
upstream backend {
    server 192.168.0.100 srun_id=a;
    server 192.168.0.101 srun_id=b;
    server 192.168.0.102 srun_id=c;
    server 192.168.0.103 srun_id=d;

    jvm_route $cookie_JSESSIONID reverse;
}

Each tomcats' configure:
    Tomcat a:
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="a">
    Tomcat b:
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="b">
    Tomcat c:
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="c">
    Tomcat d:
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="d">

3. A simple java test page
index.jsp

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>

<html>
<head>
</head>
<body>
$your_jvm_name
<br />
<%out.print(request.getSession()) ;%> <br />
<%out.println(request.getHeader("Cookie"))&nbsp;%>
</body>
</html>

Note: This is a third-party module. And you need careful test before using this module in your
product environmenta.

Questions/patches may be directed to Weibin Yao, yaoweibin@gmail.com.

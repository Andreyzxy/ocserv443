# 前言
目前的转发逻辑：  
  ![](/data/s1.jpg)  



```  
"redirect": ["*:443#127.0.0.1:10000"], // 重点：将 web 流量转发到本地的 haproxy 端口
```  



```  
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
#    option                  httplog
    option                  dontlognull
    option http-server-close
#    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
	
frontend www-https
   bind 0.0.0.0:10000
   mode tcp
   tcp-request inspect-delay 5s
   default_backend bk_ssl_default

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------

backend bk_ssl_default
   mode tcp
   acl vpn-app req_ssl_sni -i 域名1
   acl web-app req_ssl_sni -i 域名2

   use-server server-vpn if vpn-app
   use-server server-web if web-app
   use-server server-vpn if !vpn-app !web-app

   option ssl-hello-chk
   server server-vpn 127.0.0.1:端口1 send-proxy-v2
   server server-web 127.0.0.1:端口2 check
 ```  


listen-proxy-proto = true  

https://wwww.lvmoo.com/1110.love

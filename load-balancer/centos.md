yum install haproxy

mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.org 
vim /etc/haproxy/haproxy.cfg

```
# create new
global
      # for logging section
    log         127.0.0.1 local2 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
      # max per-process number of connections
    maxconn     256
      # process' user and group
    user        haproxy
    group       haproxy
      # makes the process fork into background
    daemon

defaults
      # running mode
    mode               http
      # use global settings
    log                global
      # get HTTP request log
    option             httplog
      # timeout if backends do not reply
    timeout connect    10s
      # timeout on client side
    timeout client     30s
      # timeout on server side
    timeout server     30s

# define frontend ( set any name for "http-in" section )
frontend http-in
      # listen 80
    bind *:80
      # set default backend
    default_backend    backend_servers
      # send X-Forwarded-For header
    option             forwardfor

# define backend
backend backend_servers
      # balance with roundrobin
    balance            roundrobin
      # define backend servers
    server             www01 10.0.0.31:80 check
    server             www02 10.0.0.32:80 check
```

[root@host ~]# systemctl start haproxy
[root@host ~]# systemctl enable haproxy

=== Rsyslog configuration for getting of logs for HAProxy

vim /etc/rsyslog.conf

```
# line 15,16: uncomment, lne 17: add
$ModLoad imudp
$UDPServerRun 514
$AllowedSender UDP, 127.0.0.1

# line 54: change like follows
*.info;mail.none;authpriv.none;cron.none,local2.none   /var/log/messages
 local2.*                                                /var/log/haproxy.log
```

[root@host ~]# systemctl restart rsyslog


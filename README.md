# f5awstuff2
-------------

you need the key which is fortunately not on github 

ssh://admin@34.234.145.60  brett-bigip1

ssh://ec2-user@54.152.70.153 web1

ssh://ec2-user@54.209.254.153 web2

ssh://ec2-user@54.89.241.193 web3

http://35.153.13.226/   VIP

ssh://ec2-user@34.238.154.1 brett-docker1

This has docker, you can pull or docker run a lamp, it also has haproxy and nginx on it.

      ********** HAPROXY CONFIG FOR DVWA WITH PHPSESSIONID PERSISTENCE   **********
      

[root@ip-172-31-95-223 haproxy]# cat haproxy.cfg


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


defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
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


frontend  main *:80
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             app


backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check


backend app
    balance     roundrobin

    cookie SERVERID insert indirect nocache
    server app1_1 172.31.92.179:80 check cookie s1
    server app1_2 172.31.81.148:80 check cookie s2
    server app1_3 172.31.82.251:80 check cookie s3
    #server  app1 127.0.0.1:5001 check
    #server  app2 127.0.0.1:5002 check
    #server  app3 127.0.0.1:5003 check
    #server  app4 127.0.0.1:5004 check

[root@ip-172-31-95-223 haproxy]#


      ********** NGINX CONFIG FOR DVWA WITH PHPSESSIONID PERSISTENCE   **********
      
      [root@ip-172-31-95-223 nginx]# cat nginx.conf

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;


include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;


    include /etc/nginx/conf.d/*.conf;

    upstream myapp1 {
        server 172.31.92.179;
        server 172.31.81.148;
        server 172.31.82.251;

        sticky cookie srv_id expires=1h path=/;
    }

    index   index.html index.htm;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  localhost;
        #root         /usr/share/nginx/html;

  
        include /etc/nginx/default.d/*.conf;

        location / {
            proxy_pass http://myapp1;
        }

        
        error_page 404 /404.html;
            location = /40x.html {
        }

  
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }

    }


}

[root@ip-172-31-95-223 nginx]#



    ****** FULL SETUP HISTORY for DOCKER/NGINX/HAPROXY Utiltiy Box ********

[ec2-user@ip-172-31-95-223 ~]$ history
    1  docker info
    2  ocker run -d -p 80:5000 training/webapp:latest python app.py
    3  docker run -d -p 80:5000 training/webapp:latest python app.py
    4  curl http://localhost
    5  docker build -t imageName .
    6  sudo docker build -t imageName .
    7  docker run -it ubuntu:14.04
    8  docker status
    9  docker services
   10  docker
   11  docker stats
   12  docker --help
   13  docker --help  more
   14  docker --help   more
   15  docker --help|more
   16  docker service
   17  docker service inspect
   18  docker --help|more
   19  docker service stats
   20  docker service ls
   21  docker service stats ls
   22  docker service ls
   23  docker ps
   24  sudo docker run -it lamp
   25  docker pull fauria/lamp
   26  docker run -i -t --rm fauria/lamp bash
   27  ls
   28  node
   29  curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -
   30  su
   31  sudo yum install -y nodejs
   32  sudo apt-get update
   33  sudo yum install nginx
   34  sudo yum install haproxy
   35  sudo ufw app list
   36  systemctl status nginx
   37  sudo systemctl start nginx
   38  sudo service start nginx
   39  sudor sevice nginx restart
   40  sudo service nginx restart
   41  sudo service httpd stop
   42  sudo service apache2 stop
   43  sudo bash
   44  sudo yum install -y docker
   45  sudo service docker start
   46  sudo usermod -a -G docker ec2-user
   47  sudo bash
   48  sudo bash
   49  history
[root@ip-172-31-95-223 ec2-user]# history
    1  lsof | grep 80
    2  lsof | grep 80 | more
    3  telnet 127.0.0.1
    4  docker ps
    5  docker stop
    6  7d1bcec3173a
    7  docker stop 7d1bcec3173a
    8  docker ps
    9  service nginx start
   10  curl 127.0.0.1
   11  vi /etc/nginx/nginx.conf
   12  service nginx restart
   13  curl 127.0.0.1
   14  vi /etc/nginx/nginx.conf
   15  curl 127.0.0.1
   16  service nginx restart
   17  curl 127.0.0.1
   18  vi /etc/nginx/nginx.conf
   19  wget 127.0.0.1
   20  service nginx restart
   21  wget 127.0.0.1
   22  vi /etc/hosts
   23  ping www.backofheadbook.com
   24  wget http://www.backofheadbook.com
   25  ls -altr /var/log/nginx
   26  tail /var/log/nginx/error.log
   27  tail /var/log/nginx/access.log
   28  tail /var/log/nginx/error.log
   29  curl 172.31.82.251
   30  curl 172.31.82.251/index.php
   31  curl 172.31.82.251/index.phpd
   32  ls
   33  ls
   34  pwd
   35  curl 172.31.92.179
   36  curl 172.31.92.179/login.php
   37  ping 172.31.92.179
   38  curl 172.31.92.179/login.phpd
   39  curl 172.31.92.179/login.php
   40  curl http://172.31.81.148/login.php
   41  curl http://172.31.92.179/login.php
   42  history
   43  pwd
   44  ls
   45  cd /etc/nginx
   46  ls
   47  vi nginx.conf
   48  service nginx restart
   49  curl 127.0.0.1
   50  curl 127.0.0.1/login.php
   51  vi nginx.conf
   52  service nginx restart
   53  vi nginx.conf
   54  service nginx restart
   55  ls
   56  ps
   57  ps -A
   58  cd /etc/nginx/
   59  less nginx.conf
   60  vi nginx.conf
   61  service nginx restart
   62  vi nginx.conf
   63  service nginx restart
   64  find / -name haproxy -print
   65  service nginx stop
   66  service haproxy restart
   67  cd /etc/haproxy/
   68  ls
   69  vi haproxy.cfg
   70  vi /etc/nginx/nginx.conf
   71  vi haproxy.cfg
   72  service haproxy restart
   73  curl 127.0.0.1
   74  vi haproxy.cfg
   75  service haproxy restart
   76  curl 127.0.0.1
   77  curl 127.0.0.1/index.xxx
   78  vi haproxy.cfg
   79  service haproxy restart
   80  cd /var/www/html
   81  ls
   82  cd /var/www
   83  ls
   84  cd /var
   85  ls
   86  cd http
   87  find / index.xxx -print
   88  find / -name index.xxx
   89  find / -name index.php
   90  find / -name index.php -print
   91  find / -name index.html -print
   92  cd /var/www
   93  psgrep apa
   94  ps -A | grep apa
   95  ps -A | grep ttp
   96  ps -A | grep pach
   97  history
   98  history | more
   99  history
[root@ip-172-31-95-223 ec2-user]#




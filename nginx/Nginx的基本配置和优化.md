Nginx的基本配置和优化
=====================

##示例

      #使用的用户和组
      #user  www www;
      
      #指定工作衍生进程数（一般为CPU核数*2）
      worker_processes  8;
      
      #指定文件描述符数量
      worker_rlimit_nofile 51200;
      #指定错误日志存放路径[]
      #error_log  /data/logs/error.log  crit;
      
      #指定pid存放路径
      pid        /user/local/nginx/logs/nginx.pid;
      
      events {
          #使用网络I/O模型，Linux使用epoll，FreeBSD使用kquenue
          use epoll;
          #允许的最大连接数
          worker_connections  51200;
      }
      
      http {
          include       mime.types;
          default_type  application/octet-stream;
          #默认编码
          #charset utf-8; 
      
          server_names_hash_bucket_size 128;
          clieng_header_buffer_size 32k;
          large_client_header_size 4 32k;
          
          #设置客户端能够上传的文件大小
          #client_max_body_size 8m;
          
          sendfile on;
          tcp_nopush    on;
          
          keepalive_timeout 60;
          tcp_nodelay on;
          
          fastcgi_connect_timeout 300;
          fastcgi_seng_timeout 300;
          fastcgi_read_timeout 300;
          fastcgi_buffer_size 64k;
          fastcgi_buffers 4 64k;
          fastcgi_busy_buffers_size 128k;
          fastcgi_temp_file_write_size 128k;
          #开启gzip
          gzip on;
          gzip_min_length 1k;
          gzip_buffers 4 16k;
          gzip_http_version 1.1;
          gzip_comp_level 2;
          gzip_types text/plan application/x-javascript text/css application/xml;
          gzip_vary on;
          #limit_zone crawler $binary_remote_addr 10m;
          
      
          #设定虚拟主机配置
          server {
              listen       80 ;
              server_name  localhost;
              index index.html index.htm index.php;
              root /data0/htdocs;
              
              #limit_conn crawler 20;
              
              location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
              {
                  expires     30d;
              }
              
              location ~ .*\.(js|css)$
              {
                  expires     1h;
              }
                    
          #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';

            } 
        }
        
##结构

      ......
      events
      {
      ......
      }
      
      http
      {
      ......
            server
            {
            ......
            }
            
                        server
            {
            ......
            }
      ......
      }

##基于IP的虚拟主机
      
      使用IP
            

##基于域名的虚拟主机

      使用DNS
      
##日者配置和切割
      
      配置：access_log
                  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';
##压缩
      gzip on;
      gzip_min_length 1k;
      gzip_buffers 4 16k;
      gzip_http_version 1.1;
      gzip_comp_level 2;
      gzip_types text/plan application/x-javascript text/css application/xml;
      gzip_vary on;
          
##自动列目录配置

      autoindex on;

##浏览器本地缓存

      location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
      {
            expires     30d;
      }
              
      location ~ .*\.(js|css)$
      {
            expires     1h;
      }
      

负载均衡和反向代理的配置和优化
==============================

##负载均衡

###常见方法
      1. 用户手动选择（例如华军软件园下载时的选择）
      2. DNS轮询方式
            缺点：
                  可靠性低
                  负载分配不均匀
                  
      3. 四/七层负载均衡设备
      4. 多线多地区智能DNS解析与混合负载均衡方式

##Nginx负载均衡和反向代理实例

      worker_processes  10;
      
      #设置最大打开文件数的限制
      worker_rlimit_nofile 65535;
      
      #error_log  x:/xxxxx/xxxx/log/error.log;
      #error_log  logs/error.log  notice;
      #error_log  logs/error.log  info;
      
      work_rlimit_nofile 51200;
      
      pid        logs/nginx.pid;
      
      #Nginx的事件模块，用于控制Nginx如何处理连接，参数会影响性能，慎重设置。
      events {
          user epoll;
          worker_connections  51200;
      }
      

      http {
          include       mime.types;
          #默认文件类型
          default_type  application/octet-stream;
          #默认编码
          #charset utf-8; 
          
          server_names_hash_bucket_size 128;
          client_header_buffer_size 32k;
          large_client_header_buffers 4 32k;
          
          sendfile on;
          
          keepalive_timeout 65;
          
          tcp_nodelay on;
          
          gzip  on;
          gzip_min_length 1k;
          gzip_buffers 4 16k;
          gzip_http_version 1.1;
          gzip_comp_level 2;
          gzip_types text/plain application/xml text/css application/x-javascript;
          gzip_vary on;
          
          #允许客户端请求的最大单个文件字节数
          client_max_body_size 300m;
          
          #缓冲区代理缓冲用户端请求的最大字节数
          client_body_buffer_size 128;
          
          #跟后端服务器连接的超时时间
          proxy_connect_timeout 600;
          
          #连接成功后等待后端服务器响应时间
          proxy_read_timeout 600;
          
          #后端服务器数据回传时间
          proxy_send_timeout 600;
          
          #代理请求缓存区
          proxy_buffer_size 16k;
          
          proxy_buffers 4 32k;
          
          proxy_busy_buffers_size 64k;
          
          proxy_temp_file_write_size 64k;
          
          upstream pool1{
          ......
          }
          
          upstream pool2{
          ......
          }
          
          upstream pool3{
          ......
          }
          
          #反向代理1
          server
          {
          .......
          }
          #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';
        }

##注意点
      * 真实ip使用 HTTP_X_FORWARDED_FOR

## Upstream模块

### ip_hash

      设置同一ip访问相同的服务器，不推荐使用
      
###server指令

      * 在upstream下使用
      * weight 权重
      * max_fails
      * fail_timeout
      * down
      * backup
      
###upstream指令

      * 在http下使用
      * $upstream_addr  处理请求的服务器地址
      * $upstream_status      服务器的应答状态
      * $upstream_response_time     服务器响应时间
      * $upstream_http_$HEADER      任意的HTTP协议头协议

##Nginx负载均衡服务器的双机可用

###第一种方式
      使用基于VRRP路由协议的keepalive的软件实现

###第二种方式
      DNS轮询

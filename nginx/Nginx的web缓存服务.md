Nginx的web缓存服务
==================

##nginx的proxy_cache模块

         所需软件：
         wget http://labs.frickle.com/files/ngx_cache_purge-1.3.tar.gz
         pcre,nginx-0.8.29.tar.gz
         # purge模块
         http://labs.frickle.com/nginx_ngx_cache_purge/
          
         安装
         tar -zxvf ngx_cache_purge-1.0.tar.gz
         tar -zxvf nginx-0.8.29.tar.gz
         cd nginx-0.8.29
         ./configure --user=www --group=www --add-module=../ngx_cache_purge-1.0 
         --prefix=/usr/local/gp/nginx --with-http_stub_status_module 
         --with-http_ssl_module
         make && make install
   
##配置

         1.创建缓存目录，路径同一分区下
           mkdir -p /data0/proxy_temp_path
           mkdir -p /data0/proxy_cache_path
           
         2.proxy_temp这个目录用于存储临时文件，默认在nginx下.需要看下是否www有权限写入，
           如果不可写，无法在这个目录生成文件的话，会导致反向代理失败。也可以在
           nginx的配置里设置proxy_temp_path指定存储临时文件的目录。
         3.proxy_cache_path
           /data0/proxy_cache_path levels=1:2 keys_zone=cache_one:200m
           inactive=1d max_size=500m;
           levels设置目录层次 
           keys_zone设置缓存空间名字和共享内存大小(热点内容放在内存) 
           inactive在指定时间内没人访问则被删除在这里是1天 
           max_size最大缓存空间
           
           所有active的key和元数据都保存在共享内存中-缓存区域(zone),
           在keys_zone中定义该缓存区域的名字和大小.

##nginx.conf 只写关键步骤

          # 日志格式设置命中状态有HIT,EXPIRED
           $upstream_cache_status
          # 据说先写到temp再移动到cache
          proxy_temp_path /data0/proxy_temp_path;
          proxy_cache_path /data0/proxy_cache_path levels=1:2 keys_zone=cache_one:200m
          inactive=10m max_size=500m;
      
          upstream my_server_pool {
            server 115.79.90.78:80;
          }
          
              server 
          {
              listen 80;
              server_name file.51yuncai.com;
              location /
              {
               proxy_set_header Host $host;
               proxy_set_header X-Forwarded-For $remote_addr;
               proxy_pass http://my_server_pool;
              }
         ### 特别注意 下面的purge功能部分一定不要放在下面,放在最下面的话purge怎么都报404#####
              location ~ /purge(/.*)
               {
                allow 127.0.0.1;
                allow 192.168.1.0/24;
                deny all;
                proxy_cache_purge cache_one $host$1$is_args$args;
               }
          # 设置缓存的文件类型,建议在后端设置缓存时间通过expires cache-control:max-age=[ses]
      
              location ~ .*\.(gif|jpg|png|bmp|swf|js|css)$
              {
               proxy_cache cache_one;
               
           # 在cache端强制设置缓存时间
               proxy_cache_valid 200 304 12h;
               proxy_cache_valid 301 302 1m;
               proxy_cache_valid any 1m;
      
               proxy_cache_key $host$uri$is_args$args;
      
               #####
               add_header X-Cache $upstream_cache_status;
               proxy_set_header X-Forwarded-For $remote_addr;
               proxy_pass http://my_server_pool;
              }
      
            }

Nginx的基本配置和优化
=====================
      
      #关掉访问日志可以优化服务器
      
      #定义Nginx运行的用户和用户组
      #user  nobody;
      
      #nginx进程数
      #如果NginX提供了SLL或者gzip，对cpu的使用率较高，且系统有两个以上的cpu
      #可设置为等于cpu总核心数
      worker_processes  1;
      
      #设置最大打开文件数的限制
      worker_rlimit_nofile 65535;
      
      #error_log  x:/xxxxx/xxxx/log/error.log;
      #error_log  logs/error.log  notice;
      #error_log  logs/error.log  info;
      
      #进程文件
      pid        logs/nginx.pid;
      
      #Nginx的事件模块，用于控制Nginx如何处理连接，参数会影响性能，慎重设置。
      events {
          #单个后台worker process进程的最大并发链接数
          #书本建议值为 65535 
          worker_connections  1024;
          #woker_connections 65535;
          # 并发总数是 worker_processes 和 worker_connections 的乘积
          # 即 max_clients = worker_processes * worker_connections
          # 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么
          # 为什么上面反向代理要除以4，应该说是一个经验值
          # 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000
          # worker_connections 值的设置跟物理内存大小有关
          # 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数
          # 而系统可以打开的最大文件数和内存大小成正比，一般1GB内存的机器上可以打开的文件数大约是10万左右
          # 我们来看看360M内存的VPS可以打开的文件句柄数是多少：
          # $ cat /proc/sys/fs/file-max
          # 输出 34336
          # 32000 < 34336，即并发连接总数小于系统可以打开的文件句柄总数，这样就在操作系统可以承受的范围之内
          # 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置
          # 使得并发总数小于操作系统可以打开的最大文件数目
          # 其实质也就是根据主机的物理CPU和内存进行配置
          # 当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。
          # ulimit -SHn 65535
      }
      
      
      
      
      #Nginx的HTTP内核模块，设定http服务器
      http {
          #文件扩展名与文件类型映射表
          include       mime.types;
          #默认文件类型
          default_type  application/octet-stream;
          #默认编码
          #charset utf-8; 
      
          #隐藏Nginx版本号
          #server_tokens off
      
         #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';
      
          #access_log  x:/xxxx/xxxx/log/access.log  main;
          #关闭日志
          #access_log off;
          #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件
          #对于普通应用设为 on，
          #如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。
          #注意：如果图片显示不正常把这个改成off。
          sendfile        on;
          #tcp_nopush     on;
      
          #keepalive_timeout  0;
          keepalive_timeout  60;
      
      
          #启用gzip压缩功能
          #gzip  on;
          #用于设置响应体的最小长度，单位为字节。
          #gzip_min_length 1000;
          #设置启用或禁用从代理服务器上收到的响应体Gzip压缩功能
          # expired---如果Expires header阻止缓存，那么启用压缩
          # auth -----假如设置了Authorization header，则启用压缩
          #gzip_proxied expired no-cache no-store private auth;
          #gzip_types text/plain application/xml;
          #gzip_disable "MSIE [1-6].";
      
          #用于指定客户端请求体的最大值。超过将收到413错误
          client_max_body_size       16m;
          #指定客户端请求体缓存的大小。
          #入股请求体大于该缓存大小，那么整个请求体或者请求体的某些部分会被写入临时文件
          #默认值 8k/16k
          #默认值等于两页面的大小，页面的大小依赖于所在的操作系统
          client_body_buffer_size    256k;
          proxy_temp_file_write_size 128k;
          proxy_temp_path  x:/xxx/xxxxx/public;
          #levels设置目录层次
          #keys_zone设置缓存名字和共享内存大小
          #inactive在指定时间内没人访问则被删除在这里是1天
          #max_size最大缓存空间
          #proxy_cache_path x:/xxxx/xxx/cache levels=1:2 keys_zone=cache_one:256m inactive=2d max_size=300m;
      
      
      
      
          upstream mywebapp_thin 
          {
          #指定服务器组的负载均衡方法，请求基于客户端的IP地址在服务器间进行分发。 
          #IPv4地址的前三个字节或者IPv6的整个地址，会被用来作为一个散列key。 
          #这种方法可以确保从同一个客户端过来的请求，会被传给同一台服务器。
          #除了当服务器被认为不可用的时候，这些客户端的请求会被传给其他服务器，
          #而且很有可能也是同一台服务器。
          #如果使用了该指明，那么将会导致客户端的请求以客户端的IP地址分配在upstream中的server之间。
          #它的关健技术在于对这个请求客户但ip地址进行哈希计算
          #这种方法保证了客户端请求总是能够传递到同一台后台服务器，但是如果该服务器被认定为无效，
          #那么这个客户端的请求将会被传递到其他服务器。
          #可以高概率将客户端请求总是连接到同一台服务器
          #如果使用这个指令，就不可以使用权重。
          ip_hash;
          #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。
          #weigth参数表示权值，权值越高被分配到的几率越大。
          #如果服务器要移除，在 ip后加个down
          #backup表示备用
          server 127.0.0.1:3000;
          server 127.0.0.1:3001 down;
          }
      
      
      
          #设定虚拟主机配置
          server {
              #监听端口
              listen       80 default_server;
              #定义使用 localhost访问
              #域名可以有多个，用空格隔开
              #将进入的HTTP请求的主机头（Host header）与Nginx配置文件中各个Server｛...｝区段比较
              #并且选择第一个被匹配的server区段——。
              #服务器名字按照一下顺序处理：
              #1.全域名，静态域名
              #2.开始部分使用通配符的域名，例如：*.exmple.com
              #3.结尾部分使用通配符的域名，例如：www.example.*
              #4.带有正则表达式的域名。
              server_name  localhost;
      
              #默认编码
              #charset koi8-r;
      
              #rewrite_log on;
      
              #设定本虚拟主机的访问日志
              #access_log  logs/host.access.log  main;
              #access_log  x:/xxx/xxx/log/host.access.log;
      
              root   x:/xxx/xxx/public;
      
      
      
              #图片缓存时间设置
              #location ~* ^/(images|javascripts|stylesheets|img)/ 
              #location ~* .(gif|jpg|jpeg|png|bmp|swf)$ 
              #{
              #    root   x:/xx/xxx/public;
              #    access_log    off;
              #    log_not_found off;
              #    expires       max;
      
                  #通过key来hash，定义KEY的值
              #    proxy_cache_key      $host$uri$is_args$args;
                  #根keys_zone后的内容对应
              #    proxy_cache         cache_one;
                  #哪些状态缓存多长时间
              #    proxy_cache_valid    200 302 7d;
                  #其他的缓存多长时间
              #    proxy_cache_valid    301 3d;
              #    proxy_cache_valid    any 1m;
      
              #   if (!-f $request_filename) {
              #    proxy_pass http://mywebapp_thin;
              #    break;
              #    } 
              #break;
              #}
      
              #允许根据URI的需要进行配置访问
              #可以跟进字面字符串配置也可以根据正则表达式配置
              #如果使用正则表达式配置，必须使用一下前缀
              # “~” 区分大小写匹配
              # “~*” 不区分大小写
              #先按照字面字符串检测，然后再正则表达式，正则表达式
              #默认请求,对 "/" 启用反向代理
              location / {
                  #定义了一个请求的根文档目录
                  root   x:/xxx/xxx/public;
                  #定义首页索引文件的名称
                  index  index.html index.htm;
                  if (!-f $request_filename) {
                  proxy_pass http://mywebapp_thin;
                   break;
                  } 
                  proxy_redirect off;
                  proxy_set_header X-Real-IP $remote_addr;
                  #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  #以下是一些反向代理的配置，可选。
                  proxy_set_header Host $host;
                  #允许客户端请求的最大单文件字节数
                  client_max_body_size 10m; 
                  #缓冲区代理缓冲用户端请求的最大字节数，
                  client_body_buffer_size 128k; 
                  #nginx跟后端服务器连接超时时间(代理连接超时)
                  proxy_connect_timeout 90; 
                  #后端服务器数据回传时间(代理发送超时)
                  proxy_send_timeout 90;
                  #连接成功后，后端服务器响应时间(代理接收超时) 
                  proxy_read_timeout 90; 
                  #设置代理服务器（nginx）保存用户头信息的缓冲区大小
                  proxy_buffer_size 4k; 
                  #proxy_buffers缓冲区，网页平均在32k以下的设置
                  proxy_buffers 4 32k;
                  #代理的时候，开启或关闭缓冲后端服务器的响应。 
                  proxy_buffering on;
                  #高负荷下缓冲大小（proxy_buffers*2）
                  proxy_busy_buffers_size 64k; 
                  #设定缓存文件夹大小，大于这个值，将从upstream服务器传
                  proxy_temp_file_write_size 64k;
      
                    #通过key来hash，定义KEY的值
              #    proxy_cache_key      $host$uri$is_args$args;
                  #根keys_zone后的内容对应
              #    proxy_cache         cache_one;
                  #哪些状态缓存多长时间
              #    proxy_cache_valid    200 302 7d;
                  #其他的缓存多长时间
              #    proxy_cache_valid    301 3d;
              #    proxy_cache_valid    any 1m;
              }
      
      
              #指定一个uri,在访问出错的时候会显示的页面
              #error_page  404              /404.html;
      
              # redirect server error pages to the static page /50x.html
              #
              # 定义错误提示页面
              error_page   500 502 503 504  /50x.html;
              location = /50x.html {
                  root   html;
              }
      
              # proxy the PHP scripts to Apache listening on 127.0.0.1:80
              #
              #location ~ .php$ {
              #    proxy_pass   http://127.0.0.1;
              #}
      
              # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
              #
              #location ~ .php$ {
              #    root           html;
              #    fastcgi_pass   127.0.0.1:9000;
              #    fastcgi_index  index.php;
              #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
              #    include        fastcgi_params;
              #}
      
              # deny access to .htaccess files, if Apache's document root
              # concurs with nginx's one
              #
              #location ~ /.ht {
              #    deny  all;
              #}
      
              #图片缓存时间设置
              #location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$
              #{
              #    expires 10d;
              #}  
              #JS和CSS缓存时间设置
              #location ~ .*.(js|css)?$
              #{
              #    expires 1h;
              #}
      
              #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。
              fastcgi_connect_timeout 300;
              fastcgi_send_timeout 300;
              fastcgi_read_timeout 300;
              fastcgi_buffer_size 64k;
              fastcgi_buffers 4 64k;
              fastcgi_busy_buffers_size 128k;
              fastcgi_temp_file_write_size 128k;
          }
      
      
          # another virtual host using mix of IP-, name-, and port-based configuration
          #
          #server {
          #    listen       8000;
          #    listen       somename:8080;
          #    server_name  somename  alias  another.alias;
      
          #    location / {
          #        root   html;
          #        index  index.html index.htm;
          #    }
          #}
      
      
          # HTTPS server
          #
          #server {
          #    listen       443 ssl;
          #    server_name  localhost;
      
          #    ssl_certificate      cert.pem;
          #    ssl_certificate_key  cert.key;
      
          #    ssl_session_cache    shared:SSL:1m;
          #    ssl_session_timeout  5m;
      
          #    ssl_ciphers  HIGH:!aNULL:!MD5;
          #    ssl_prefer_server_ciphers  on;
      
          #    location / {
          #        root   html;
          #        index  index.html index.htm;
          #    }
          #}
        }

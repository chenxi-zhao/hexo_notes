```
#定义Nginx运行的用户和用户组
user www www;

#Nginx进程数，建议设置为等于CPU总核心数。
#阻塞和非阻塞网络模型：
#同步阻塞模型，一请求一进（线）程，当进（线）程增加到一定程度后
#更多CPU时间浪费到切换一，性能急剧下降，所以负载率不高
#Nginx基于事件的非阻塞多路复用(epoll或kquene)模型
#一个进程在短时间内可以响应大量的请求
#建议值 <= cpu核心数量，一般高于cpu数量不会带好处，也许还有进程切换开销的负面影响
worker_processes 4;

#将work process绑定到特定cpu上，避免进程在cpu间切换的开销
worker_cpu_affinity 0001 0010 0100 1000
#8内核4进程时的设置方法 worker_cpu_affinity 00000001 00000010 00000100 10000000

# 每进程最大可打开文件描述符数量(linux上文件描述符比较广义，网络端口、设备、磁盘文件都是)
# 文件描述符用完了，新的连接会被拒绝，产生502类错误
# linux最大可打开文件数可通过ulimit -n FILECNT或 /etc/security/limits.conf配置
# 理论值 系统最大数量 / 进程数。但进程间工作量并不是平均分配的，所以可以设置的大一些
worker_rlimit_nofile 655350

#全局错误日志定义类型，多个等级可并存，[ debug | info | notice | warn | error | crit ]，从左到右错误信息越来越少；此指令可以在全局、http、server、location块中配置）
error_log /var/log/nginx/error.log notice;
error_log /var/log/nginx/error.log info;

#Nginx进程文件
pid /var/run/nginx.pid;

#工作模式与连接数上限
events{
    # 并发响应能力的关键配置值
    # 每个进程允许的最大同时连接数，work_connectins * worker_processes = maxConnection;
    # 要注意maxConnections不等同于可响应的用户数量，
    # 因为一般一个浏览器会同时开两条连接，如果反向代理，nginx到后端服务器的连接也要占用连接数
    # 所以，做静态服务器时，一般 maxClient = work_connectins * worker_processes / 2
    # 做反向代理服务器时 maxClient = work_connectins * worker_processes / 4

    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
    use epoll;
    #单个进程最大连接数（最大连接数=连接数*进程数）参照getconf PAGESIZE
    worker_connections 4096;

    # 备注：要达到超高负载下最好的网络响应能力，还有必要优化与网络相关的linux内核参数
}

#设定http服务器
http{
    include mime.types;
    default_type application/json; #默认文件类型

    #charset utf-8; #默认编码

    client_header_buffer_size 32k; #上传文件大小限制

    large_client_header_buffers 4 64k; #设定请求缓

    client_max_body_size 8m; #设定请求缓

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    sendfile on;

    #autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。

    #server_tokens off; #关闭服务器版本号显示。

    # 简单说，启动如下两项配置，会在数据包达到一定大小后再发送数据
    # 这样会减少网络通信次数，降低阻塞概率，但也会影响响应及时性
    # 比较适合于文件下载这类的大数据包通信场景
    tcp_nopush on; #防止网络阻塞
    tcp_nodelay on; #防止网络阻塞

    keepalive_timeout 65; #长连接超时时间，单位是秒

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5类似应用请使用1.0）
    gzip_comp_level 2; #压缩等级
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    #limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用


    # 静态文件缓存
    # 最大缓存数量，文件未使用存活期
    open_file_cache max=655350 inactive=20s;
    # 验证缓存有效期时间间隔
    open_file_cache_valid 30s;
    # 有效期内文件最少使用次数
    open_file_cache_min_uses 2;

    upstream test.com {
        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;
    }

    #虚拟主机的配置
    server{
        #监听端口
        listen 80;

        #域名可以有多个，用空格隔开
        server_name localhost;
        index index.html index.htm index.php;
        root /html;

        #location表达式：
        #syntax: location [=|~|~*|^~|@] /uri/ { … }
        #分为两种匹配模式，普通字符串匹配，正则匹配
        #无开头引导字符或以=开头表示普通字符串匹配
        #以~或~* 开头表示正则匹配，~*表示不区分大小写
        #多个location时匹配规则
        #总体是先普通后正则原则，只识别URI部分，例如请求为/test/1/abc.do?arg=xxx
        #1. 先查找是否有=开头的精确匹配，即location = /test/1/abc.do {...}
        #2. 再查找普通匹配，以 最大前缀 为规则，如有以下两个location
        #   location /test/ {...}
        #   location /test/1/ {...}
        #   则匹配后一项
        #3. 匹配到一个普通格式后，搜索并未结束，而是暂存当前结果，并继续再搜索正则模式
        #4. 在所有正则模式location中找到第一个匹配项后，以此匹配项为最终结果
        #   所以正则匹配项匹配规则受定义前后顺序影响，但普通匹配不会
        #5. 如果未找到正则匹配项，则以3中缓存的结果为最终结果
        #6. 如果一个匹配都没有，返回404

        #location =/ {...} 与 location / {...} 的差别
        #前一个是精确匹配，只响应/请求，所有/xxx类请求不会以前缀匹配形式匹配到它
        #而后一个正相反，所有请求必然都是以/开头，所以没有其它匹配结果时一定会执行到它

        #location ^~ / {...} ^~意思是非正则，表示匹配到此模式后不再继续正则搜索
        #所有如果这样配置，相当于关闭了正则匹配功能
        #因为一个请求在普通匹配规则下没得到其它普通匹配结果时，最终匹配到这里
        #而这个^~指令又相当于不允许正则，相当于匹配到此为止

        location ~ .*\.(php|php5)?$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
        #图片缓存时间设置
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)${
            expires 10d;
        }
        #JS和CSS缓存时间设置
        location ~ .*\.(js|css)?${
            expires 1h;
        }

        #日志格式设定
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';

        #定义本虚拟主机的访问日志
        access_log /var/log/nginx/ha97access.log access;

        #对 "/" 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;
            client_max_body_size 10m; #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
            proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
            proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file conf/htpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
        }

        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }

        #所有静态文件由nginx直接读取不经过tomcat或resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)${
            expires 15d;
        }
        location ~ .*.(js|css)?${
            expires 1h;
        }
        #nginx禁止访问所有.开头的隐藏文件设置
        location ~* /.* {
            deny all;
        }
        #定义错误提示页面
        error_page   500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

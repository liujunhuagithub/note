# 基本命令

1. 启动：nginx/sbin/nginx(配置文件为/nginx/conf/nginx.conf)  或  nginx/sbin/nginx -c  指定配置文件位置
2. 停止：nginx -s stop
3. 重载配置文件：nginx -s reload
4. conf中http 1——n server 1——n location     server是根据名字区别的vhost

# 配置项

## 全局块——全局系统参数

| 参数名       | 意义            | 备注                                             |
| :----------- | --------------- | ------------------------------------------------ |
| user         | 使用nginx的用户 |                                                  |
| work_process | 工作进程数      | 并发数=worker_process*worker_connections(CPU\*2) |
| error_log    | 日志路径        | error_log  logs/error.log  notice                |
| pid          | 进程pid文件路径 |                                                  |
|              |                 |                                                  |
|              |                 |                                                  |



##events块——nginx的配置

| 参数名             | 意义               | 备注 |
| ------------------ | ------------------ | ---- |
| worker_connections | 每个worker的连接数 |      |
|                    |                    |      |
|                    |                    |      |
|                    |                    |      |
|                    |                    |      |
|                    |                    |      |



##HTTP块——功能配置

###HTTP全局块

| 参数名            | 意义                  | 备注                                                         |
| ----------------- | --------------------- | ------------------------------------------------------------ |
| include           | 支持的application类型 | 保存于conf/mime.types                                        |
| default_type      | 不支持时的默认格式    | application/octet-stream                                     |
| log_format        | 日志格式              | log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' |
| access_log        | 访问日志路径和级别    | access_log  logs/access.log  main;                           |
| sendfile          | 开启高效文件输出      |                                                              |
| tcp_nopush        | 防止网络阻塞          |                                                              |
| keepalive_timeout | 超时时间              |                                                              |
| gzip              | 开启gzip压缩          |                                                              |

###upstream负载均衡块

| 参数名          | 意义                          | 备注                                                 |
| --------------- | ----------------------------- | ---------------------------------------------------- |
| upstream   name | 配置某集群，名为name          | 一般在server块的proxy_pass http://name访问           |
| server ...      | 此集群各节点IP：port          | 后缀backup:后备,全宕机才启动；down：此节点不处理请求 |
| weight          | 权重模式负载均衡，大则优先    | 在server节点后，server ip:port weight=*              |
| ip_hash         | ip哈希策略                    |                                                      |
| least_conn      | 最少连接数策略                | 哪个节点conn数最少则优先                             |
| fair            | 根据后端响应时间，短的优先。  |                                                      |
| url_hash        | 按访问url的hash结果来分配请求 |                                                      |
| 默认轮询        |                               |                                                      |

### Server块

| 参数名      | 意义         | 备注                                       |
| ----------- | ------------ | ------------------------------------------ |
| listen      | 监听端口     |                                            |
| server_name | 主机名       | 一般是同一IP的不同的域名，可多个(空格分开) |
| charset     | 字符集       | 默认UTF-8                                  |
| access_log  | 访问日志路径 |                                            |
|             |              |                                            |
|             |              |                                            |

1. server是根据名字区别的vhost，server_name一般与域名联系。仅一个server可忽略server_name
2. server_name匹配：
   - 全精确匹配：全名，多个时空格分开  server_name  domain.com  www.domain.com;
   - *通配符匹配：server_name  \*.domain.com  www.domain.\*;   
   - ~开头正则匹配  server_name  ~^(?.+)\.domain\.com$;
   - 优先级 ：==全精确     前置通配符*        后置通配符*           正则            第一个==
3. 
4. 6
5. 

#### Location

| 参数名                | 意义                            | 备注           |
| --------------------- | ------------------------------- | -------------- |
| location 匹配规则     | 匹配访问路径                    | =  ^~ ~ 无 /   |
| root                  | 映射后root根目录                |                |
| index                 | index文件相对于新的root目录位置 |                |
| proxy_pass            | 反向代理的URL                   | http://IP:port |
| peoxy_connect_timeout | 超时时间                        |                |
|                       |                                 |                |
|                       |                                 |                |

1. root目录为映射后的/，==新目录必须包含被访问的目录==    /abc => /root/abc  (root下必须有abc目录)；index配置的页面相对于root新目录
2. location路径匹配规则：
   - = patt	精确匹配
   - ^~patt   非URL编码的前缀匹配(还原路径的url编码如%20)
   - ~  patt     区分大小写正则
   - ~*  patt     不区分大小写正则
   - 无修饰符    前缀匹配
   - /    默认通用匹配   最长优先
   - 优先级：===     ^~    \~或\~*(按声明顺序)      无修饰的普通前缀匹配        /通用匹配==
3. location  /  通用匹配往往配置index和error页面用来兜底
4. 

# 反向代理

```
server {
	listen 80;
	server_name localhost;
	location / {
		proxy_passhttp: http://127.0.0.1:8080;
		index index .html index.htm ;
	}
}
```

#动静分离和负载均衡

==**宕机节点自动排除，上线后------------------------**==

```
server {
	listen 80;
	server_name localhost;
	
	upstream myserver{
		# ip_hash;   least_conn;
		server 115.28.52.63:8080 weight=1;
		server 115.28.52.63:8180 weight=1;
	}

	location ~ /resources/.*$ {
		root /root/*;        最终是/root/resources
		index index .html index.htm ;
	}
	location / {
		proxy_passhttp: http://myserver;
		proxy_connect_timeout 10;
	}

}
```



参考:[Nginx 从入门到实践，万字详解！_SHERlocked_93的博客-NGINX开源社区](https://www.nginx.org.cn/article/detail/545)




# 解决跨域问题

## 正向代理

参考：https://www.cnblogs.com/yanjieli/p/15229907.html

正向代理代理的实质是nginx捕获客户端请求，然后进行代理，
![[Pasted image 20241018093010.png]]



对应到配置文件中，就是nginx如何获取内网机器的请求参数（请求ip、请求路径等），内置变量存放在  ngx_http_core_module 模块中。
参考：https://www.cnblogs.com/larry-luo/p/10119842.html
```
server {
    listen                           8080;
    server_name                      localhost;
    resolver                         114.114.114.114 ipv6=off;
    proxy_connect;
    proxy_connect_allow              443 80;
    proxy_connect_connect_timeout    10s;
    proxy_connect_read_timeout       10s;
    #proxy_coneect_send_timeout       10s;
    location / {
        proxy_pass $scheme://$http_host$request_uri;
    }
}

```


## 反向代理
解决浏览器跨域问题的实质：nginx通过反向代理，将前端的同域请求代理到跨域请求。

```
server {
  listen 9001;
  server_name fe.sherlocked93.club;

  location / {
    proxy_pass be.sherlocked93.club;
  }
}

```


## header 



```
location / {
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
    
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
        return 204;
    }
}

```

参考： https://blog.redis.com.cn/doc/


格式：  add_header name value

- **`Access-Control-Allow-Origin`**：指定哪些源可以访问资源。在示例中，我们使用通配符`*`表示允许所有源访问，但在生产环境中，建议设置具体的源。
    
- **`Access-Control-Allow-Methods`**：指定允许的HTTP方法。在示例中，我们允许GET、POST和OPTIONS方法。
    
- **`Access-Control-Allow-Headers`**：指定允许的HTTP头部。在示例中，我们列出了一些常见的头部，如User-Agent、Content-Type等




## nginx鉴权认证: auth_request模块


参考：
[Nginx的auth_request 模块详解与应用指南_nginx auth-CSDN博客](https://blog.csdn.net/u010260632/article/details/139173856)
[Nginx反向代理服务的配置 - 简书 (jianshu.com)](https://www.jianshu.com/p/f0a84484d225)
[ngx_http_auth_request_module 模块 - Nginx 中文 (nginxserver.cn)](https://nginxserver.cn/en/docs/http/ngx_http_auth_request_module.html)



是Nginx的一个官方模块，用于在处理客户端请求时，将这些请求传递给一个外部的认证服务进行认证，要求进入该模块的请求都会发送一个子请求 `/auth`，并等待其响应结果

 `auth_request`的工作流程是这样的：

1. 客户端发起请求到Nginx。
2. Nginx遇到配置有`auth_request`指令的location块，会先向鉴权服务发送一个子请求。
3. 鉴权服务根据请求头中的信息（如Token）判断请求是否合法，并返回响应码。
4. 如果鉴权服务返回2xx状态码，表示鉴权通过，Nginx将继续处理该请求，比如执行后续的`proxy_pass`指令将请求转发到后端服务器。
5. 若鉴权服务返回非2xx状态码，Nginx将根据`auth_request`的配置决定如何响应客户端，通常会返回一个错误页面或特定的状态码，而不会继续执行其他模块中的处理逻辑。

注意：鉴权模块可以看做在匹配到某一模块时跳转到另外一模块进行验证，此处就是如果匹配到`/api/`模块，就会去发送一个子请求`/auth`，然后被nginx的`=/auth`location模块精确匹配

```
	 location /api/ {
		 # 鉴权模块，
        auth_request /auth;
        # 鉴权通过后的处理方式
        proxy_pass http://127.0.0.1:8081/success;
    }
	
	  location = /auth {
        internal;
        # 设置参数query为''
        set $query '';
        if ($request_uri ~* "[^\?]+\?(.*)$") {
            set $query $1;
        }
        # 验证成功，返回200 OK
        proxy_pass http://127.0.0.1:8081/verify?$query;
        proxy_pass_request_body off;
        proxy_set_header Content-Type "";
     }
```

`$request_uri ~* "[^\?]+\?(.*)$"`
这是一个正则表达式匹配语句，它检查请求URI是否包含查询字符串（即URL中的?后面的部分）。

`internal;`
表示该模块仅内部调用

`proxy_pass_request_body off;`
是否发送客户端请求的请求体（指在HTTP请求中由客户端发送给服务器的数据部分）
` proxy_set_header Content-Type "";` 清空请求头设置


## 请求匹配规则

[nginx 处理请求的方式 - Nginx 中文 (nginxserver.cn)](https://nginxserver.cn/en/docs/http/request_processing.html)

1，nginx 首先搜索由字面字符串给出的最具体的前缀位置，而不管列出的顺序，

这里的意思是除去正则表达式的前缀，且注意由于'/'匹配任何请求，一般将其称为通配location模块，因此，作为最后手段使用。因此第一步骤会简单根据具体的前缀去匹配，如果能够找到和请求url一样前缀，就直接执行，如果找不到，该步骤结束了

*所有类型的 location 仅测试请求行中不带参数的 URI 部分*
2，然后 nginx 按配置文件中列出的顺序检查由正则表达式给出的位置，检索到第一个匹配的表达式停止，nginx 将使用此位置
3，如果没有正则表达式与请求匹配，则 nginx 使用前面找到的最具体的前缀位置。


匹配优先级：精确字面字符串>第一个正则表达式>正则表达式>前缀匹配>`/` 通配location模块



## location模块匹配模式
该分类仅仅表明该location能够使用什么匹配模式，和请求的匹配规则并不冲突

[深入解析Nginx Location匹配规则：顺序详解与最佳实践 - 我和你并没有不同 - 博客园 (cnblogs.com)](https://www.cnblogs.com/testzcy/p/18217337)



# 配置记录

## include
引入其他nginx.conf配置文件，将Nginx的server拆分为多个文件。

例如：默认的80监听server放在sites-enabled/default文件中。主配置文件默认什么都没有
```
include /etc/nginx/conf.d/*.conf; 
include /etc/nginx/sites-enabled/*;
```



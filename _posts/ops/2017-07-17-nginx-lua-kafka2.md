---
layout: post
title: nginx之实时打印日志到kafka（二）
category: 运维
keywords: linux，nginx，lua，kafka
---

## 编译安装nginx

涉及的包软件及其下载地址：

[nginx-1.10.3](http://nginx.org/)

[LuaJIT](http://luajit.org/download.html)

[lua-nginx-module](https://github.com/openresty/lua-nginx-module)

[ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)

[lua-resty-kafka](https://github.com/doujiang24/lua-resty-kafka)

[lua-cjson](https://github.com/mpx/lua-cjson)

#### 安装LuaJIT
```shell
cd LuaJIT
make PREFIX=/usr/local/LuaJIT
make install PREFIX=/usr/local/LuaJIT
echo "/usr/local/LuaJIT/lib" > /etc/ld.so.conf.d/usr_local_luajit_lib.conf
ldconfig
#注意环境变量!
export LUAJIT_LIB=/usr/local/LuaJIT/lib
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.0
```
把export的内容追加到 /etc/profile 也可以

#### 解压lua-nginx-module

#### 解压ngx_devel_kit

#### 安装nginx

编译内容：
```shell
--prefix=/usr/local/nginx/ --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --add-module=/tmp/ngx_log_if-master --add-module=/usr/local/lua-nginx-module --add-module=/usr/local/ngx_devel_kit --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' --with-ld-opt=' -Wl,-E' --with-pcre=/tmp/pcre-8.40
```

导入lua-cjson模块
```
cd lua-cjson
make
make install
```
*注意*：这里很容易报错，原因就是找不到正确的luajit路径，解决方法：`vim Makefile`,更改`PREFIX`及其下面的路径即可。

到此安装基本完成，剩下的就是写nginx的配置文件。提供一下我个人测试用的，如下：
```
lua_package_path "/usr/local/nginx/lualib/kafka/?.lua;;";
    lua_shared_dict   config 1m;

    include /etc/nginx/conf.d/*.conf;

    server {
	listen       80;
	server_name  192.168.20.66;

	access_log  /var/log/nginx/edu.wps.cn.access.log  main;
        location /lua {
                default_type text/plain;
                content_by_lua 'ngx.say("hello world")';
		log_by_lua '
		local cjson = require "cjson"  
		local client = require "resty.kafka.client"
                local producer = require "resty.kafka.producer"
		local broker_list = {  
                { host = "kafka-ip", port = 9092 },
                }
		local log_json = {}  
                log_json["uri"]=ngx.var.uri  
                log_json["args"]=ngx.var.args  
                log_json["host"]=ngx.var.host  
                log_json["request_body"]=ngx.var.request_body  
                log_json["remote_addr"] = ngx.var.remote_addr  
                log_json["remote_user"] = ngx.var.remote_user  
                log_json["time_local"] = ngx.var.time_local  
                log_json["status"] = ngx.var.status  
                log_json["body_bytes_sent"] = ngx.var.body_bytes_sent  
                log_json["http_referer"] = ngx.var.http_referer  
                log_json["http_user_agent"] = ngx.var.http_user_agent  
                log_json["http_x_forwarded_for"] = ngx.var.http_x_forwarded_for  
                log_json["upstream_response_time"] = ngx.var.upstream_response_time  
                log_json["request_time"] = ngx.var.request_time
		local message = cjson.encode(log_json);
		local bp = producer:new(broker_list, { producer_type = "async" })
		local ok, err = bp:send("test1", nil, message)

		if not ok then  
                	ngx.log(ngx.ERR, "kafka send err:", err)  
                	return  
            	end  
        	';
        }


    }
```
还有，nginx.conf的error_log 开到info级别，这样lua模块报的错误都可以在error.log中看到，对解决问题很有帮助。

####  lua-resty-kafka
`cp -rf /tmp/lua-resty-kafka-master/lib/resty /usr/local/nginx/lualib/kafka/`

这样就会向kafka里面打进去一个json字符串。

在kafka那里起一个consumer接收如下：
![result-kafka](/assets/img/kafka01.png)

## 总结

因为之前没用过kafka，对于nginx嵌入lua的应用也比较少，此次遇到了很多的坑。差不多用了三天 才把这套系统弄完。总之，相信自己，不懂的就去学，不会的就去查，在一次次填坑中进步是最快的。

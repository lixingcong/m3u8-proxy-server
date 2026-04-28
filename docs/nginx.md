# 使用Nginx反代m3u8监听的服务器

```
server {
	server_name my.domain.com;
	listen 80;
	location /proxy {
        proxy_pass http://127.0.0.1:18080;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```

如果监听的是unix socket，可以替换proxy_pass这一行。

```
proxy_pass http://unix:/tmp/app.sock;
```


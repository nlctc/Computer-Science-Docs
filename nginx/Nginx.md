# Nginx

## 正向代理和反向代理

### 1.1 正向代理

代替客户端向服务端发送请求，服务端不知道具体的客户是谁。

### 1.2 反向代理

代替服务器响应服务端的请求，客户端不知道请求具体是在哪台服务器上处理的。



## 2. 配置文件

nginx.conf

```
http {
		include mime.types; # 包含了常见的文件类型

		types { 						# 只会加载指定了的文件类型
				text/css		css;
				text/html		html;
		}

		server {
				listen 8080;
				root /xxx/mysite; # 默认页
				
				location /fruits {  # 指定根路劲下的路径
						root /xxx/mysite;
				}
				
				location /carbs {
						root /xxx/mysite2;
						alias /xxx/mysite/fruits;
				}
				
				location /vegetables {
						root /xxx/mysite;
						try_files /vegetables/veggies.html /index.html =404;  # 没有指定时，默认找index.html，找不到报就报404
				}
				
				location ~* /count/[0-9] {
						root /xxx/mysite;
						try_files /index.html =404;
				}
				
				location /crops {
						return 307 /fruits;
				}
				
				rewrite ~/number/(\w+) /count/$1; # 重写为新的请求路径
		}
}
 
events {

}
```





## 3. 常用命令

```shell
./nginx # 启动

./nginx -s stop # 停止

./nginx -s quit # 安全退出

./nginx -s reload # 重新加载配置文件
# 注意用到的配置文件是 nginx 执行文件所在的那个路径下的 nginx.conf

ps aux | grep nginx # 查看nginx进程
```





Dockerfile

```
FROM node:18

WORKDIR /app

COPY server.js .
COPY index.html .
COPY package.json .

RUN npm install

EXPOSE 3000

CMD ["node", "server.js"]
```



docker-compose.yaml

```
version: '3'
services:
  app1:
    build: .
    environment:
      - APP_NAME=app1
    ports:
      - "3001:3001"

  app2:
    build: .
    environment:
      - APP_NAME=app2
    ports:
      - "3002:3002"

  app3:
    build: .
    environment:
      - APP_NAME=app3
    ports:
      - "3003:3003"

```

```shell
docker-compose up --build -d
```





## 4. HTTPS

```
worker_processes 1;
		
events {
	worker_connections 1024;
}
		
http {
	include mime.types;

	upstream nodejs_cluster {
		least_conn;
		server 127.0.0.1:3001;
		server 127.0.0.1:3002;
		server 127.0.0.1:3003;
	}
				
	server {
		listen 8080;
		server_name localhost;
						
		location / {
			proxy_pass http://nodejs_cluster;
			proxy_set_header Host $host;
			proxy_set_header X-Real_IP $remote_addr;
		}
	}
}

```



OpenSSL 生成 CA 证书

```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```

CA证书 nginx-selfsigned.key

秘钥 nginx-selfsigned.crt

```
worker_processes 1;
		
events {
	worker_connections 1024;
}
		
http {
	include mime.types;

	upstream nodejs_cluster {
		least_conn;
		server 127.0.0.1:3001;
		server 127.0.0.1:3002;
		server 127.0.0.1:3003;
	}
				
	server {
		listen 443 ssl;
		server_name localhost;

		ssl_certificate /Users/allen/Documents/app/computer-science-docs/nginx/nginx-demo/nginx-selfsigned.crt;
		ssl_certificate_key /Users/allen/Documents/app/computer-science-docs/nginx/nginx-demo/nginx-selfsigned.key;
						
		location / {
			proxy_pass http://nodejs_cluster;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
		}
	}

	server {
		listen 8088;
		server_name localhost;
		location / {
			return 301 https://$host$request_uri;
		}
	}
}
```


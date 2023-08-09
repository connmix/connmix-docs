# 生产部署

线上生产部署时，支持多种部署方式，包括：单机、集群、k8s集群。

## 标准部署

当采用 aliyun ecs, aws ec2 等服务器直接部署。

### 单机部署

创业公司业务量不大时，可以将 `center` 和 `engine` 节点在同一个进程启动，等后续业务扩展再采用集群部署。

```shell
% bin/connmix single -f conf/connmix.yaml
```

### 集群部署

> 免费的8cpu授权，建议给center节点配置2cpu，engine节点配置6cpu

#### 部署 `center` 节点

- 通过 `-p` 来配置使用的 `cpus` 数量，超过许可证数量将导致 `engine` 节点无法启动。

```shell
% bin/connmix center -f conf/connmix.yaml -p 2

 _________________________________________________________________________________________________         
  ______________________________________________________________________________/\\\_______________      
   _____/\\\\\\\\_____/\\\\\_____/\\/\\\\\\____/\\/\\\\\\______/\\\\\__/\\\\\___\///___/\\\____/\\\_     
    ___/\\\//////____/\\\///\\\__\/\\\////\\\__\/\\\////\\\___/\\\///\\\\\///\\\__/\\\_\///\\\/\\\/__    
     __/\\\__________/\\\__\//\\\_\/\\\__\//\\\_\/\\\__\//\\\_\/\\\_\//\\\__\/\\\_\/\\\___\///\\\/____   
      _\//\\\________\//\\\__/\\\__\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\____/\\\/\\\___  
       __\///\\\\\\\\__\///\\\\\/___\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\__/\\\/\///\\\_ 
        ____\////////_____\/////_____\///____\///__\///____\///__\///___\///___\///__\///__\///____\///__
        
        connmix1.0.1, go1.17.5, lua5.1+bit64, darwin, arm64

2022-04-22 01:31:51.686326      INFO    commands/licenses.go:143        licenses request successful, activation_code: JJGkbZ4tJvjSOmWCfKtxdqQUYS7ijJWH, cpus: 8
2022-04-22 01:31:51.686425      WARN    commands/welcome.go:30  cpu underutilized, max_procs: 2, device_cpus: 8
2022-04-22 01:31:51.686474      INFO    center/server.go:46     start the center server 0.0.0.0:6787
2022-04-22 01:31:51.686621      INFO    registry/server.go:42   start the registry server (0.0.0.0:6786)
```

#### 部署 `engine` 节点

- 需要先启动 `center` 节点，才能成功启动 `engine` 节点。
- 你可以启动任意个 `engine` 节点，只要在你的许可证数量之内。
- 所有的 `engine` 节点的 `lua server` 端口是业务端口，你需要加入到 nginx 或 SLB 中对外提供服务。

```shell
% bin/connmix engine -f conf/connmix.yaml -p 2

 _________________________________________________________________________________________________         
  ______________________________________________________________________________/\\\_______________      
   _____/\\\\\\\\_____/\\\\\_____/\\/\\\\\\____/\\/\\\\\\______/\\\\\__/\\\\\___\///___/\\\____/\\\_     
    ___/\\\//////____/\\\///\\\__\/\\\////\\\__\/\\\////\\\___/\\\///\\\\\///\\\__/\\\_\///\\\/\\\/__    
     __/\\\__________/\\\__\//\\\_\/\\\__\//\\\_\/\\\__\//\\\_\/\\\_\//\\\__\/\\\_\/\\\___\///\\\/____   
      _\//\\\________\//\\\__/\\\__\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\____/\\\/\\\___  
       __\///\\\\\\\\__\///\\\\\/___\/\\\___\/\\\_\/\\\___\/\\\_\/\\\__\/\\\__\/\\\_\/\\\__/\\\/\///\\\_ 
        ____\////////_____\/////_____\///____\///__\///____\///__\///___\///___\///__\///__\///____\///__
        
        connmix1.0.1, go1.17.5, lua5.1+bit64, darwin, arm64

2022-04-22 01:32:00.262744      WARN    commands/welcome.go:30  cpu underutilized, max_procs: 2, device_cpus: 8
2022-04-22 01:32:00.263168      INFO    mesh/server.go:41       start the mesh point (0.0.0.0:6788)
2022-04-22 01:32:00.263237      INFO    ws/server.go:152        start the api server (0.0.0.0:6789)
2022-04-22 01:32:00.263628      INFO    lua/registrycli.go:36   center registry 127.0.0.1:6786 connect successful
2022-04-22 01:32:00.263661      INFO    lua/registrycli.go:76   register engine node_id c9gpa43in567ju39lab0
2022-04-22 01:32:00.264456      INFO    lua/servers.go:96       start the lua server /Users/liujian/Documents/mycode/connmix/lua/entry.lua (0.0.0.0:6790)
```

### 配置反向代理 `nginx` 或 `SLB`

使用 `nginx` 或者 `SLB` 代理到所有 `engine` 节点的端口即可。

- ws

```
server{
    listen 80;
    server_name www.demo.com;
    location / {
        proxy_pass http://127.0.0.1:6790;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

- wss

`ssl` 需要在 `nginx`、`SLB` 中去实现。

```
server{
    listen 443 ssl;
    server_name www.demo.com;
    ssl_certificate /opt/nginx/ssl/demo.crt;
    ssl_certificate_key /opt/nginx/ssl/demo.key; 
    location / {
        proxy_pass http://127.0.0.1:6790;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

## 在 `k8s` 中部署

在k8s中部署我们需要部署两个服务：

- `connmix-center` 服务：在 `Dockerfile` 中暴露6786、6787端口，只可以启动1个pod。
- `connmix-engine` 服务：在 `Dockerfile` 中暴露6788、6789端口，加上配置文件 `servers` 字段下面全部的端口，同时将 `centerRegistry` 字段修改为 `connmix-center` 服务的k8s内网地址，可启动任意pod数量。
- 在 `ingress-nginx` 或者你使用的其他微服务网关中，将 `connmix-engine` 服务名称、加上配置文件 `servers` 字段下你配置的业务端口和对应的url path配置到 `ingress.yaml` 中的 `rules` 字段里。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: www.test.com                  #域名
    http:
      paths:
      - path: /chat
        backend:
          serviceName: connmix-engine   #关联service
          servicePort: 6790             #关联service的映射端口
```

## 常见问题

### 客户端连接 `60s` 后就断线

这是由 `connmix.yaml` 的 `readTimeout` 配置项控制的，该配置项是为了节约服务器资源，`nginx`、`SLB` 也有类似的配置项，因此在生产环境你修改了此配置后可能依旧还会遇到该问题，正确的做法应该在客户端做ping/pong心跳处理。

```yaml
  servers:
    - port: 6790
      bind: 0.0.0.0
      protocol: websocket    # (websocket,socket)
      options:
        - name: path
          value: /chat
      entry: lua/entry.websocket.lua
      byteCodeExpires: 10    # 设置0关闭热更新
      readBufferSize: 1024   # 修改后对新连接生效
      writeBufferSize: 1024  # 修改后对新连接生效
      readTimeout: 60        # 修改后对新连接生效
      writeTimeout: 10       # 修改后对新连接生效
```

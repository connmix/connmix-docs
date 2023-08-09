# Production Deployment

When deploying online, multiple deployment methods are supported, including: single machine, cluster, and k8s cluster.

## Standard Deployment

For direct deployment on servers like Aliyun ECS or AWS EC2.

### Single Machine Deployment

When the business volume of a startup company is not large, the `center` and `engine` nodes can be started in the same process. Cluster deployment can be adopted when the business expands later.

```shell
% bin/connmix single -f conf/connmix.yaml
```

### Cluster Deployment

> For a free 8-CPU license, it is recommended to allocate 2 CPUs for the `center` node and 6 CPUs for the `engine` node.

#### Deploy `center` Node

- Use `-p` to configure the number of `cpus` used. Exceeding the license quantity will result in the inability to start the `engine` node.

```shell
% bin/connmix center -f conf/connmix.yaml -p 2
```

#### Deploy `engine` Node

- The `center` node must be started first to successfully start the `engine` node.
- You can start any number of `engine` nodes as long as they are within your license quantity.
- The `lua server` port of all `engine` nodes is the business port. You need to add it to Nginx or SLB to provide external services.

```shell
% bin/connmix engine -f conf/connmix.yaml -p 2
```

### Configure Reverse Proxy `nginx` or `SLB`

Use `nginx` or `SLB` to proxy to all `engine` node ports.

- ws

```nginx
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

`ssl` needs to be implemented in `nginx`, `SLB`.

```nginx
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

## Deployment in `k8s`

In k8s, we need to deploy two services:

- `connmix-center` service: Expose ports 6786 and 6787 in the `Dockerfile`. Only one pod can be started.
- `connmix-engine` service: Expose ports 6788 and 6789 in the `Dockerfile`, along with all ports under the `servers` field in the configuration file. Modify the `centerRegistry` field to the k8s intranet address of the `connmix-center` service, and you can start any number of pods.
- In `ingress-nginx` or other microservice gateways you are using, configure the `connmix-engine` service name, business ports in the `servers` field of the configuration file, and corresponding URL paths into the `rules` field of `ingress.yaml`.

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
  - host: www.test.com                  # Domain
    http:
      paths:
      - path: /chat
        backend:
          serviceName: connmix-engine   # Related service
          servicePort: 6790             # Related service mapping port
```

## Common Issues

### Client connection drops after `60s`

This is controlled by the `readTimeout` configuration item in `connmix.yaml`. `nginx`, `SLB` also have similar configuration items, so even if you modify this configuration in the production environment, you may still encounter this issue. The correct approach is to perform a ping/pong heartbeat on the client-side.

```yaml
  servers:
    - port: 6790
      bind: 0.0.0.0
      protocol: websocket    # (websocket,socket)
      options:
        - name: path
          value: /chat
      entry: lua/entry.websocket.lua
      byteCodeExpires: 10    # Set to 0 to disable hot update
      readBufferSize: 1024   # Takes effect for new connections after modification
      writeBufferSize: 1024  # Takes effect for new connections after modification
      readTimeout: 60        # Takes effect for new connections after modification
      writeTimeout: 10       # Takes effect for new connections after modification
```

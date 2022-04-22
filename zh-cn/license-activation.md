# 激活授权

线上部署需要先在 `https://connmix.com` 获的许可证授权。

1. 注册账号
2. 创建应用
3. 免费获取激活码
4. 将激活码输入到 `conf/connmix.yaml` 配置文件中
5. 启动引擎就会激活许可证，并且绑定`IP`，所以请不要在开发环境(动态IP)中激活。

```yaml
licenses:
  activation_code: ***
  server: https://connmix.com
```

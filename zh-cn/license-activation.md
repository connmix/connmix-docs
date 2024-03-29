## 激活授权

线上部署需要先在 `https://connmix.com` 获的许可证授权。

1. 注册账号
2. 获取激活码（可免费获取）
3. 将激活码输入到 `conf/connmix.yaml` 配置文件中
4. 启动网关就会激活许可证，并且绑定IP等设备信息，所以免费版请不要在开发环境(动态IP)中激活；**专业版不限制IP只需确保同一时刻只有1个设备即可**；

```yaml
licenses:
  activationCode: ***             # 不支持热更新
  server: https://connmix.com     # 支持热更新
```

## 离线授权

> 开发中

适用于：

- 司法、国防等不能与互联网连通的涉密网络环境
- 数据安全要求极高，有数据隔离需求的客户

## 授权高可用

- 授权成功后，许可证授权信息会保存在本地，通常有效期为 `90` 天，因此即便 `https://connmix.com` 被攻击依然不会影响到服务的执行。
- 即便攻击超过 `90` 天，我们也会提供备用站点激活。

## 错误码

当收到 `license checkout failed Error-%d` 这样的错误信息时，请查看以下原因：

| 错误码        | 描述             |
|------------|----------------|
| 1          | 解密失败           |
| 2          | 参数异常           |
| 3          | 设备时间异常         |
| 4          | 授权已过期，请续期      |
| 5          | 授权码异常          |
| 6,7,8,9,10 | 设备信息和授权不匹配     |

## Activate License

To deploy online, you need to obtain a license authorization from `https://connmix.com`.

1. Register an account
2. Obtain an activation code (available for free)
3. Enter the activation code in the `conf/connmix.yaml` configuration file
4. Upon starting the gateway, the license will be activated and the IP and device information will be bound. Therefore, do not activate the free version in the development environment (with dynamic IP); **For the professional version, just make sure that there is only 1 device at the same time**;

```yaml
licenses:
  activationCode: ***             # Hot update not supported
  server: https://connmix.com     # Hot update supported
```

## Offline Authorization

> In development

Applicable to:

- Confidential network environments that cannot be connected to the Internet, such as judicial and defense
- Customers with high data security requirements and data isolation demands

## High Availability of Authorization

- After successful authorization, the license authorization information will be saved locally, usually valid for `90` days. Therefore, even if `https://connmix.com` is attacked, it will not affect the execution of the service.
- If the attack lasts for more than `90` days, we will also provide a backup site for activation.

## Error Codes

When you receive an error message such as "license checkout failed Error-%d", please check the following reasons:

| Error Code | Description                     |
|------------|---------------------------------|
| 1          | Decryption failed               |
| 2          | Parameter exception             |
| 3          | Device time exception           |
| 4          | License expired, please renew   |
| 5          | Activation code exception       |
| 6,7,8,9,10 | Device information does not match the authorization   |

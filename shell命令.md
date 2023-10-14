### Shell 常用命令
### 1，awk 相关


### 2，免密登录
- 生成密钥对：`ssh-keygen -t rsa -C "my-new-ssh-config"`
  - `-t`：表示 ssh 的密钥类型，常用的有：rsa, ed25519, dss；
  - `-C`：注释或者名称标识，自定义即可；
  - `-f`
- 将**公钥**拷贝到服务器上：`ssh-copy-id -i ~/.ssh/id_rsa.pub user@$HOST -p $PORT`
- 配置本地机器 `~/.ssh/config` 文件

```
Host 远程机器IP
  HostName 远程机器IP
  User root
  IdeitifyFile 私钥文件地址
```
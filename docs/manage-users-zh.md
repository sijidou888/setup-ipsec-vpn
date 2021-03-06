# 管理 VPN 用户

*其他语言版本: [English](manage-users.md), [简体中文](manage-users-zh.md).*

在默认情况下，将只创建一个用于 VPN 登录的用户账户。如果你需要添加，更改或者删除用户，请阅读本文档。

**注：** 现在提供一个辅助脚本，以方便更新 VPN 用户。请参见 [辅助脚本](#辅助脚本)。

首先，IPsec PSK (预共享密钥) 保存在文件 `/etc/ipsec.secrets` 中。如果要更换一个新的 PSK，可以编辑此文件。所有的 VPN 用户将共享同一个 IPsec PSK。

```bash
%any  %any  : PSK "你的IPsec预共享密钥"
```

对于 `IPsec/L2TP`，VPN 用户信息保存在文件 `/etc/ppp/chap-secrets`。该文件的格式如下：

```bash
"你的VPN用户名1"  l2tpd  "你的VPN密码1"  *
"你的VPN用户名2"  l2tpd  "你的VPN密码2"  *
... ...
```

你可以添加更多用户，每个用户对应文件中的一行。**不要** 在用户名，密码或 PSK 中使用这些字符：`\ " '`

对于 `IPsec/XAuth ("Cisco IPsec")`， VPN 用户信息保存在文件 `/etc/ipsec.d/passwd`。该文件的格式如下：

```bash
你的VPN用户名1:你的VPN密码1的加盐哈希值:xauth-psk
你的VPN用户名2:你的VPN密码2的加盐哈希值:xauth-psk
... ...
```

这个文件中的密码以加盐哈希值的形式保存。该步骤可以借助比如 `openssl` 工具来完成：

```bash
# 以下命令的输出为：你的VPN密码1的加盐哈希值
openssl passwd -1 '你的VPN密码1'
```

最后，如果你更换了新的 PSK，则需要重启服务。对于添加，更改或者删除 VPN 用户，一般不需重启。

```bash
service ipsec restart
service xl2tpd restart
```

## 辅助脚本

你可以使用 [这个辅助脚本](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/extras/update_vpn_users.sh) 来更新 VPN 用户。首先下载脚本：

```bash
wget -O update_vpn_users.sh https://raw.githubusercontent.com/hwdsl2/setup-ipsec-vpn/master/extras/update_vpn_users.sh
```

要更新 VPN 用户，从以下选项中选择一个：

**重要：** 这个脚本会将你当前**所有的** VPN 用户移除并替换为你指定的新用户。如果你需要保留当前的 VPN 用户，则必须将它们包含在下面的变量中。或者你也可以按照上面的说明手动更新 VPN 用户。

**选项 1:** 编辑脚本并输入 VPN 用户信息：

```bash
nano -w update_vpn_users.sh
[替换为你自己的值： YOUR_USERNAMES 和 YOUR_PASSWORDS]
sudo sh update_vpn_users.sh
```

**选项 2:** 将 VPN 用户信息定义为环境变量：

```bash
# VPN用户名和密码列表，用空格分隔
# 所有变量值必须用 '单引号' 括起来
# *不要* 在值中使用这些字符：  \ " '
sudo \
VPN_USERS='用户名1 用户名2 ...' \
VPN_PASSWORDS='密码1 密码2 ...' \
sh update_vpn_users.sh
```

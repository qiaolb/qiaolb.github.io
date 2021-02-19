---
layout: blog
title: vCenter升级过程中因密码过期导致的问题处理
date: 2021-02-19 19:00:00
tags:
    - VMware
    - vCenter
---

今天在升级`vCenter` `7.0`时，升级检查时提示错误，错误日志显示如下：

```
2021-02-19T06:12:41.374Z - debug: initiateFileTransferFromGuest error: ServerFaultCode: Failed to authenticate with the guest operating system using the supplied credentials.
2021-02-19T06:12:41.374Z - debug: Failed to get fileTransferInfo:ServerFaultCode: Failed to authenticate with the guest operating system using the supplied credentials.
2021-02-19T06:12:41.374Z - debug: Failed to get url of file in guest vm:ServerFaultCode: Failed to authenticate with the guest operating system using the supplied credentials.
```

经过检查发现是`vCenter`的`root`密码过期导致，`ssh`登录`vCenter`，进入`shell`，执行：

```
root@vcsa [ ~ ]# chage -l root
You are required to change your password immediately (root enforced)
chage: PAM: Authentication token is no longer valid; new one required

```

说明`root`的密码过期，修改密码即可：

```
root@vcsa [ ~ ]# passwd
New password: 
Retype new password: 
passwd: password updated successfully
```

然后重新执行升级程序检查，通过。
---
layout: blog
title: 删除AD用户重建后AAD Connect同步错误
date: 2021-11-19 08:00:00
tags:
    - PowerShell
    - Azure
    - M365
    - AD
---

# 问题背景
我们订阅了`Micorsoft 365`，并通过`Azure AD Connect`将本地的`AD`的用户同步到`Azure AD`中。一次误操作将一个用户从`AD`删除，在同步为执行前，立刻重新创建了正用户。导致该用户无法同步了。

# 问题原因分析
`Azure AD`中的用户通过`Azure AD Connect`同步后，会将`AD`中用户的属性`sourceAnchor`标记到`Azure AD Connect`的属性`immutableId`，用来标识两方的用户。 

`sourceAnchor`有很多中选择，可以参考： [Azure AD Connect：设计概念](https://docs.microsoft.com/zh-cn/azure/active-directory/hybrid/plan-connect-design-concepts)

在目前的版本中，系统将`ConsistencyGuid`作为`sourceAnchor`属性，老版本默认使用`ObjectGuid`。

而`ConsistencyGuid`一般是和`ObjectGuid`相同，当重新创建一个用户时，`ObjectGuid`就会发生改变，最终导致在同步时，`AD`中的`ConsistencyGuid`和`Azure AD`的`immutableId`不同，但用户有相同的`UPN`（`userPrincipalName`），导致同步冲突。

如果删除`Azure AD`的中用户，当然可以简单解决，但也会导致该用户的邮件等全部丢失。

# 解决方法

了解了问题的原因，我们就容易解决。思路是将`AD`中的`ConsistencyGuid`和`Azure AD`的`immutableId`进行统一。

1. 查询`Azure AD`的`immutableId`
    ```
    PS C:\>  Get-MsolUser -UserPrincipalName user1@abc.com | Select-Object UserprincipalName,ImmutableID

    UserPrincipalName         ImmutableId
    -----------------         -----------
    user1@abc.com             K0fuEdJZe0ulRQq3+WlZTA==
    ```

    可以看到该用户的`immutableId`值为`K0fuEdJZe0ulRQq3+WlZTA==`
2. 转换`immutableId`的值
   `immutableId`的值为`Base64`编码的十六进制，需要进行转换，可以用 [在线工具](https://the-x.cn/base64/) 进行处理，输出结果需要选择十六进制。

   通过转换我们得到的值类似： `2B 47 EE 11 D2 59 7B 4B  A5 45 0A B7 F9 69 59 4C`
3. 修改`AD`中的`ConsistencyGuid`
   登录`AD`服务器，打开`ADSI编辑器`，找到该用户，点开属性，找到属性`ms-DS-ConsistencyGuid`修改值为上步转换的值。
4. 同步
   可以等待同步，也可以手工同步。同步后确定结果是否正确。

# 后续补充

在本次问题解决中，也尝试过，去修改`Azure AD`的`immutableId`，感觉这个更合理一些。但在修改过程中遇到2个问题：
1. 如果`Azure AD`中用户的状态是同步状态，这个值是无法修改的，必须等这个用户变成非同步用户或者关闭同步
2. 我从测试中修改过`Azure AD`的`immutableId`，然后通过手工同步发现依旧有错，后续没有再测试。理论上这个方法应该可以

记录一下两个命令：

* 修改`Azure AD`的`immutableId`
  ```
  Get-MsolUser -UserPrincipalName user1@abc.com | Set-MsolUser -ImmutableId L0b1Dn3oIkGiFLPW9fhY+Q==
  ```
* 关闭同步
  ```
  Set-MsolDirSyncEnabled -EnableDirSync $false
  ```

  ***注意：*** 官方文档说明修改后如果要重新打开需要等待`72`小时。

另外，网上找到有人说可以将`Azure AD Connect`卸载后，重新安装来解决，其思路也是通过重新对应`AD`和`Azure AD`的用户，但这个方法我试过，问题没有解决。

最后，本次误操作的最好解决办法是，打开`AD`的回收站功能，具体在`AD`的管理中心进行修改，这样就可以完全避免这种问题了。

---
layout: blog
title: Azure Bicep开发环境搭建 
date: 2022-02-08 08:00:00
tags:
    - Azure
    - Bicep
    - ARM
    - VSCode
---

最近在`MacOS`搭建`Bicep`环境，发现`VS Code`始终无法成功识别`Bicep`，`Bicep`插件在启动时依赖`.net`环境，虽然手动安装了`.net`环境，但依然无法启动。

参考：

https://github.com/dotnet/vscode-dotnet-runtime/blob/main/Documentation/troubleshooting-runtime.md#install-script-timeouts 

https://www.azuredeveloper.cn/article/azure-tutorial-auzre-bicep-introduction 


修改设置，增加以下内容：

```
    "dotnetAcquisitionExtension.existingDotnetPath": [
        {"extensionId": "ms-azuretools.vscode-bicep", "path": "/usr/local/share/dotnet/dotnet"},
        {"extensionId": "msazurermtools.azurerm-vscode-tools", "path": "/usr/local/share/dotnet/dotnet"}
    ]
```

这样就可以正常使用了。

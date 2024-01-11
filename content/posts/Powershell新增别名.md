+++
title = "Powershell新增别名"
date = 2024-01-11T13:50:37+08:00
draft = true
author = 'vdong'
categories = ['技术']
tags = ['powershell']
+++

## 查看别名

```Powershell
Get-Alias
```

## 创建永久别名

在 Powershell 中使用 Set-Alias 和 New-Alias 定义的别名，在此session关闭后即会失效，防止此现象，可以将定义别名的命令写入 Windows Powershell profile 文件中。查看此文件位置:
```Powershell
$profile
```
一般该文件在没有创建时是不存在的，使用以下命令创建：
```Powershell
New-Item -Type file -Force $profile
```
可以在该文件中定义函数和别名等，如下：
```Powershell
function py-v{
    python --version
}

Set-Alias -Name py-v -Value py-v
set-Alias -Name vim -Value nvim

Invoke-Expression (&starship init powershell)
```
重新打开 Powershell ，就会加载 $profile 文件。

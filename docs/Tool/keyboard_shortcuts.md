---
title: 工具设置
date: 2020-11-11`
--- 

# 给ssh挂代理

C:\\Users\\user\\.ssh\\config 文件

```
Host *
    ProxyCommand  C:\Program Files\Git\mingw64\bin\connect.exe -H 127.0.0.1:7890 %h %p 
```




---
layout: post
title: "思考"
description: ""
category: 思考
tags: [blog]
---
{% include JB/setup %}

### Shell 工具脚本

#### 批量修改文件名
```shell
#!/bin/bash

for name in /path/*
do
        long_name=$name
        short_name="${long_name##*ll}"
        mv ${long_name}  "/path/virtualhost_and_mysql_install_${short_name}"
done
```

---
title: Docker-Start-Up-1
date: 2017-07-17 00:16:52
categories:
- 笔记
tags:
- docker
- mac
---

- 使用 `docker-machine` & 使用阿里云加速国内 docker hub 访问速度

  + [Docker Hub 镜像站点 操作文档](https://cr.console.aliyun.com/?spm=5176.1971733.0.2.1dcs9Z#/accelerator)
  + 开启终端自动使用 `docker-machine`

    `.bash_profile` 添加
    ```
    eval $(docker-machine env default)
    ```
  + 开启启动 `docker-machine`
    ```
    cd ~/Library/LaunchAgents
    ```
    创建 `com.docker.machine.default.plist` 文件,添加如下内容:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
        <dict>
            <key>EnvironmentVariables</key>
            <dict>
                <key>PATH</key>
                <string>/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin</string>
            </dict>
            <key>Label</key>
            <string>com.docker.machine.default</string>
            <key>ProgramArguments</key>
            <array>
                <string>/usr/local/bin/docker-machine</string>
                <string>start</string>
                <string>default</string>
            </array>
            <key>RunAtLoad</key>
            <true/>
        </dict>
    </plist>
    ```


- Mac 下使用 `docker-machine` ，本机无法访问 docker machine 内 docker 容器问题解决

  ```
  sudo route add 172.17.0.0/16 $(docker-machine ip <default>)
  ```

  via:
  
  [https://github.com/moby/moby/issues/19119](https://github.com/moby/moby/issues/19119)

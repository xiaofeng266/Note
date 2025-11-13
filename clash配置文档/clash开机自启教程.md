# clash开机自启教程



### 1. 创建shell脚本

```shell

sudo nano /usr/local/bin/start-mihomo.sh
```

脚本内容示例：

```shell
#!/bin/bash

# 等待用户登录完成
sleep 10

# 切换到mihomo目录
cd /home/$(whoami)/.config/mihomo/

# 执行clash-linux
./clash-linux

echo "clash启动完成"
```

给脚本执行权限：

```

sudo chmod +x /usr/local/bin/start-mihomo.sh
```

### 2. 创建systemd用户服务（更好的方式）

由于mihomo通常需要在用户会话中运行，建议创建用户级服务：

```shell

# 创建用户systemd目录
mkdir -p ~/.config/systemd/user/

# 创建服务文件
nano ~/.config/systemd/user/mihomo.service
```

服务文件内容：

```shell

[Unit]
Description=Mihomo Service
After=network.target
Wants=network.target

[Service]
Type=simple
WorkingDirectory=/home/%u/.config/mihomo/
ExecStart=/home/%u/.config/mihomo/clash-linux
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

### 3. 启用用户服务

```shell

# 重新加载用户systemd配置
systemctl --user daemon-reload

# 启用服务（开机自启动）
systemctl --user enable mihomo.service

# 启动服务
systemctl --user start mihomo.service

# 检查状态
systemctl --user status mihomo.service
```



判断systemd用户服务是否设置成功，可以通过以下几个步骤来验证：

### 4. 检查服务是否正常启用

```shell

# 检查服务是否已enable（开机自启动）
systemctl --user is-enabled mihomo.service

# 查看服务状态
systemctl --user status mihomo.service
```

**成功标志**：

`is-enabled` 返回 `enabled`

`status` 显示 `active (running)`

### 5. 手动测试服务启动

```

# 停止服务（如果正在运行）
systemctl --user stop mihomo.service

# 启动服务
systemctl --user start mihomo.service

# 再次检查状态
systemctl --user status mihomo.service
```

### 6. 检查进程是否运行

```

# 查看clash-linux进程是否在运行
ps aux | grep clash-linux

# 或者查看mihomo相关进程
ps aux | grep mihomo

# 查看端口占用（clash默认端口7890）
netstat -tlnp | grep 7890
# 或者使用ss命令
ss -tlnp | grep 7890
```
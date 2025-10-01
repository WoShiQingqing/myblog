+++
title = '从 502 到恢复：我把一台新主机拉上线的过程（附脚本）'
date = 2025-10-01T10:44:19+08:00
draft = false
+++


## Context
目标：新主机初始化、Nginx 反代、模拟一次 502 错误

## Steps
1. init.sh 初始化
2. Flask 后端（8081）
3. Nginx 配置（proxy_pass）
4. 故障注入：写错端口 → 502
5. diagnose.sh 三板斧
6. systemd 常驻 gunicorn
7. smoke.sh 验证

## Data
- 截图占位符（502、200 OK、diagnose.sh 输出、systemctl status）

## Pitfalls
- 端口对不上
- 默认站点干扰
- WSL2 没开启 systemd

## Wrap-up
已完成：从错误到恢复的闭环  
下一步：Docker 化 → 镜像瘦身 → 非 root

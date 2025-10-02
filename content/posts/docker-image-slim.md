+++
title = 'Docker 镜像优化实战：从 900MB 到 5MB，multi-stage 构建与非 root'
date = 2025-10-02T11:35:11+08:00
draft = true
+++


## Context
目标：用 Go 写的 demo 服务，从最初的近 1GB 镜像，优化到不足 5MB。  
核心手法：multi-stage build、静态编译、scratch 空镜像。

## Steps
1. Basic 阶段：直接用官方 `golang` 镜像  
   - 镜像体积：896MB  
   - 特点：开发方便，但太臃肿。  

2. Multi-stage 阶段：builder + debian  
   - 镜像体积：87MB  
   - 特点：编译在 builder，运行只要二进制。  

3. Slim 阶段：scratch + 非 root 用户  
   - 镜像体积：4.7MB  
   - 特点：极致瘦身 + 安全性更高。  

## Data
- `docker images` 截图对比  
- `curl http://localhost:8080` 访问结果  

## Pitfalls
- glibc 版本不匹配（解决办法：静态编译）  
- root 用户运行风险  

## Wrap-up
完成：镜像优化的三个阶段。  
下一步：W3 — 多容器编排（Flask + Redis + Nginx）。

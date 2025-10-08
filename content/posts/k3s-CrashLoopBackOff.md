+++
title = '用 k3s 把 API+Worker 跑起来：HPA 首次生效与一次 CrashLoopBackOff 的排障'
date = 2025-10-08T19:00:23+08:00
tags = ["Kubernetes", "DevOps", "CloudNative", "Debug", "CrashLoopBackOff"]
draft = false
+++


## Context  
目标：  
验证 Ingress → jobflow-api → Redis 链路是否可通，并模拟一次容器崩溃（CrashLoopBackOff），完成定位与恢复。

环境：  
- kind 单节点集群  
- 命名空间：`jobflow`  
- 组件：`jobflow-api`、`jobflow-worker`、`redis`、`ingress-nginx`  
- 域名：`jobflow.local`  
- 服务端口：`8080`

---

## Steps  

1. **Ingress 验证**
```bash
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
curl -i -H "Host: jobflow.local" http://127.0.0.1:8080/status/123
```
✅ 返回：
```bash
{"status":"not found"}
```

2. **注入 CrashLoopBackOff 故障**
```bash
kubectl -n jobflow patch deploy jobflow-api --type='json' -p='[
  {"op":"add","path":"/spec/template/spec/containers/0/command","value":["/bin/sh","-c","echo BOOM && exit 1"]}
]'
kubectl -n jobflow get pods -w
```
'Pod'状态进入：'CrashLoopBackOff'

3. **查看故障详情**
```bash
kubectl -n jobflow describe pod -l app=jobflow-api
```
输出：
```bash
Back-off restarting failed container
```

4. **确认退出码**
```bash
kubectl -n jobflow get pod -l app=jobflow-api \
  -o jsonpath='{.items[0].status.containerStatuses[0].lastState.terminated.exitCode}'
```
结果'128'

5. **执行回滚**
```bash
kubectl -n jobflow rollout undo deploy/jobflow-api
kubectl -n jobflow get pods -w
```
✅'Pod'恢复为 Running

6. **验证服务恢复**
```bash
curl -i -H "Host: jobflow.local" http://127.0.0.1:8080/status/123
```
✅ 正常返回：
```bash
{"status":"not found"}
```


## Data

截图占位：
- 'kubectl get pods'出现'CrashLoopBackOff'
- 'kubectl describe'显示'Back-off restarting failed container'
- 'kubectl rollout undo'后恢复'Running'
- 浏览器访问'jobflow.local:8080/status/123'成功页面


## Pitfalls

'CrashLoopBackOff'常见原因：
- 启动命令异常退出（'exit 1'）
- 环境变量缺失（如'REDIS_ADDR'）
- 探针检测失败（'livenessProbe'）

避坑：
- 用'exitCode'判断异常类型
- 用'kubectl describe'看事件，而不是只看'logs'
- 频繁重启时用'rollout undo'回滚更安全


## Wrap-up

本周闭环：
从「Ingress 通路 → 故障注入 → 诊断分析 → 回滚恢复」完整跑通。

下一步：
引入健康探针（livenessProbe + /ping 路由）
→ 让服务具备自愈能力。

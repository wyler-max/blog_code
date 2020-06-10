---
title: Prometheus--（五）Prometheus 告警
date: 2020-06-10 20:44:22
tags:
- Monitor
- Prometheus
categories:
- DevOps
---
## Prometheus 告警

### 概览

用普罗米修斯发出警报分为两部分。

1. 普罗米修斯服务器中的警报规则向警报管理器发送警报。
2. Alertmanager管理这些警报，包括静音、抑制、聚合和通过电子邮件、通话通知系统和聊天平台等方法发送通知。

设置警报和通知的主要步骤是：
- 设置和配置Alertmanager
- 配置普罗米修斯与警报管理器对话
- 在普罗米修斯中创建警报规则

### 告警管理器 Alertmanager

Alertmanager负责处理来自客户端应用程序（如Prometheus服务器）的警报。它负责重复数据消除、分组，并将它们路由到正确的接收器集成（如邮件、WebHook等）。它还负责消除和抑制警报。

#### 分组 Grouping

分组将类似性质的警报分类为单个通知。当许多系统同时发生故障，并且可能同时触发成百上千个警报时，这在较大的停机期间尤其有用。

作为用户，用户只想获得一个页面，同时仍然能够确切地看到哪些服务实例受到了影响。因此，我们可以将Alertmanager配置为按其群集和alertname对警报进行分组，以便发送单个压缩通知。

警报分组、分组通知的计时以及这些通知的接收者由配置文件中的路由树配置。

#### 抑制 Inhibition

抑制是是指当某些其他警报已经触发时，抑制某些警报的通知。

通过 Alertmanager 的配置文件配置。

#### 静默 Silences

可以在给定时间内将警报静音。静默是基于匹配器配置的，就像路由树一样。将检查传入警报是否与活动静默的所有相等或正则表达式匹配项匹配。如果匹配到，就不会发送该警报的通知。

通过 Alertmanager 的web界面中配置静默。

#### 客户端行为 Client behavior

Alertmanager对其客户端的行为有特殊要求。只与Prometheus不用于发送警报的高级用例相关。

#### 高可用 High Availability

Alertmanager支持配置以创建高可用性群集。这可以使用--cluster-*标志进行配置。

重要的不是在Prometheus和它的Alertmanagers之间进行负载平衡，而是将Prometheus指向一个所有Alertmanagers的列表。

### 安装 alertmanager

下载
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.19.0-rc.0/prometheus-2.19.0-rc.0.linux-amd64.tar.gz

tar zxvf prometheus-2.19.0-rc.0.linux-amd64.tar.gz
cd prometheus-2.19.0-rc.0.linux-amd64

./prometheus --help
```

配置 alertmanager.yml
```yml
global:
route:
  group_by: ['alertname', 'instance']
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 1m
  receiver: 'dingding'
receivers:
- name: 'dingding'
  webhook_configs:
  - url: http://localhost:8060/dingtalk/webhook/send
    send_resolved: true
```

### 安装 Webhook-dingtalk

下载：
```bash
wget https://github.com/timonwong/prometheus-webhook-dingtalk/releases/download/v1.4.0/prometheus-webhook-dingtalk-1.4.0.linux-amd64.tar.gz

tar prometheus-webhook-dingtalk-1.4.0.linux-amd64.tar.gz
cd prometheus-webhook-dingtalk-1.4.0.linux-amd64
```

启动：run.sh
```bash
#!/usr/bin/env bash

nohup ./prometheus-webhook-dingtalk --ding.profile="webhook=https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx" >/dev/null &
```

测试钉钉：
```bash
curl -H "Content-Type:application/json"  -d '{"msgtype": "text","text": {"content": "我就是我, 是不一样的烟火"}}' https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx
```

测试webhook-dingding：
```bash
curl -H "Content-Type:application/json"  -d '{ "version": "4", "status": "firing", "description":"description_content"}' http://localhost:8060/dingtalk/webhook/send
```

配置多个webhook可参考[链接](https://theo.im/blog/2017/10/16/release-prometheus-alertmanager-webhook-for-dingtalk/)

### 配置Prometheus

配置 prometheus.yml 中的 alerting和 rule_files（上一节已配置好）。一旦触发规则，即触发钉钉机器人报警。


### Other

Prometheus WebUI：
```
http://outer-ip:6090/prom/targets
```

线上Alertingmanager WebUI：
```
http://outer-ip:6093/alertmanager/#/alerts
```

关系
prometheus-client --- 采集信息 --> prometheus-server --- 根据rules，发送消息给 -—> alertmanager --- 调用第三方 -—> dingding-webhook



**参考链接**：

https://www.cnblogs.com/g2thend/p/11865302.html
https://www.jianshu.com/p/9fdd4f3497c6
https://theo.im/blog/2017/10/16/release-prometheus-alertmanager-webhook-for-dingtalk/



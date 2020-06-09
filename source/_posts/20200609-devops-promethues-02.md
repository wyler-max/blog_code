---
title: Prometheus--（二）安装 Prometheus
date: 2020-06-09 22:29:06
tags:
- Monitor
- Prometheus
categories:
- DevOps
---
## 概览

Prometheus是一个监控平台，它通过在这些目标上刮取度量HTTP端点来收集来自被监控目标的度量。
下面介绍普罗米修斯的安装、配置和监控配置。


### 下载

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.19.0-rc.0/prometheus-2.19.0-rc.0.linux-amd64.tar.gz

tar zxvf prometheus-2.19.0-rc.0.linux-amd64.tar.gz
cd prometheus-2.19.0-rc.0.linux-amd64

./prometheus --help
```

### 配置
```yml
# 全局配置
global:
  scrape_interval:     15s  #采样频率
  evaluation_interval: 15s  #规则触发频率

rule_files:
  # - "first.rules"
  # - "second.rules"

# 采样配置, 由于普罗米修斯将自己的数据公开为一个HTTP端点，因此它可以对自己的健康状况进行清理和监视
scrape_configs:
  # 作业名
  - job_name: prometheus
    static_configs:
      # 采样目标地址
      - targets: ['localhost:9090']
```

### 启动
```bash
./prometheus --config.file=prometheus.yml

# 或者指定端口（default 9090）
./prometheus --config.file=prometheus.yml --web.enable-lifecycle   --web.external-url="http://ip:port/prom"  \
 --web.route-prefix=prom   --web.listen-address="0.0.0.0:port" --log.level="info"
```

重启
```bash
curl -XPOST http://127.0.0.1:port/prom/-/reload
```

### 监控面板

面板地址：
```
http://localhost:9090
```

API总地址：
```
http://localhost:9090/metrics
```

### 使用内建表达式浏览器

访问如下地址，并选择 Console
```
http://localhost:9090/graph
```

选择关于Prometheus自身的一个度量选项：
```
promhttp_metric_handler_requests_total
```

这将返回多个不同的时间序列（以及为每个时间序列记录的最新值），所有时间序列的度量名称都是promhttp_metric_handler_requests_total，但标签不同。这些标签指定不同的请求状态。

过滤http_code 为200的请求：
```
promhttp_metric_handler_requests_total{code="200"}
```

要计算返回的时间序列数：
```
count(promhttp_metric_handler_requests_total)
```

过滤指定的job和group
```
http_requests_total{job="prometheus",group="canary"}
```

过滤5分钟内指定job的作业
```
http_requests_total{job="prometheus"}[5m]
```

详细可参考：[expression language documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)



### 其他可视化工具

Grafana

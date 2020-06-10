---
title: Prometheus--（四）SpringBoot集成Prometheus
date: 2020-06-10 20:23:18
tags:
- Monitor
- Prometheus
categories:
- DevOps
---
## SpringBoot集成Prometheus

### 依赖

在 pom.xml中添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.0.4</version>
</dependency>
```

### 配置 client

在 application.properties 添加
```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus,restart,pause
management.endpoint.pause.enabled=true
management.endpoint.restart.enabled=true
```

### 启动 client 服务

### 配置 Prometheus
修改prometheus.yml，默认端口号对应application.yml的server.port，检查端口开放状态
```yml
global:
  scrape_interval:     1m #采样频率
  evaluation_interval: 1m #执行 rules 频率

# 采样配置
scrape_configs:
  - job_name: 'test-server'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['192.168.1.1:8000','192.168.1.2:8000']

# 报警配置
alerting:
  alertmanagers:
  - path_prefix: /alertmanager
    scheme: http
    static_configs:
    - targets:
      - 127.0.0.1:9090 #alertmanager 端口

# 报警规则
rule_files:
  - 'rules/alert-instance-rules.yml'
```

rules/alert-instance-rules.yml
```yml
# severity按严重程度由高到低：red、orange、yellow、blue
groups:
  - name: jvm-instance-alerting
    rules:

      # down了超过30秒
      - alert: instance-down
        expr: up == 0
        for: 30s
        labels:
          severity: yellow
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 30 seconds."

      # down了超过1分钟
      - alert: instance-down
        expr: up == 0
        for: 1m
        labels:
          severity: orange
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."

      # down了超过5分钟
      - alert: instance-down
        expr: up == 0
        for: 5m
        labels:
          severity: red
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

```

### 启动 Prometheus

脚本启动
```bash
#!/usr/bin/env bash

cd /data/prometheus/prometheus-2.1.0.linux-amd64

nohup ./alertmanager --config.file=alertmanager-config.yml  --web.listen-address=":6093"  --web.external-url=http://ip:6093/alertmanager --web.route-prefix=alertmanager --log.level=info >/dev/null &
```

reload
```bash
#!/usr/bin/env bash
curl -XPOST http://localhost:6090/prom/-/reload
```

服务启动
```bash
systemctl restart prometheus.service
```

### 配置 Grafana

配置Grafana 监控 JVM(Micrometer) Dashboard模板，模板id：4701

1. 在左侧边栏选择Dashboards --> Manage 菜单；
2. 在主界面，点击import按钮；
3. Granfana.com Dashboard中填写 4701，自动跳转到import详情界面；
   可以通过以下地址查询更多模板：https://grafana.com/grafana/dashboards?dataSource=prometheus&collector=nodeExporter
4. Prometheus Data Source选择上面的数据源名称，点击import按钮；
5. 进入仪表盘查看相应的监控指标；





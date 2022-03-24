# Promethues

> [Prometheus简介 · Prometheus中文技术文档](https://www.prometheus.wang/quickstart/why-monitor.html)

## 一、安装

docker-compose 安装 Promethues 和 Grafana

```yml
version: '3.8'
services:
  prometheus:
    container_name: promethues-test
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  grafana:
    container_name: grafana-test
    image: grafana/grafana
    ports:
      - "3000:3000"
```

prometheus.yml

```yml
global:
  scrape_interval: 60s
  evaluation_interval: 60s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: [ 'localhost:9090' ]
        labels:
          instance: prometheus
```

---

## 二、使用

### 2.1 Prometheus 的简单使用

Prometheus 安装完成后，配置文件里添加了第一个 Job，监控 Prometheus 的信息，可以通过自带的面板查询指标信息，如下图所示。

![image-20220324173602955](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324173602955.png)

### 2.2 Grafana 的简单使用

安装好 Grafana 后，在 Configuration - DataSource 里配置 Prometheus 数据源。

![image-20220324174723513](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324174723513.png)

添加 Prometheus Dashboard。

![image-20220324174845027](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324174845027.png)

配置好之后，即可通过 Grafana 大盘更方便的浏览监控数据。

![image-20220324175015122](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324175015122.png)

---

## 三、监控Java应用 

### 3.1 新建SpringBoot 应用

1. 新建一个 SpringBoot 应用，依赖如下

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-core</artifactId>
        </dependency>
    
```

1. spring-boot-starter-actuator 是 Spring 自带的监控组件，提供了多种不同的 endpoint，例如 health，beans，metrics 等，但是这些数据的格式不符合 Prometheus 规范，所以需要引入其他的依赖。
2. 引入 micrometer 依赖后，可以暴露 prometheus endpoint，返回符合 Prometheus 格式的指标数据。

配置文件 application.yml

```yml
management:
  endpoint:
    metrics:
      enabled: true
  endpoints:
    web:
      exposure:
        include:
          - prometheus
          - health
```

服务启动后，返回 `/actuator/prometheus` 接口，可以看到 Prometheus 指标数据。

![image-20220324192314074](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324192314074.png)



### 3.2 Prometheus 采集 Java 应用监控数据

1. 修改配置文件，添加新的 Job

prometheus.yml

```yml
global:
  scrape_interval: 60s
  evaluation_interval: 60s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: [ 'localhost:9090' ]
        labels:
          instance: prometheus

  - job_name: 'java-demo'
    scrape_interval: 5s
    # 指标数据接口地址
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: [ '{java-app-ip}:{port}' ]
```

2. 添加 Grafana 大盘，监控 JVM 信息。

选择 Import Dashboard，输入对应的 ID 即可。当前用的是 [JVM (Micrometer) dashboard for Grafana](https://grafana.com/grafana/dashboards/4701)，然后选择对应的 Prometheus 数据源即可。

![image-20220324201215936](https://cdn.jsdelivr.net/gh/chenqi4547/image-repo@master/prom/image-20220324201215936.png)



 
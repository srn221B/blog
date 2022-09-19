---
title: "Flaskを活用して自作exporter作成。Prometheus->Grafanaで可視化"
date: 2020-01-01T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Flask", "Prometheus", "Grafana"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "old"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/srn221B/blog/tree/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## 概要
Flaskを活用して自作exporterを作成し、Prometheusでmetrics取得->Grafanaでmetrics可視化を行う手順の備忘録

## 手順
### 構成
```
.
├── docker-compose.yml
├── flask
│   ├── Dockerfile
│   └── app
│       └── app.py
├── grafana
│   ├── Dockerfile
│   └── datasource.yml
└── prometheus.yml
```

### flask/Dockerfile
```Dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install python3 python3-pip -y
RUN pip3 install flask prometheus-client
RUN mkdir /app
```
### flask/app.py
- pythonの[PrometheusClientライブラリ](https://github.com/prometheus/client_python)を使ってexporter化。`curl http://localhost:3000/hoge`でGauge型のmetricsが増減するexporterです。
```python
from flask import Flaskimport json
import queue
from werkzeug.middleware.dispatcher import DispatcherMiddleware
from prometheus_client import make_wsgi_app,Gauge

app = Flask(__name__)
G1 = Gauge('Gauge1','Gauge test')
G2 = Gauge('Gauge2','Gauge test')

@app.route('/upG1',methods=["GET"])
def upG1():
    G1.inc()
    return "upG1"

@app.route('/upG2',methods=["GET"])
def upG2():
    G2.inc()
    return "upG2"

@app.route('/downG1',methods=["GET"])
def downG1():
    G1.dec()
    return "downG1"

@app.route('/downG2',methods=["GET"])
def downG2():
    G2.dec()
    return "downG2"

app.wsgi_app = DispatcherMiddleware(app.wsgi_app, {
    '/metrics': make_wsgi_app()
})

if __name__ == '__main__':
    G1.set(0)
    G2.set(0)
    app.run(host='0.0.0.0',port=5000)
```
### grafana/Dockerfile
```Dockerfile
FROM grafana/grafana:master
COPY ./datasource.yml /etc/grafana/provisioning/datasources/
```
### grafana/datasource.yml
- https://grafana.com/docs/grafana/latest/administration/provisioning/#datasources
```yaml
datasources:
  - name: prometheus
    type: prometheus
    access: proxy
    url: "http://prometheus:9090"
```
### prometheus.yml
- 15秒間隔でexporterからmetricsを取得するように設定しています。
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['flask:5000']
```
### docker-compose.yml
```yaml
version: "3"
services:
  flask:
    build: ./flask
    command: python3 app/app.py
    volumes:
       - ./flask/app:/app
    ports:
       - 5000:5000
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
  grafana:
      build: ./grafana
      ports:
        - 3000:3000
      environment:
        - GF_SECURITY_ADMIN_PASSWORD=password
        - GF_USERS_ALLOW_SIGN_UP=false
```

## 起動確認
- 「docker-compose build」→「docker-compose up -d」を行って３つのコンテナが立ち上がっているのを確認
- `http://localhost:3000`でGrafanaへ接続
  - ID：admin、PASSWORD：passwordでsign in
  - 「Configuration」→「Data Sources」に「Prometheus」があればおk
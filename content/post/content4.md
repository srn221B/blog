---
title: "Dockerを使ったGrafanaのDatasourceとDashboardの起動時追加方法"
categories: ["", ""]
date: 2021-01-17T20:54:56+09:00
description: "メモ"
draft: false
image: ""
tags: ["spark", "zeppelin","kubernetes","cassandra"]
author: "srn221B"
---

# こんにちは
ブログ書くの久々すぎて書き方忘れた。うそです。  
去年の卒研終わったあたりから作り始めたこのサイトですが、あまりにも初期状態すぎたので、重い腰をあげて「HugoのthemaをBeautifulHugoからCupperに変更」「repositoryを整理」の２点をしました。一新したのでブログ書くモチベもこれで上がるはず（？）仕事もだいぶ慣れてきたので2022年は些細なことでももう少し書いていきたいな。。。  
タイトル通り
「Dockerを使ったGrafanaのDatasourceとDashboardの起動時追加方法」のメモを簡単に備忘録としてまとめておきます。

# Datasourceの起動時追加方法について
Dashboardの起動時追加を設定する前に、まずはこっちを作っていきます。
## 構成
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
ここでは
grafanaで可視化するサーバーは**prometheus**で、prometheusでの監視は**flaskを用いた自作exporter**としています。

## flask
### Dockerfile
```
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install python3 python3-pip -y
RUN pip3 install flask prometheus-client
RUN mkdir /app
```
### `app.py`
pythonの[PrometheusClientライブラリ](https://github.com/prometheus/client_python)を使ってexporter化しましした。testしやすいように`curl http://localhost:3000/hoge`でGauge型のmetricsが増減するexporterです。
```
from flask import Flask,render_template,request
import json
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
## grafana
### Dockerfile
とりあえず最低限。
```
FROM grafana/grafana:master
COPY ./datasource.yml /etc/grafana/provisioning/datasources/
```
### datasource.yml
起動時追加するdatasourceについて書きます。詳しいパラメータについては[Grafanaの公式ドキュメント](https://grafana.com/docs/grafana/latest/administration/provisioning/#datasources)に書いてあります。
```
datasources:
  - name: prometheus
    type: prometheus
    access: proxy
    url: "http://prometheus:9090"
 # 複数書きたい場合はこんな感じで
 #- name: prometheus
 #  type: prometheus
 #  access: proxy
 #  url: "http://prometheus:9090"
```
## prometheus.yml
15秒間隔でexporterからmetricsを取得するように設定しています。
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['flask:5000']
```
## docker-compose.yml

```

```
---
title: "GrafanaでのDashboard作成を自動化する"
date: 2020-01-01T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Grafana"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: true
description: "old"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://user-images.githubusercontent.com/60976262/191047722-dd7ed9ad-6e00-4b50-8073-24ae9602b8d6.png" # image path/url
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
[Flaskを活用して自作exporter作成。Prometheus->Grafanaで可視化](https://467tn.com/post/content3/)の続き。現状のままだとコンテナRestart都度GrafanaへDashboardの作成をしなくてはならないので、起動時に自動でDashboardが作成できるようにする備忘録

## 手順
### 構成
```
├── docker-compose.yml
├── flask
│   ├── Dockerfile
│   └── app
│       └── app.py
├── grafana
│   ├── Dockerfile　# 変更
│   ├── dashboard.yml　# 作成
│   ├── datasource.yml　
│   └── prometheus
│       └── <hoge>.json　# 作成
└── prometheus.yml
``` 
- Dashboard自動作成設定ファイル「dashboard.yml」、自動作成するDashboardのファイル「<hoge>.json」を作成。
- 追加ファイルをコンテナへ置くためにGrafanaの「Dockerfile」を変更。

### grafana/prometheus/<hoge>.json
- `http://localhost:3000`でGrafanaへ接続
- 「Create」→「Dashboard」で自動作成するDashboardを作成する。
![Dashboardの追加準備](https://user-images.githubusercontent.com/60976262/191035233-1b1b1b1f-aaf9-4522-af1a-c7046e75dc65.png "Dashboardの追加準備")   
- 「Share dashboard」→「Export」→「Save to file」でJSONファイルを出力。「./grafana/prometheus」配下にファイルを置く。
![Dashboardの追加準備2](https://user-images.githubusercontent.com/60976262/191037763-82a5042f-97f4-43a7-8950-f4ac29c1442b.png "Dashboardの追加準備2")  

### grafana/dashboard.yml
- https://grafana.com/docs/grafana/latest/administration/provisioning/#datasources
```yaml
apiVersion: 1
providers:
  - name: 'prometheus metrics'
    orgId: 1
    folder: ''
    folderUid: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: false
    options:
      path: /etc/grafana/provisioning/dashboards/prometheus
      foldersFromFileStructure: true
```
### grafana/Dockerfile
```Dockerfile
FROM grafana/grafana:master
COPY ./datasource.yml /etc/grafana/provisioning/datasources/
COPY ./dashboard.yml /etc/grafana/provisioning/dashboards/ 
COPY ./prometheus /etc/grafana/provisioning/dashboards/prometheus
```

## 起動確認
- 「docker-compose build」→「docker-compose up -d」
- `http://localhost:3000`でGrafanaへ接続。
  - ID：admin、PASSWORD：passwordでsign in

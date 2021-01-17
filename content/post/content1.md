---
title: "pysparkをjupyter-notebookから使う"
categories: ["spark"]
date: 2020-03-28T00:08:00+09:00
description: "spark2.4.5をjupyter-notebook6.0.3から使う方法です。"
draft: false
image: ""
tags: ["pyspark", "jupyter"]
author: "srn221B"
---

今までpysparkをいじるときはコンソールでネチネチやっていたのですが、
pythonをjupyter-notebookで開発しているときに便利だなあと思ったので、思い切ってpysparkをいじるときもjupyter-notebookを使うようにしました。その時の備忘録。


## 前提条件
- Sparkがインストールされていること（[この記事](https://qiita.com/Hiroki11x/items/4f5129094da4c91955bc)がわかりやすかったです。)
- jupyterがインストールされていること(`pip`または`conda`でインストール)

## 環境
- OS：Mac
- python：Python 3.7.6
- Spark：2.4.5
- jupyter-notebook：6.0.3

## Sparkの設定
Spark上の環境ファイルのテンプレート(`spark-env-sh.template`)をコピーします。
```
$ cd $SPARK_HOME/conf
$ cp spark-env.sh.template spark-env.sh
```
コピーしたファイルを書き換えていきます。
```
$ vim spark-env.sh
export PYSPARK_PYTHON=/usr/local/bin/python3 #pythonの場所
export PYSPARK_DRIVER_PYTHON=jupyter 
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
```

## 環境変数の設定
以下の環境変数を`.bashrc`,`.bash_profile`に書きます。
```
$ vim ~/.bashrc
SPARK_HOME=#Sparkをインストールした場所
PATH=$SPARK_HOME/bin:$PATH
```

## 動作確認
早速、jupyterを使って簡単な演算を実装して見ます。  
```
$ pyspark
http://localhost:8888/?token= *Token*が表示されます
```
<http://localhost:8888/login>を開きます。表示された**Token**を元にログインします。  
![login画面](https://srn221b.github.io/images/content1_1.png)  
Pythonファイルを作ります（`New`>`python3`）  
![ファイル作成画面](https://srn221b.github.io/images/content1_2.png)    
以下のようにsc(SparkSession)が使えていたら完了。  
![実装画面](https://srn221b.github.io/images/content1_3.png)    

## 終わり
**快適な開発環境**が整いました:heart_eyes: 

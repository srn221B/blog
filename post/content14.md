---
title: "ワークフロー管理についてメモ"
publishedAt: 2024-01-09
description: "workflow"
slug: "content14"
isPublish: true
tags: ["Workflow"]
---

WEB+DB PRESS plusの「ビッグデータを支える技術」という本の180ページから書かれているワークフロー管理について
ワークフロー周りを開発している身としては忘れないようにしたい内容だと思ったので、備忘録として雑に残します。


## ワークフロー管理

### ワークフロー管理ツール

主な役割としては「定期的にタスクを実行すること」と「異常を検知してその解決を手助けすること」。
例えば以下のようなものがある

|名称|種類|開発元|
|---|---|---|
|Airflow|スクリプト型|Airbnb|
|Azkaban|宣言型|Linkedin|
|Digdag|宣言型|TreasureData|
|Oozie|宣言型|The Apache Software Foundation|
|Luigi|スクリプト型|Spotify|
|Prefect|スクリプト型|Prefect Technologies|
|Metaflow|スクリプト型|Netflix|
|Dagster|スクリプト型|Dagster|

### 基本機能とビッグデータで求められる機能

- タスクを定期的なスケジュールで実行し、その結果を通知する
- タスク間の依存関係を定めて、決められた順にもれなく実行する
- タスクの実行結果を保持し、エラー発生時には再実行できるようにする

### 宣言型とスクリプト型

- 宣言型のメリット
  - 最小限の記述でタスクを定義できる
- スクリプト型のメリット
  - 柔軟性

ETLのプロセスにはスクリプト型のツール、SQLの実行には宣言型のツールをなどと使い分けるのも一つの方法。

## エラーからのリカバリ方法を先に考える

- あらかじめ予期せぬエラーが発生する可能性を考慮して、エラー発生時の対処方法を決めておくことが重要
- ワークフロー管理ではタスクの実行順序を決めると同時にエラーからどのように回復するかというプランを定める
- 何か問題が起きても速やかに回復できるようなエラーに強いワークフローを構築し、毎日のデータ処理を安定して実行できるように勤める

### リカバリとフローの再実行

- エラーには無数の可能性があるため、ワークフロー管理では基本的にはエラーから自動回復できるものとは考えない
- 代わりに手作業によるリカバリー前提としてタスクを設計する
- 失敗したタスクは全て記録してそれを後から再実行できるようにする

### リトライ

- 何度も発生するエラーについてはなるべくリカバリを自動化する

### バックフィル

- パラメータに含まれる日時を順に変えながら一定期間のフローを連続して実行する仕組みをバックフィルという
- 過去に遡って実行したい場合などに使う

## 冪等な操作としてタスクを実行する

- リカバリーの前提として忘れてはならないのが、再実行の安全性
- タスクは前提として「最後まで成功」するか「失敗しても何も残らない」のどちらかであるべき

### アトミック操作

- 各タスクがシステムに変更を加えるのを一度きりにする

### 冪等な操作

- タスクに与えられたパラメータをうまく使って固有の名前を生成し、何度実行しても常に置き換えが行われるように設計する

### 冪等な追記

- 一般にテーブルから一部のデータだけを削除するのは非効率であり、予期せぬ劣化を招く可能性がある
- テーブルパーティショニングの考え方を導入し、パーティション単位で置き換えを行うようにする
- DBの構成上パーティショニングが不可能な場合はアトミックな追記を検討する

### アトミックな追記

- 一度中間テーブルを作成してから、最後に一度だけ目的のテーブルに追記するのが安全
- 実行途中で問題が起きても、中途半端にデータが書き込まれることがなくなる

## ワークフロー全体を冪等にする

- 全てのフローが冪等になるようになっていれば、安心してワークフローを再実行できる

## タスクキュー

- リソースの消費量をコントロールする
- 全てのタスクは一旦キューに格納され、一定数のワーカープロセスがそれらを順に取り出すことで並列化が実現される

### ボトルネックの解消 

よく発生する問題
- CPU使用率100%
- メモリ不足
- ディスク溢れ
- ディスクI/Oの限界
- ネットワーク帯域の限界
- 通信エラーやタイムアウト

### タスク数の適正化

- 限られた計算リソースを無駄なく活用できるように各タスクが適度な大きさになるように調整する


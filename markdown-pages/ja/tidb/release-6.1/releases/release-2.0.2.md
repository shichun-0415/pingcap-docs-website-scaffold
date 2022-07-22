---
title: TiDB 2.0.2 Release Notes
---

# TiDB2.0.2リリースノート {#tidb-2-0-2-release-notes}

2018年5月21日、TiDB2.0.2がリリースされました。 TiDB 2.0.1と比較して、このリリースではシステムの安定性が大幅に向上しています。

## TiDB {#tidb}

-   10進数の除算式をプッシュダウンする問題を修正します
-   `Delete`ステートメントでの`USE INDEX`構文の使用のサポート
-   `Auto-Increment`の列で`shard_row_id_bits`つの機能を使用することを禁止します
-   Binlogを書き込むためのタイムアウトメカニズムを追加します

## PD {#pd}

-   バランスリーダースケジューラに切断されたノードをフィルタリングさせる
-   転送リーダー演算子のタイムアウトを10秒に変更します
-   クラスタリージョンが異常な状態にあるときにラベルスケジューラがスケジュールしない問題を修正します
-   `evict leader scheduler`の不適切なスケジューリングの問題を修正します

## TiKV {#tikv}

-   Raftログが印刷されない問題を修正します
-   より多くのgRPC関連パラメーターの構成をサポート
-   リーダー選出のタイムアウト範囲の構成をサポート
-   廃止された学習者が削除されない問題を修正します
-   スナップショット中間ファイルが誤って削除される問題を修正します
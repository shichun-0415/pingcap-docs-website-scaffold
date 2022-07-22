---
title: tiup dm patch
---

# tiup dm patch {#tiup-dm-patch}

クラスタの実行中にサービスのバイナリーを動的に置き換える必要がある場合（つまり、置き換え中にクラスタを使用可能な状態に保つため）、 `tiup dm patch`コマンドを使用できます。このコマンドは次のことを行います。

-   交換用のバイナリパッケージをターゲットマシンにアップロードします。
-   APIを使用して関連ノードをオフラインにします。
-   ターゲットサービスを停止します。
-   バイナリパッケージを解凍し、サービスを置き換えます。
-   ターゲットサービスを開始します。

## 構文 {#syntax}

```shell
tiup dm patch <cluster-name> <package-path> [flags]
```

-   `<cluster-name>` ：操作するクラスタの名前。
-   `<package-path>` ：置換に使用されるバイナリパッケージへのパス。

### 準備 {#preparation}

次の手順に従って、このコマンドに必要なバイナリパッケージを事前にパックする必要があります。

-   置き換えるコンポーネントの名前`${component}` （dm-master、dm-worker ...）、コンポーネントの`${version}` （v2.0.0、v2.0.1 ...）、およびオペレーティングシステム`${os}`とプラットフォーム`${arch}`を決定します。コンポーネントが実行します。
-   コマンド`wget https://tiup-mirrors.pingcap.com/${component}-${version}-${os}-${arch}.tar.gz -O /tmp/${component}-${version}-${os}-${arch}.tar.gz`を使用して、現在のコンポーネントパッケージをダウンロードします。
-   `mkdir -p /tmp/package && cd /tmp/package`を実行して、ファイルをパックするための一時ディレクトリを作成します。
-   `tar xf /tmp/${component}-${version}-${os}-${arch}.tar.gz`を実行して、元のバイナリパッケージを解凍します。
-   `find .`を実行して、一時パッケージディレクトリ内のファイル構造を表示します。
-   バイナリファイルまたは構成ファイルを一時ディレクトリの対応する場所にコピーします。
-   `tar czf /tmp/${component}-hotfix-${os}-${arch}.tar.gz *`を実行して、ファイルを一時ディレクトリにパックします。
-   最後に、 `tiup dm patch`コマンドで`<package-path>`の値として`/tmp/${component}-hotfix-${os}-${arch}.tar.gz`を使用できます。

## オプション {#options}

### --overwrite {#overwrite}

-   特定のコンポーネント（dm-workerなど）にパッチを適用した後、tiup-dmがコンポーネントをスケールアウトすると、tiup-dmはデフォルトで元のコンポーネントバージョンを使用します。クラスタが将来スケールアウトするときにパッチを適用するバージョンを使用するには、コマンドでオプション`--overwrite`を指定する必要があります。
-   データ型： `BOOLEAN`
-   このオプションは、デフォルトで`false`の値で無効になっています。このオプションを有効にするには、このオプションをコマンドに追加し、 `true`の値を渡すか、値を渡さないようにします。

### -N、-node {#n-node}

-   置き換えるノードを指定します。このオプションの値は、ノードIDのコンマ区切りのリストです。ノードIDは、 `[tiup dm display](/tiup/tiup-component-dm-display.md)`コマンドによって返されるクラスタステータステーブルの最初の列から取得できます。
-   データ型： `STRING`
-   このオプションが指定されていない場合、TiUPはデフォルトで置き換えるすべてのノードを選択します。

> **ノート：**
>
> オプション`-R, --role`が同時に指定された場合、TiUPは`-N, --node`と`-R, --role`の両方の要件に一致するサービスノードを置き換えます。

### -R、-role {#r-role}

-   置き換える役割を指定します。このオプションの値は、ノードの役割のコンマ区切りのリストです。ノードの役割は、 `[tiup dm display](/tiup/tiup-component-dm-display.md)`コマンドによって返されるクラスタステータステーブルの2番目の列から取得できます。
-   データ型： `STRING`
-   このオプションが指定されていない場合、TiUPはデフォルトで置き換えるすべての役割を選択します。

> **ノート：**
>
> オプション`-N, --node`が同時に指定された場合、TiUPは`-N, --node`と`-R, --role`の両方の要件に一致するサービスノードを置き換えます。

### &#x20;--offline {#offline}

-   現在のクラスタがオフラインであることを宣言します。このオプションを指定すると、 TiUP DMは、サービスを再起動せずに、クラスタコンポーネントのバイナリファイルのみを置き換えます。

### -h、-help {#h-help}

-   ヘルプ情報を印刷します。
-   データ型： `BOOLEAN`
-   このオプションは、デフォルトで`false`の値で無効になっています。このオプションを有効にするには、このオプションをコマンドに追加し、 `true`の値を渡すか、値を渡さないようにします。

## 出力 {#outputs}

tiup-dmの実行ログ。

[&lt;&lt;前のページに戻るTiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)
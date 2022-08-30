---
title: TiDB Computing
summary: Understand the computing layer of the TiDB database.
---

# TiDB コンピューティング {#tidb-computing}

TiKV が提供する分散ストレージに基づいて、TiDB はトランザクション処理の優れた機能とデータ分析の機能を組み合わせたコンピューティング エンジンを構築します。このドキュメントでは、まず TiDB データベース テーブルのデータを TiKV の (キー、値) キーと値のペアにマップするデータ マッピング アルゴリズムを紹介し、次に TiDB がメタデータを管理する方法を紹介し、最後にTiDB SQLレイヤーのアーキテクチャを示します。

コンピューティング レイヤーが依存するストレージ ソリューションについては、このドキュメントでは TiKV の行ベースのストレージ構造のみを紹介します。 OLAP サービスの場合、TiDB は列ベースのストレージ ソリューション[ティフラッシュ](/tiflash/tiflash-overview.md)を TiKV 拡張機能として導入します。

## テーブル データを Key-Value にマッピングする {#mapping-table-data-to-key-value}

このセクションでは、データを TiDB の (キー、値) キーと値のペアにマッピングするためのスキームについて説明します。ここでマッピングされるデータには、次の 2 つのタイプがあります。

-   表の各行のデータ。以下、表データと呼びます。
-   テーブル内のすべてのインデックスのデータ。以下、インデックス データと呼びます。

### テーブル データの Key-Value へのマッピング {#mapping-of-table-data-to-key-value}

リレーショナル データベースでは、テーブルに多数の列が含まれる場合があります。行の各列のデータを (キー、値) キーと値のペアにマップするには、キーの作成方法を検討する必要があります。まず、OLTP シナリオでは、1 行または複数行のデータの追加、削除、変更、検索などの多くの操作があり、データベースがデータ行をすばやく読み取る必要があります。したがって、各キーには一意の ID (明示的または暗黙的) を付けて、すばやく見つけられるようにする必要があります。次に、多くの OLAP クエリでフル テーブル スキャンが必要になります。テーブル内のすべての行のキーを範囲にエンコードできれば、テーブル全体を範囲クエリで効率的にスキャンできます。

上記の考慮事項に基づいて、TiDB でのテーブル データの Key-Value へのマッピングは、次のように設計されています。

-   簡単に検索できるように同じテーブルのデータをまとめておくために、TiDB は`TableID`で表される各テーブルにテーブル ID を割り当てます。テーブル ID は、クラスタ全体で一意の整数です。
-   TiDB は、 `RowID`で表される行 ID を、テーブル内のデータの各行に割り当てます。行 ID も整数で、テーブル内で一意です。行 ID について、TiDB は小さな最適化を行いました。テーブルに整数型の主キーがある場合、TiDB はこの主キーの値を行 ID として使用します。

データの各行は、次の規則に従って (キー、値) キーと値のペアとしてエンコードされます。

```
Key:   tablePrefix{TableID}_recordPrefixSep{RowID}
Value: [col1, col2, col3, col4]
```

`tablePrefix`と`recordPrefixSep`はどちらも、キー スペース内の他のデータを区別するために使用される特別な文字列定数です。文字列定数の正確な値は[マッピング関係のまとめ](#summary-of-mapping-relationships)で紹介されています。

### インデックス付きデータの Key-Value へのマッピング {#mapping-of-indexed-data-to-key-value}

TiDB は、プライマリ キーとセカンダリ インデックスの両方 (一意のインデックスと非一意のインデックスの両方) をサポートします。テーブル データ マッピング スキームと同様に、TiDB は`IndexID`で表されるテーブルの各インデックスにインデックス ID を割り当てます。

主キーと一意のインデックスの場合、キーと値のペアに基づいて対応する`RowID`をすばやく見つける必要があるため、そのようなキーと値のペアは次のようにエンコードされます。

```
Key:   tablePrefix{tableID}_indexPrefixSep{indexID}_indexedColumnsValue
Value: RowID
```

一意性制約を満たす必要のない通常のセカンダリ インデックスの場合、1 つのキーが複数の行に対応する場合があります。キーの範囲に従って、対応する`RowID`を照会する必要があります。したがって、キーと値のペアは次の規則に従ってエンコードする必要があります。

```
Key:   tablePrefix{TableID}_indexPrefixSep{IndexID}_indexedColumnsValue_{RowID}
Value: null
```

### マッピング関係のまとめ {#summary-of-mapping-relationships}

上記のすべてのエンコーディング ルールの`tablePrefix` 、 `recordPrefixSep` 、および`indexPrefixSep`は、次のように定義されているキー スペース内の他のデータから KV を区別するために使用される文字列定数です。

```
tablePrefix     = []byte{'t'}
recordPrefixSep = []byte{'r'}
indexPrefixSep  = []byte{'i'}
```

また、上記のコード化スキームでは、テーブル データまたはインデックス データ キーのコード化スキームに関係なく、テーブル内のすべての行に同じキー プレフィックスがあり、インデックスのすべてのデータにも同じプレフィックスがあることに注意してください。したがって、同じプレフィックスを持つデータは、TiKV のキー スペースにまとめて配置されます。したがって、サフィックス部分のエンコード方式を慎重に設計して、エンコード前とエンコード後の比較が同じになるようにすることで、テーブル データまたはインデックス データを順番に TiKV に格納できます。このコード化スキームを使用すると、テーブル内のすべての行データが TiKV のキー空間で`RowID`ずつ順番に配置され、特定のインデックスのデータも、インデックス データの特定の値に従ってキー空間で順番に配置されます ( `indexedColumnsValue` )。

### Key-Value マッピング関係の例 {#example-of-key-value-mapping-relationship}

このセクションでは、TiDB の Key-Value マッピング関係を理解するための簡単な例を示します。次のテーブルが TiDB に存在するとします。

```sql
CREATE TABLE User (
     ID int,
     Name varchar(20),
     Role varchar(20),
     Age int,
     PRIMARY KEY (ID),
     KEY idxAge (Age)
);
```

テーブルに 3 行のデータがあるとします。

```
1, "TiDB", "SQL Layer", 10
2, "TiKV", "KV Engine", 20
3, "PD", "Manager", 30
```

データの各行は (キー、値) キーと値のペアにマップされ、テーブルには`int`型の主キーがあるため、値`RowID`はこの主キーの値です。テーブルの`TableID`が`10`で、TiKV に格納されているテーブル データが次のとおりであるとします。

```
t10_r1 --> ["TiDB", "SQL  Layer", 10]
t10_r2 --> ["TiKV", "KV  Engine", 20]
t10_r3 --> ["PD", " Manager", 30]
```

主キーに加えて、テーブルには一意でない通常のセカンダリ インデックス`idxAge`があります。 `IndexID`が`1`で、TiKV に保存されているそのインデックス データが次のとおりであるとします。

```
t10_i1_10_1 --> null
t10_i1_20_2 --> null
t10_i1_30_3 --> null
```

上記の例は、リレーショナル モデルから TiDB の Key-Value モデルへのマッピング ルールと、このマッピング スキームの背後にある考慮事項を示しています。

## メタデータ管理 {#metadata-management}

TiDB の各データベースとテーブルには、その定義とさまざまな属性を示すメタデータがあります。この情報も永続化する必要があり、TiDB はこの情報を TiKV にも保存します。

各データベースまたはテーブルには、一意の ID が割り当てられます。一意の識別子として、テーブル データが Key-Value にエンコードされる場合、この ID は`m_`プレフィックスを使用して Key にエンコードされます。これにより、シリアル化されたメタデータが格納されたキーと値のペアが構築されます。

さらに、TiDB は専用の (キー、値) キーと値のペアも使用して、すべてのテーブルの構造情報の最新バージョン番号を格納します。このキーと値のペアはグローバルであり、DDL 操作の状態が変化するたびにバージョン番号が`1`ずつ増えます。 TiDB はこのキーと値のペアを`/tidb/ddl/global_schema_version`のキーで PD サーバーに永続的に格納し、Value は`int64`型のバージョン番号の値です。一方、TiDB はスキーマの変更をオンラインで適用するため、PD サーバーに格納されているテーブル構造情報のバージョン番号が変更されていないかどうかを常にチェックするバックグラウンド スレッドを保持します。このスレッドは、一定期間内にバージョンの変更を取得できることも保証します。

## SQL レイヤーの概要 {#sql-layer-overview}

TiDB の SQL レイヤーである TiDB サーバーは、SQL ステートメントを Key-Value 操作に変換し、その操作を分散 Key-Value ストレージ レイヤーである TiKV に転送し、TiKV によって返された結果を組み立て、最後にクエリ結果をクライアントに返します。

この層のノードはステートレスです。これらのノード自体はデータを保存せず、完全に同等です。

### SQL コンピューティング {#sql-computing}

SQL コンピューティングの最も簡単なソリューションは、前のセクションで説明した[テーブル データの Key-Value へのマッピング](#mapping-of-table-data-to-key-value)です。これは、SQL クエリを KV クエリにマップし、KV インターフェイスを介して対応するデータを取得し、さまざまな計算を実行します。

たとえば、 `select count(*) from user where name = "TiDB"` SQL ステートメントを実行するには、TiDB はテーブル内のすべてのデータを読み取る必要があり、次に`name`フィールドが`TiDB`であるかどうかをチェックし、そうであれば、この行を返します。プロセスは次のとおりです。

1.  キー範囲を構築します。テーブル内の`RowID`はすべて`[0,  MaxInt64)`範囲にあります。行データ`Key`のエンコード規則に従って、 `0`と`MaxInt64`を使用すると、左が閉じて右が開いた`[StartKey, EndKey)`の範囲を構築できます。
2.  キー範囲のスキャン: 上記で作成したキー範囲に従って、TiKV のデータを読み取ります。
3.  データのフィルター処理: 読み取られたデータの各行について、 `name = "TiDB"`の式を計算します。結果が`true`の場合、この行に戻ります。そうでない場合は、この行をスキップしてください。
4.  `Count(*)`を計算します。要件を満たす行ごとに、合計して`Count(*)`の結果になります。

**全体のプロセスを以下に示します。**

![naive sql flow](https://download.pingcap.com/images/docs/tidb-computing-native-sql-flow.jpeg)

このソリューションは直感的で実行可能ですが、分散データベースのシナリオでは明らかな問題がいくつかあります。

-   データがスキャンされている間、各行は少なくとも 1 つの RPC オーバーヘッドを持つ KV 操作を介して TiKV から読み取られます。スキャンするデータが大量にある場合、これは非常に高くなる可能性があります。
-   すべての行に適用できるわけではありません。条件を満たさないデータは読み込む必要はありません。
-   このクエリの返された結果から、要件に一致する行の数だけが必要であり、それらの行の値は必要ありません。

### 分散 SQL 操作 {#distributed-sql-operations}

上記の問題を解決するには、大量の RPC 呼び出しを回避するために、計算をできるだけストレージ ノードに近づける必要があります。まず、SQL 述語条件`name = "TiDB"`を計算のためにストレージ ノードにプッシュ ダウンする必要があります。これにより、有効な行のみが返され、無意味なネットワーク転送が回避されます。次に、集計関数`Count(*)`を事前集計のためにストレージ ノードにプッシュ ダウンすることもでき、各ノードは`Count(*)`の結果を返すだけで済みます。 SQL レイヤーは、各ノードから返された`Count(*)`の結果を合計します。

次の図は、データがレイヤーごとに返される方法を示しています。

![dist sql flow](https://download.pingcap.com/images/docs/tidb-computing-dist-sql-flow.png)

### SQL 層のアーキテクチャ {#architecture-of-sql-layer}

前のセクションでは、SQL 層のいくつかの関数を紹介しました。SQL ステートメントがどのように処理されるかについての基本的な理解が得られたことを願っています。実際、TiDB の SQL レイヤーは、多くのモジュールとレイヤーで構成されており、はるかに複雑です。次の図は、重要なモジュールと呼び出し関係を示しています。

![tidb sql layer](https://download.pingcap.com/images/docs/tidb-computing-tidb-sql-layer.png)

ユーザーの SQL リクエストは、直接または`Load Balancer`経由で TiDB サーバーに送信されます。 TiDB サーバーは`MySQL Protocol Packet`を解析し、リクエストのコンテンツを取得し、SQL リクエストを構文的および意味的に解析し、クエリ プランを開発して最適化し、クエリ プランを実行し、データを取得して処理します。すべてのデータは TiKVクラスタに保存されるため、このプロセスでは、TiDB サーバーは TiKV と対話してデータを取得する必要があります。最後に、TiDB サーバーはクエリ結果をユーザーに返す必要があります。
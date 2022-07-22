---
title: Manage User Access
summary: Learn how to manage the user access in your TiDB Cloud cluster.
---

# ユーザーアクセスの管理 {#manage-user-access}

このドキュメントでは、 TiDB Cloudクラスタでユーザーアクセスを管理する方法について説明します。

## ログイン {#sign-in}

1.  TiDB Cloudログインページに移動します： [https://tidbcloud.com](https://tidbcloud.com) 。

2.  TiDB Cloudのサインアップ方法に応じて、次のいずれかを実行します。

    -   Googleアカウントでサインアップした場合は、[Google**でサインイン**]をクリックします。
    -   GitHubアカウントでサインアップした場合は、[GitHub**でサインイン**]をクリックします。
    -   メールアドレスとパスワードでサインアップした場合は、メールアドレスとパスワードを入力し、[**サインイン**]をクリックします。

## サインアウト {#sign-out}

TiDB Cloudにサインインした後、サインアウトする必要がある場合は、次の手順を実行します。

1.  ウィンドウの右上にあるアカウント名をクリックします。

2.  [**ログアウト]**をクリックします。

## ユーザーパスワードを管理する {#manage-user-passwords}

> **ノート：**
>
> このセクションの内容は、電子メールとパスワードを使用したTiDB Cloud登録にのみ適用されます。 GoogleまたはGitHubでTiDB Cloudにサインアップする場合、パスワードはGoogleまたはGitHubによって管理され、 TiDB Cloudコンソールを使用してパスワードを変更することはできません。

システムのセキュリティを向上させるために、電子メールとパスワードを使用してTiDB Cloudにサインアップする場合は、90日ごとにパスワードをリセットすることをお勧めします。

パスワードを変更するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**アカウント]**をクリックします。

3.  [**パスワードの変更**]タブをクリックします。

4.  [**パスワードの変更]**をクリックし、メールボックスでパスワードをリセットするためのリンクをオンにします。

パスワードが90日以内に変更されない場合、 TiDB Cloudにログインするときにパスワードをリセットするように求めるプロンプトが表示されます。プロンプトに従ってパスワードをリセットすることをお勧めします。

## ユーザープロファイルを管理する {#manage-user-profiles}

TiDB Cloudでは、名、前回、会社名、国、電話番号などのプロファイルを簡単に管理できます。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**アカウント]**をクリックします。デフォルトでは、[<strong>プロファイル</strong>]タブが選択されています。

3.  プロファイル情報を更新し、[**保存**]をクリックします。

## 組織とプロジェクトをビューする {#view-the-organization-and-project}

TiDB Cloudは、組織とプロジェクトに基づく階層構造を提供して、TiDBクラスタの管理を容易にします。組織とプロジェクトの階層では、組織には複数のプロジェクトと組織メンバーを含めることができ、プロジェクトには複数のクラスターとプロジェクトメンバーを含めることができます。

この構造の下で：

-   請求は、各プロジェクトおよびクラスタでの使用状況の可視性を維持しながら、組織レベルで行われます。

-   組織内のすべてのメンバーを表示できます。

-   プロジェクトのすべてのメンバーを表示することもできます。

組織内のプロジェクトのクラスタにアクセスするには、ユーザーは組織のメンバーとプロジェクトのメンバーの両方である必要があります。組織の所有者は、ユーザーをプロジェクトに招待して、プロジェクト内のクラスターを作成および管理できます。

所属するプロジェクトを確認するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。
2.  [**組織の設定]**をクリックします。デフォルトでは、[<strong>プロジェクト</strong>]タブが表示されます。

## 組織のメンバーを招待する {#invite-an-organization-member}

組織の所有者である場合は、組織のメンバーを招待できます。それ以外の場合は、このセクションをスキップしてください。

メンバーを組織に招待するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。

3.  [**ユーザー管理]**をクリックし、[<strong>すべてのユーザー別</strong>]タブを選択します。

4.  [**招待]を**クリックします。

5.  招待するユーザーのメールアドレスを入力し、ユーザーの役割を選択してから、ユーザーのプロジェクトを選択します。

    > **ヒント：**
    >
    > 一度に複数のメンバーを招待したい場合は、複数のメールアドレスを入力できます。

6.  [**確認]**をクリックします。次に、新しいユーザーがユーザーリストに正常に追加されます。同時に、確認リンクが記載された電子メールが招待された電子メールアドレスに送信されます。

7.  このメールを受信した後、ユーザーはメール内のリンクをクリックして本人確認を行う必要があり、新しいページが表示されます。

8.  新しいページで、ユーザーはライセンスを表示して同意する必要があります。次に、[**送信**]をクリックしてTiDB Cloudにアカウントを作成します。その後、ユーザーはログインページにリダイレクトされます。

> **ノート：**
>
> メールの確認リンクは24時間で期限切れになります。ユーザーがメールを受信しない場合は、[**再送信**]をクリックします。

## プロジェクトメンバーを招待する {#invite-a-project-member}

組織の所有者であれば、プロジェクトメンバーを招待できます。それ以外の場合は、このセクションをスキップしてください。

メンバーをプロジェクトに招待するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。

3.  [**ユーザー管理]**をクリックし、[<strong>プロジェクト</strong>別]タブを選択します。

4.  [**招待]を**クリックします。

5.  招待するユーザーのメールアドレスを入力し、ユーザーの役割を選択してから、ユーザーのプロジェクトを選択します。

    > **ヒント：**
    >
    > 一度に複数のメンバーを招待したい場合は、複数のメールアドレスを入力できます。

6.  [**確認]**をクリックします。次に、新しいユーザーがユーザーリストに正常に追加されます。同時に、確認リンクが記載された電子メールが招待された電子メールアドレスに送信されます。

7.  このメールを受信した後、ユーザーはメール内のリンクをクリックして本人確認を行う必要があり、新しいページが表示されます。

8.  新しいページで、ユーザーはライセンスを表示して同意する必要があります。次に、[**送信**]をクリックしてTiDB Cloudにアカウントを作成します。その後、ユーザーはログインページにリダイレクトされます。

> **ノート：**
>
> メールの確認リンクは24時間で期限切れになります。ユーザーがメールを受信しない場合は、[**再送信**]をクリックします。

## メンバーの役割を構成する {#configure-member-roles}

組織の所有者である場合は、次の手順を実行して、組織メンバーの役割を構成できます。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。
2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。
3.  [**ユーザー管理]**をクリックし、[<strong>すべてのユーザー別</strong>]タブを選択します。
4.  ターゲットメンバーの役割をクリックしてから、役割を変更します。

組織には4つの役割があります。各役割の権限は次のとおりです。

-   オーナー：
    -   メンバーを組織に招待し、組織からメンバーを削除します
    -   組織メンバーの役割を構成する
    -   プロジェクトの作成と名前の変更
    -   メンバーをプロジェクトに招待し、プロジェクトからメンバーを削除します
    -   タイムゾーンを編集する
    -   請求書のビューと支払い情報の編集
-   メンバー：
    -   プロジェクトに招待して、プロジェクトインスタンスの管理権限を取得できます
-   請求管理者：
    -   請求書のビューと支払い情報の編集
    -   プロジェクトに招待して、プロジェクトインスタンスの管理権限を取得できます
-   監査管理者：
    -   監査ログのビューと構成
    -   プロジェクトに招待して、プロジェクトインスタンスの管理権限を取得できます

## 組織のメンバーを削除する {#remove-an-organization-member}

組織の所有者である場合は、組織のメンバーを削除できます。それ以外の場合は、このセクションをスキップしてください。

組織からメンバーを削除するには、次の手順を実行します。

> **ノート：**
>
> メンバーが組織から削除されると、そのメンバーは所属するプロジェクトからも削除されます。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。

3.  [**すべてのユーザーによる]を**クリックします。

4.  削除するユーザー行の[**削除]**をクリックします。

## プロジェクトメンバーを削除する {#remove-a-project-member}

組織の所有者である場合は、プロジェクトメンバーを削除できます。それ以外の場合は、このセクションをスキップしてください。

プロジェクトからメンバーを削除するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。

3.  [**プロジェクト**別]をクリックします。

4.  削除するユーザー行の[**削除]**をクリックします。

## ローカルタイムゾーンを設定する {#set-the-local-time-zone}

組織の所有者である場合は、タイムゾーンに応じてシステムの表示時間を変更できます。

ローカルタイムゾーン設定を変更するには、次の手順を実行します。

1.  TiDB Cloudコンソールの右上隅にあるアカウント名をクリックします。

2.  [**組織の設定]**をクリックします。組織設定ページが表示されます。

3.  [**タイムゾーン]**をクリックします。

4.  ドロップダウンリストをクリックして、タイムゾーンを選択します。

5.  [**確認]**をクリックします。
---
title: "Microsoft Graph APIを使ってMicrosoft 365のライセンスを付与する"
emoji: "🍔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nodejs", "slack", "api", "azure"]
published: false
---
# はじめに
Slack上で申請承認が完了すると自動的にSaaSのライセンスを付与するBolt.js製のSlack Appを社内で運用しているのですが、そこにMicrosoft 365のライセンスを付与する処理を組み込んだ際のメモです。

# Microsoft Graphとは
Microsoft Graphは、Microsoft 365、Windows 10、およびEnterprise Mobility + Securityをプログラマブルに操作するためのプラットフォームです。
今回はMicrosoft Graphが提供するAPIを使用してAzure AD上のユーザーにライセンスを付与します。
> [Microsoft Graph の概要 - Microsoft Graph | Microsoft Docs](https://docs.microsoft.com/ja-jp/graph/overview)
> ![](https://storage.googleapis.com/zenn-user-upload/9d24d0d93098d6d6b5bfe462.png)

# Microsoft Azure portalからアプリを登録する
通常IT担当者などの人間がライセンスを付与する際にはまずAzure Active Directory（以下 Azure AD）での認証が完了しており、ライセンスを付与できる権限が認可されていることが前提となります。

今回は人間に代わってアプリにライセンスを付与してもらうため、その権限をアプリに持たせないといけません。
だれでも実行できては困るのでそのアプリは本当に正規のアプリなのか、権限を認可して良いかを認証する必要があります。

この認証情報を発行するためにAzure AD上にアプリを登録します。

Azure ADの管理画面で `アプリの登録` > `新規登録` を選択します
![](https://storage.googleapis.com/zenn-user-upload/3bee6caa49449098dd6d195b.png)

適当な名前を付けてアプリの公開範囲を選択し、登録を選択します
![](https://storage.googleapis.com/zenn-user-upload/10f555f65bcc0972da93a9a3.png)

次に認証情報であるクライアントシークレットを生成するため、`証明書とシークレット` の画面で `新しいクライアントシークレット` を選択します。
![](https://storage.googleapis.com/zenn-user-upload/14f6db09a52c7e2821242cd3.png)

適当な説明を入力して有効期限を選択し `追加` を選択します
![](https://storage.googleapis.com/zenn-user-upload/5aadd7f660e1fa5d46a6b1a1.png)

:::message
**シークレットの期限は24か月以上にできる？**
従来シークレットの有効期限は無制限にもできましたが2021年4月よりセキュリティ懸念があるということで最長でも24か月へ変更になり、前述の画像通りGUIからだと24か月までしか選択できません。
しかし2021年7月現在、PowerShellから発行する場合は無制限にできることを確認しています（ただし当然リスクを伴う選択肢であることにご注意ください）。

[クライアント シークレット作成画面の変更について | Japan Azure Identity Support Blog](https://jpazureid.github.io/blog/azure-active-directory/azuread-clientsecrets-202104/#PowerShell-%E3%81%8B%E3%82%89%E3%81%AF%E3%80%81%E6%9C%89%E5%8A%B9%E6%9C%9F%E9%99%90%E3%81%8C-2-%E5%B9%B4%E4%BB%A5%E4%B8%8A%E3%81%AE%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88-%E3%82%B7%E3%83%BC%E3%82%AF%E3%83%AC%E3%83%83%E3%83%88%E3%81%8C%E4%BD%9C%E6%88%90%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%A7%E3%81%99%E3%81%8C%E2%80%A6)
> 現在は PowerShell コマンドでクライアントシークレットを作成する際は、2 年以上の有効期限を指定することが可能です。
> （中略）
> ただし、いずれは PowerShell コマンドによるクライアントシークレット作成にも、Azure ポータル上と同じ制限がかかるようになる方針です (制限が反映された際には、この項目は削除する予定です)。この制限が実施されるスケジュールについては、現時点では未定です。
:::


---
title: "Microsoft Graph API経由でMicrosoft 365のライセンスを付与する"
emoji: "🍔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["api", "azure", "microsoft365"]
published: false
---
# はじめに
Microsoft Graphは、Microsoft 365、Windows 10、Enterprise Mobility + Security等をプログラマブルに操作するためのプラットフォームです。
https://docs.microsoft.com/ja-jp/graph/overview

先日このMicrosoft GraphのAPIを使用してSlack AppからMicrosoft 365 Apps for Businessのライセンスを付与するツールを作成したのですが、その際にAzure AD側で必要となったステップについてメモとして残しておこうと思います。
（※ Slack App側の実装については本記事では割愛します）

# Azure ADのアクセストークンを取得する
IT担当者がAzure ADにサインインして操作するのと違って、今回はアプリがライセンスを付与するため、OAuth 2.0クライアント資格情報の付与フローを使用してAzure ADからアクセストークンを取得します。

## Microsoft Azure portalからアプリを登録する
Azure ADの管理画面で `アプリの登録` > `新規登録` を選択します
![](https://storage.googleapis.com/zenn-user-upload/3bee6caa49449098dd6d195b.png)

適当な名前を付けてアプリの公開範囲を選択し、登録を選択します
![](https://storage.googleapis.com/zenn-user-upload/10f555f65bcc0972da93a9a3.png)

## シークレットの生成
次に認証に用いるクレデンシャルを生成するため、`証明書とシークレット` の画面で `新しいクライアントシークレット` を選択します。
![](https://storage.googleapis.com/zenn-user-upload/14f6db09a52c7e2821242cd3.png)

適当な説明を入力して有効期限を選択し `追加` を選択します
![](https://storage.googleapis.com/zenn-user-upload/5aadd7f660e1fa5d46a6b1a1.png)

シークレットキーの値とシークレットIDが表示されるので、コピーしてパスワード管理ツールなど適切な手段で管理します。
![](https://storage.googleapis.com/zenn-user-upload/2c2ea1061480ebd0aad224e0.png)

:::message
**シークレットの期限は24か月以上にできる？**
従来シークレットの有効期限は無制限にもできましたが2021年4月よりセキュリティ懸念があるということで最長でも24か月へ変更になり、前述の画像通りGUIからだと24か月までしか選択できません。
2021年7月現在、PowerShellから発行する場合は従来どおり無制限にできることを確認していますがこれはGUI同様の制限が入る予定となっています。

[クライアント シークレット作成画面の変更について | Japan Azure Identity Support Blog](https://jpazureid.github.io/blog/azure-active-directory/azuread-clientsecrets-202104/#PowerShell-%E3%81%8B%E3%82%89%E3%81%AF%E3%80%81%E6%9C%89%E5%8A%B9%E6%9C%9F%E9%99%90%E3%81%8C-2-%E5%B9%B4%E4%BB%A5%E4%B8%8A%E3%81%AE%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88-%E3%82%B7%E3%83%BC%E3%82%AF%E3%83%AC%E3%83%83%E3%83%88%E3%81%8C%E4%BD%9C%E6%88%90%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%A7%E3%81%99%E3%81%8C%E2%80%A6)
> 現在は PowerShell コマンドでクライアントシークレットを作成する際は、2 年以上の有効期限を指定することが可能です。
> （中略）
> ただし、いずれは PowerShell コマンドによるクライアントシークレット作成にも、Azure ポータル上と同じ制限がかかるようになる方針です (制限が反映された際には、この項目は削除する予定です)。この制限が実施されるスケジュールについては、現時点では未定です。
:::

## アクセス許可の付与
続いて、作成したアプリに必要な権限を付与します。
今回使用する `assignLicense` というAPIエンドポイントについて、Microsoftのドキュメントで必要なアクセス許可を確認します。

> [assignLicense - Microsoft Graph v1.0 | Microsoft Docs](https://docs.microsoft.com/ja-jp/graph/api/user-assignlicense?view=graph-rest-1.0&tabs=http#permissions)
> |アクセス許可の種類 | アクセス許可 (特権の小さいものから大きいものへ)|
> |---|---|
> |委任 (職場または学校のアカウント)|User.ReadWrite.All、Directory.ReadWrite.All|
> |委任 (個人用 Microsoft アカウント)|サポートされていません。|
> |アプリケーション|User.ReadWrite.All、Directory.ReadWrite.All|


実際にアクセス許可を追加するため、 `API のアクセス許可` > `アクセス許可の追加` を選択します
![](https://storage.googleapis.com/zenn-user-upload/170697b1943b3fea86dee64f.png)

`Microsoft Graph` を選択
![](https://storage.googleapis.com/zenn-user-upload/f7d02cdf9dfc2dcf5dac6547.png)

`アプリケーションの許可` を選択
![](https://storage.googleapis.com/zenn-user-upload/851df6cfe7b48082f650a948.png)

先程確認したアクセス許可である `User.ReadWrite.All` と `Directory.ReadWrite.All` を検索して追加します
![](https://storage.googleapis.com/zenn-user-upload/cd25a0acf5d2fe91c565cacb.png)

アプリケーションを実行する中で管理者のサインインを発生させたくない場合はあらかじめ管理者の同意を与えておきます。ここは要件に応じて選択します。
![](https://storage.googleapis.com/zenn-user-upload/6d4d9575d589346a0fc1bc54.png)

これでアクセストークンを取得する準備が整いました。

## 実際にアクセストークンを取得してみる
実際にライセンス付与のAPIを実行する際の認証にはアクセストークンが必要となるので、前述の手順で生成したシークレットIDとシークレットキーを用いてアクセストークンを取得します。

下記はcurlで取得する際のサンプルです。
```shell:bash
curl -d "client_id=$MSGRAPH_CLIENT_ID" \
 -d "scope=https%3A%2F%2Fgraph.microsoft.com%2F.default" \
 -d client_secret=$MSGRAPH_CLIENT_SECRET \
 -d "grant_type=client_credentials" \
 -H "Content-Type: application/x-www-form-urlencoded" \
 -X POST https://login.microsoftonline.com/$MY_AZUREAD_DOMAIN/oauth2/v2.0/token
 ```

上記を実行した際の応答の例です。

```JSON:JSON
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "ext_expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6Imd5b21CUVNlUGJPTjI5a..."
}
```

# API経由でライセンスを付与する
ここまででアクセストークンを取得できたのでいよいよライセンスを付与します。
実際に使用するAPIエンドポイントは `assignLicense` です。
https://docs.microsoft.com/ja-jp/graph/api/user-assignlicense?view=graph-rest-1.0&tabs=http

サンプルとしてcurlから特定のユーザーにMicrosoft 365 Apps for Businessのライセンスを付与する場合は下記のような記述になります。
なお `{user_id}` の部分はUPNで記載可能です。

```shell:bash
curl --request POST https://graph.microsoft.com/v1.0/users/{user_id}/assignLicense \
 -H "Authorization: Bearer $MSGRAPHTOKEN" \
 -H "Content-Type: application/json" \
 -d '{
    "addLicenses": [
        {
            "disabledPlans": [],
            "skuId": "cdd28e44-67e3-425e-be4c-737fab2899d3"
        }
    ],
    "removeLicenses": []
 }'
```

ただ前述のDocsだと分かりにくい部分がいくつかあったので補足します。

## removeLicenses パラメーターの扱いについて
このAPIはライセンスの解除もできるようになっており、Docsの例では付与と解除を同時に実行しています。
解除の必要がない場合は前述のサンプルのように単純に `removeLicenses` の値をブランクにすればOKです。

## skuIdの探し方
パラメーター `addLicenses` の中で `skuId` というプロパティがありますが、ここで何のライセンスを付与するかの種類を指定しています。

このIDを特定する方法は複数あり、まずひとつはGraphエクスプローラー等で `https://graph.microsoft.com/v1.0/subscribedSkus` のAPIエンドポイントを参照することです。（[試してみる場合はここをクリックしてGraph エクスプローラーを起動](https://developer.microsoft.com/ja-jp/graph/graph-explorer?request=subscribedSkus&method=GET&version=v1.0&GraphUrl=https://graph.microsoft.com)）
こちらの方法の特長として自社テナントで保有しているライセンスおよびその購入数と割当数が確認できます。
https://docs.microsoft.com/ja-jp/graph/api/subscribedsku-list?view=graph-rest-1.0&tabs=http

例えばMicrosoft 365 Apps for Businessを1000ライセンス保有しており950ライセンス割り当てしていると下記のような応答になります。
```JSON:JSON
{
    "@odata.id": "https://graph.microsoft.com/v1.0/$metadata#subscribedSkus",
    "capabilityStatus": "Enabled",
    "consumedUnits": 950,
    "id": "0f191825-bf21-445d-a25c-a2ad7ea94105_cdd28e44-67e3-425e-be4c-737fab2899d3",
    "skuId": "cdd28e44-67e3-425e-be4c-737fab2899d3",
    "skuPartNumber": "O365_BUSINESS",
    "appliesTo": "User",
    "prepaidUnits": {
        "enabled": 1000,
        "suspended": 0,
        "warning": 0
    },
```

もうひとつもっとお手軽な方法として、製品とskuIdのリストが記載されているページがあるのでこちらでライセンス名で検索することでも確認できます。
https://docs.microsoft.com/ja-jp/azure/active-directory/enterprise-users/licensing-service-plan-reference

# おわりに
このAPIを活用してチャットツールやワークフロー、申請フォームなどを起点に自動的にライセンス付与するように組み込めば「IT担当者が対応しなくても上長が承認すれば自動的にライセンスが付与される」といった仕組みが実現できると思います。
自動化って楽しいですね。
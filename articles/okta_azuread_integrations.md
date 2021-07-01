---
title: "OktaをIdPとしてAzure ADとのフェデレーションを構成する"
emoji: "⚔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["okta", "azuread"]
published: false
---
# はじめに
OktaをIdPとしてAzure AD(Microsoft 365)とフェデレーションを構成する際、[Okta公式のドキュメント](https://help.okta.com/en/prod/Content/Topics/Apps/Office365-Deployment/configure-sso.htm)通りだとうまくいかない部分がいくつかあったのでメモです。

# 前提
- Oktaをメインディレクトリとする
- オンプレミスのActive Directoryは無し
- Windows PCはAzure AD Join + Intuneにて管理運用している

なおOktaとAzure AD間でフェデレーションを構成するにはAzure ADに追加しているカスタムドメインごとにOkta側のアプリケーション設定をする必要があります。
また今回はOkta側をメインディレクトリとするためAzure AD側の `{yourtenant}.onmicrosoft.com` のほうはフェデレーションの対象外となります。


# 設定手順
## Azure AD側でやること
- カスタムドメインをAzure ADに追加する（まだ追加してなければ）
  - [カスタム ドメインを追加する - Azure Active Directory | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/add-custom-domain)
- プライマリドメインを `{yourtenant}.onmicrosoft.com` のほうにする
  - `Azure Active Directory > カスタム ドメイン名` の画面で確認できます
  - ![](2021-06-17-23-43-21.png)
  - プライマリになってなかったら `{yourtenant}.onmicrosoft.com` をクリックして `プライマリにする` を選択します
- プロビジョニングに使用するためのグローバル管理者アカウントを作成する（プロビジョニングを利用しない場合は不要）
  - フェデレーションするカスタムドメインではなく `{yourtenant}.onmicrosoft.com` のほうのドメインでユーザーを作成し、グローバル管理者のロールを割り当てておきます
  - 作成したユーザーのMFAが有効になっているとフェデレーションが失敗するのでMFAを無効化しておきます（参考：[Office 365 WS-Fed Integration issue](https://support.okta.com/help/s/question/0D51Y00008QoNQN/office-365-wsfed-integration-issue?language=en_US) ）

    - 全体のMFA設定をOFFにするには `Azure Active Directory > プロパティ > セキュリティの既定値群の管理` にて `セキュリティの既定値群の有効化` を `いいえ` にします
    ![](2021-06-17-23-53-37.png)
        - 無効化するとどんな影響があるのかが気になる方はこちらを参照してください
          > [Azure Active Directory のセキュリティ デフォルト | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/concept-fundamentals-security-defaults)
    - ユーザーごとのMFAについては `Azure Active Directory > ユーザー > ユーザーごとの MFA` にて有効になっているユーザーが居ないか確認してください

## Okta側でやること
`Applications > Browse App Catalog > Microsoft Office 365 > Add` を選択します
![](2021-07-01-23-51-26.png)

General Settings の設定を行います
- Application label
  - 管理用のアプリ名称を設定します
- Microsoft Tenant Name
  - 今回は `thdyjp.onmicrosoft.jp` というテナントを使用するのでここには `thdyjp` と入力します
- Display the following links
  - Oktaダッシュボードに表示するアプリアイコンを選択します
- Seats（Optional）
  - ここに購入済みライセンス数を入れておくと使用率のレポートが出せます
- 

# アカウントプロビジョニングを有効化する


# アカウント発行からAzure AD Joinまでの流れ




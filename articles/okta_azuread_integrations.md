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

なおOktaとAzure AD間でフェデレーションを構成するにはドメインごとにOkta側のアプリケーション設定をする必要があります。
今回はOkta側をメインディレクトリとするためAzure AD側の `{mydomain}.onmicrosoft.com` のほうはフェデレーションの対象外となります。


# 設定手順
## Azure AD側でやること
- カスタムドメインをAzure ADに追加する（まだ追加してなければ）
  - [カスタム ドメインを追加する - Azure Active Directory | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/add-custom-domain)
- プライマリドメインを `{mydomain}.onmicrosoft.com` のほうにする
  - `Azure Active Directory > カスタム ドメイン名` の画面で確認できます
  - ![](2021-06-17-23-43-21.png)
  - プライマリになってなかったら `{mydomain}.onmicrosoft.com` をクリックして `プライマリにする` を選択します
- `{mydomain}.onmicrosoft.com` にグローバル管理者のロールを割り当てたユーザーを用意しておく(個人的に推奨)
  - カスタムドメイン側にしかグローバル管理者が居ない場合、SSOの設定を誤った際に管理者が誰もAzure ADにアクセスできなくなるリスクがあるためです
- MFAを無効化にしてOktaとの二重MFAになるのを防ぐ
  - 全体のMFA設定をOFFにするには `Azure Active Directory > プロパティ > セキュリティの既定値群の管理` にて `セキュリティの既定値群の有効化` を `いいえ` にします
  ![](2021-06-17-23-53-37.png)
    - 二重MFAを防ぐ以外の目的として、これを無効化しておかないとOktaからのプロビジョニングがエラーになります
      > [Office 365 WS-Fed Integration issue](https://support.okta.com/help/s/question/0D51Y00008QoNQN/office-365-wsfed-integration-issue?language=en_US)
    - 無効化するとどんな影響があるのかが気になる方はこちらを参照してください
      > [Azure Active Directory のセキュリティ デフォルト | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/concept-fundamentals-security-defaults)
  - ユーザーごとのMFAについては `Azure Active Directory > ユーザー > ユーザーごとの MFA` にて有効になっているユーザーが居ないか確認してください

### Okta側でやること


# アカウントプロビジョニングを有効化する


# アカウント発行からAzure AD Joinまでの流れ




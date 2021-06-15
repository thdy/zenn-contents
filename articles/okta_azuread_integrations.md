---
title: "OktaをIdPにしてAzure AD(Microsoft 365)のSSOとプロビジョニングをした話"
emoji: "⚔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["okta", "azuread"]
published: false
---
# はじめに
OktaをIdPにしてAzure AD(Microsoft 365)とのSSOとプロビジョニングをしようとしたところ色々とハマったので振り返りを含めポストします。

# 環境
下記のライセンスを所有している前提です。
- Okta
  - SSO
  - MFA
  - Lifecycle Management
  - Universal Directory
- Microsoft
  - Microsoft Enterprise Mobility + Security E3
  - Microsoft 365 Apps for Business

またシナリオとして従来から下記のようなデバイス管理状況でした。
- 利用されているデバイスとしてはMacが中心
- Windowsデバイスも多少ある
  - オンプレADは無い
  - WindowsはAzure AD Join + Intuneで管理されている

# やりたいこと
- SSOの有効化
  - アカウントロック対応等の管理負荷を下げる
  - 利用する側の認証に関する体験を良くする
- アカウントプロビジョニングの有効化
  - 入社などに伴うアカウント管理の工数の削減

# SSOの有効化


# アカウントプロビジョニングの有効化


# アカウント発行からAzure AD Joinまでの流れ

---
title: "macOS Monterey 12.3 にしたら `code .` で VSCode を起動できなくなった問題の対処"
emoji: "🥞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "mac"]
published: true
---
# これはなに
macOS Monterey 12.3 にアップデートしてから、ターミナルから VSCode を起動しようと `code .` を実行するとエラーが出るようになったのでそれの対処をした時のメモです。

# 発生した問題
## エラー内容
```
$ code .
/usr/local/bin/code: line 6: python: command not found
/usr/local/bin/code: line 10: ./MacOS/Electron: No such file or directory
```

## 環境
- OS : macOS Monterey 12.3 Beta
- VSCode : 1.63.0 (Universal)

## 原因
macOS Monterey 12.3 で Python 2.7 が削除されたため `python` コマンドが見つからない状態になっているようです。

https://developer.apple.com/documentation/macos-release-notes/macos-12_3-release-notes

> Python
> Deprecations
> Python 2.7 was removed from macOS in this update. Developers should use Python 3 or an alternative language instead. (39795874)

# 対処
Python 3 を入れてエイリアスを張ろうかなと思い Python 3 をインストールしている間の待ち時間で VSCode の issue を検索していたところ、そのものズバリな PR (マージ済み) が既に上がっているのを発見しました。

https://github.com/microsoft/vscode/pull/141891

Python にも依存しない形になっているのでこちらのほうがいいなと感じ、とりあえず最新版の VSCode にアップデートしてみてこの修正が含まれているのかを確認しようと思いました。

そしてこの対処を行っている時点での最新版である VSCode 1.64.0 にアップデートしてみましたが、 `/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code` の内容は従来どおり Python に依存している状態でした。

VSCode のリリースサイクルは把握していないのですがまだ前述の PR はリリースバージョンにはまだ含まれていないということが分かりました。

そこで少し強引ですが手でローカルファイルの内容を書き換えてしまおうかなと思い VSCode の main リポジトリで該当のファイルを見ると前述の PR から更に修正が入っていました。

https://github.com/microsoft/vscode/blob/main/resources/darwin/bin/code.sh

上記ページのソース内容をローカル環境の `/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code` にそのまま反映し、Python に依存しない形にしたところ問題なくターミナルから `code .` で VSCode が起動するようになりました。

# まとめ
自己責任でやっているので、急いでいない方は VSCode の正式リリース版に含まれるのを待ったほうが良い気がします。

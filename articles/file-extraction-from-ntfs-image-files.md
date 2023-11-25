---
title: "ntfsdump, ntfsfind を用いたイメージからのファイル抽出"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "security", "ntfs", "windows", "forensics"]
published: true
---

## 概要

保全した NTFS イメージファイルから、ファイル, ディレクトリ, 代替データストリーム(下記、アーティファクト)を抽出する手順を示します。
本ソフトウェアは dd(raw), e01, vmdk などの形式に対応しています。

:::message
vmdk などの仮想マシンで使われるイメージファイルの場合、
スナップショットの取得等で分割されたケースには対応していません。
:::

## クイックスタート

下記はdd(raw)形式のイメージファイル `ntfs.raw` から  
`$MFT` を抽出する際のコマンドです。

第一引数の **検索クエリ** は Unix/Linux 形式のパスで指定してください。  
この手順は Windows の場合も共通です。

### Linux

```bash
$ ./ntfsdump '/$MFT' ./ntfs.raw
```

### Windows

```powershell
> ntfsdump.exe "/$MFT" .\ntfs.raw
```

## 詳細な操作手順

下記に詳細な操作手順を示します。

また、例示については基本的に Linux 版を記載していますが、  
Windows で使用する場合はリポジトリの README を参考に読み替えてください。

### 検索と抽出

アーティファクトの検索と抽出を同時に行う場合, **ntfsfind** と組み合わせて使用します。  
下記は **Windows イベントログ (.evtx 形式)** を検索して保存する例です。

#### 個別に検索と抽出を実行する場合

検索には正規表現が使用可能です。  
検索結果は1つのアーティファクトにつき1行で標準出力されます。

```bash
$ ./ntfsfind '.*\.evtx' ./ntfs.raw
Windows/System32/winevt/Logs/Hoge.evtx
Windows/System32/winevt/Logs/Fuga.evtx
Windows/System32/winevt/Logs/Piyo.evtx
```

```bash
$ ./ntfsdump '/Windows/System32/winevt/Logs/Hoge.evtx' ./ntfs.raw
```

#### パイプラインで検索結果を渡す場合

パイプラインで **ntfsfind** の検索結果を直接渡すことも可能です。

```bash
$ ./ntfsfind '.*\.evtx' ./ntfs.raw | ./ntfsdump ./ntfs.raw
```

予め抽出する項目が決まっている場合、ファイルから渡してもよいでしょう。

```bash
$ cat ./artifacts.lst | ./ntfsdump ./ntfs.raw
```

## インストール方法

### バイナリを用いた実行 (推奨)

Windows, Linux(Ubuntu) 用のバイナリを GitHub の Releases で公開しています。  
遷移先のページからバイナリをダウンロードして実行してください。  

![](https://storage.googleapis.com/zenn-user-upload/0b5de7e07939-20231125.png)

:::message
Windows Security など一部のアンチウイルスソフトウェアによって検知される場合があります。  
これは、Python コードを実行ファイルにコンパイルする **Nuitka** を利用していることに起因します。

不安に思う方は後述の **ソースコードからインストール** を参照するか、隔離された仮想環境上で実行してください。
:::

https://github.com/sumeshi/ntfsdump/releases
https://github.com/sumeshi/ntfsfind/releases

### PyPI からのインストール (推奨)

Python3.11 以上をサポートしています。  
インストールする際は下記のコマンドを実行してください。

```bash
$ pip install ntfsdump ntfsfind
```

https://pypi.org/project/ntfsdump/
https://pypi.org/project/ntfsfind/

### ソースコードからインストール

GitHub から直接ソースコードを入手して実行します。  
依存関係の管理に **poetry** を使用しているため、インストールの際はそちらをご利用ください。

```bash
# Download source code from GitHub
$ git pull https://github.com/sumeshi/ntfsdump
$ cd ntfsdump

# Install dependencies
$ pip install poetry
$ poetry install

# Run command using poetry
$ poetry run ntfsdump -h
```

## 既存ソフトウェアとの差異

Autopsy や FTK Imager などのソフトウェアを使用すると、ファイルツリーを見ながらGUIで抽出が可能です。  
また、CUIツールでは多機能な SleuthKit などが有名です。

それらのソフトウェアと比較すると **ntfsdump**, **ntfsfind** の機能は限定されているように見えますが、
2つのツールの目的は統合的なフォレンジックツールではなく、バイナリ1つで役目をこなせるミニマルなツールです。

調査の自動化や, ちょっとした確認のためにこれらのツールが役にたてば嬉しいです。

## コントリビューション

**ntfsdump**, **ntfsfind** は開発中のツールです。  
もしあなたがこのプロダクトに興味があるなら、GitHub で issues, pull requests を送ってくれれば嬉しいです。

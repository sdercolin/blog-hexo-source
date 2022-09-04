---
title: my-dev-2022
date: 2022-09-04 17:19:20
tags: coding
intro: 2022年9月の時点で開発したものをまとめます。
---

2022年9月の時点で開発したものをまとめます。

&nbsp;

### UtaFormatix

歌声合成ソフトウェアのプロジェクトファイルを変換するツールです。

Kotlin for JavaScript + React + MUI で開発しました。

詳しい設計は[昔面接で使ったプレゼン](https://docs.google.com/presentation/d/1bRupSfrmOc4CV1yBL2IxD2MNSqRXTCAQ/edit?usp=sharing&ouid=109277099970585519120&rtpof=true&sd=true)
をご参考ください。

Repository: https://github.com/sdercolin/utaformatix3

Release: https://sdercolin.github.io/utaformatix3/

&nbsp;

### UtaFormatix-Data

UtaFormatix の内部データ形式を公開したものです。

Repository: https://github.com/sdercolin/utaformatix-data

&nbsp;

### HARMOLOID

歌声合成ソフトウェアのプロジェクトファイルに基づいてコーラストラックを生成するツールです。

UtaFormatix と大体同じ作りです。（共通コードもあるけどgit submodule化などしていないくコピペーです；；）

コアロジックをライブラリーとして抽出してあります。

Repository: https://github.com/sdercolin/harmoloid2

Core library repository: https://github.com/sdercolin/harmoloid-core-kt

Release: https://sdercolin.github.io/harmoloid2/

&nbsp;

### vLabeler

音声データのラベリングアプリです。

`Labeler` と命名したConfigファイルでラベリングの仕様を定義できるように設計しました。

現時点では `oto labeler` でUTAUの原音設定を対応し、 `sinsy lab labeler` で NNSVS/ENUNU のラベリングを対応しています。

Compose Desktop (on JVM) で開発しています。Alpha段階です。

Repository / Release: https://github.com/sdercolin/vlabeler

vLabeler / UtaFormatix / HARMOLOID に関する問い合わせ・討論などはこちらの [Discord サーバー](https://discord.gg/TyEcQ6P73y)
をご参加ください。

&nbsp;

### その他

#### [LongWavOtoHelper](https://github.com/sdercolin/LongWavOtoHelper)

setParams は長いwavの対応には問題があるので、「分割->ラベリング->統合」をするツールです。(Windowsのみ)

#### [reclist-gen-cvvc](https://github.com/sdercolin/reclist-gen-cvvc)

UTAU CVVCのReclist を生成する Python application です。

#### [deskit](https://github.com/sdercolin/deskit)

Flutter の勉強で作ったカードゲームなどで使うツールボックスです（ダイス、タイマーなど）。

基本は完成したのですが、Release までしていなくて。。（Apple developer licenceまで買ったのに。。）

#### [UTAUTools](https://github.com/sdercolin/UTAUTools)

- USTRESET: UST をまとめて設定するツールです（Windows、中国語のみ）。[Release](https://akatsuki.sdercolin.com/ustreset/)
- X-SAMPALizer: なんのためか忘れた
- oto2ust: oto.ini の全音を含む ust を作るツールです。[Release](https://akatsuki.sdercolin.com/oto2ust/)

#### [bankoku-tcg.com](https://bankoku-tcg.com/)

友人のカードショップの通販サイトです。
OpenCart で開発しました。

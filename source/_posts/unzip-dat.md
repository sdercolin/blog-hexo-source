---
title: あるゲームのデータを解凍してみた
date: 2019-04-26 01:13:14
tags: coding, cs
intro: あるゲームのデータを見たいけど内容が見れない助けてください、と言われたので試しました。
---
あるゲームのデータを見たいけど内容が見れない助けてください、と言われたので試しました。

渡されたファイルはばかでかい".dat"と".toc"の２つである。
datファイルはすべてのファイルを接続して作ったそうで、tocファイルはそのファイルたちの長さと名前を記述したもの。

tocファイルの一部
```
      1dabe5 28 GeneratedSoundBanks\Switch\959055590.wem
       f42f8 28 GeneratedSoundBanks\Switch\259128839.wem
       edb2b 28 GeneratedSoundBanks\Switch\433205816.wem
       c2e8c 24 GeneratedSoundBanks\Switch\bank0.bnk
        6156 2d GeneratedSoundBanks\Switch\SoundbanksInfo.xml
        5cb2 24 GeneratedSoundBanks\Switch\bank0.txt
         3f6 23 GeneratedSoundBanks\Switch\Init.bnk
         30b 23 GeneratedSoundBanks\Switch\Init.txt
         10d 29 GeneratedSoundBanks\Switch\PluginInfo.xml
```
1列目はファイルのサイズで3列目は名前。

&nbsp;

## 解凍してみる
さてとこれは簡単そうだな。

```csharp
// C#
var size = Convert.ToInt32(row.Substring(0, 12).Trim(), 16);
var name = row.Substring(16);
var bytes = new byte[size];
stream.Read(bytes, 0, size);
var dir = new FileInfo(name).Directory;
if (dir != null)
{
    Directory.CreateDirectory(dir.FullName);
}
File.WriteAllBytes(name, bytes);
```
行ごとに以上のコード実行すれば行けるんじゃね？

と思ったらだめだった。
解凍されたほとんどのファイルは開かない。

&nbsp;

## 間違い探し

そこで詳しくファイルの内容を見る。

幸い連続しているtxtとxmlファイルがあるので読める。
winhexでByte単位でよく見たらファイルの間に確かに`00`が追加されている。

さらに計算してみたら記載されている全てのファイルのサイズの合計はdatのサイズよりも小さい。その差はファイル数の約1.46倍。

んんん？？

1.46倍？？？

セパレータとして追加しているのではないのか？？

…

&nbsp;

## とりあえず空白をskipしよう

一つのファイルを読み込んだ後、`00`を全部skipsして、初めて`00`ではないByteから次のファイルの読み込みを始めるというのはどうだろう。

試したら、半分ぐらいのファイルは正しく解凍された！！！

嬉しいけど。どうやらもともと`00`から始まるファイルがあるらしい。

一体どうやって`00`の数を決めたんだ？？

&nbsp;

## こういう時はlogしよう

とりあえずlog出してskipした`00`の数を見る。

0,2,1,3,0,2,2,1,...

あら？0~3ではないか。

4の倍数になるため足してるのでは？？

正解だった。

ファイルのサイズ+空白の数=4の倍数になるように足しているのだ。

そこまでわかったら簡単だよね。

無事解凍できた。

コード張るわ。

```csharp
// C#
using System;
using System.IO;
using System.Linq;

namespace dat_unzipper
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            var datPath = "DATA.dat";
            var tocPath = "DATA.toc";

            var toc = File.ReadAllLines(tocPath).ToList().Skip(1);
            var stream = new FileStream(datPath, FileMode.Open);
            foreach (var row in toc)
            {
                var size = Convert.ToInt32(row.Substring(0, 12).Trim(), 16);
                var name = row.Substring(16);
                Console.WriteLine(string.Format("Reading {0} of {1} bytes...", name, size));
                var bytes = new byte[size];
                var read = stream.Read(bytes, 0, size);
                if (read == 0)
                {
                    throw new EndOfStreamException();
                }
                var dir = new FileInfo(name).Directory;
                if (dir != null)
                {
                    Directory.CreateDirectory(dir.FullName);
                }
                File.WriteAllBytes(name, bytes);
                Console.WriteLine(string.Format("Successfully written {0}.", name));

                var blankCount = 4 - size % 4;
                if (blankCount == 4)
                {
                    continue;
                }
                var skip = new byte[blankCount];
                stream.Read(skip, 0, blankCount);
                Console.WriteLine(string.Format("Skipped {0} blank byte for {1} bytes.", blankCount, size));
            }
            stream.Close();
        }
    }
}

```

&nbsp;

## 復元しろって？？

案の定復元も頼まれた。

構造を知った以上なんの問題もなく書けるつもりだったけど問題が１つあった。

Windows環境で`.WriteLine`したらCRLFになるけど、tocファイルはLFだった。

なぜかnodepad++はCRLFだと教えてたんだけどね...
winhexで確認したほうが確実。

以下コード。

```csharp
// C#
using System;
using System.IO;
using System.Linq;

namespace dat_zipper
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            var datOutPath = "DATA_OUT.dat";
            var tocPath = "DATA.toc";
            var tocOutPath = "DATA_OUT.toc";

            var toc = File.ReadAllLines(tocPath).ToList();
            var tocWriter = new StreamWriter(tocOutPath);
            tocWriter.Write(toc.First());
            tocWriter.Write("\n");
            var datWriter = new BinaryWriter(new FileStream(datOutPath, FileMode.Create));
            foreach (var row in toc.Skip(1))
            {
                var nameSizeHex = row.Substring(13, 2);
                var name = row.Substring(16);
                Console.WriteLine(string.Format("Reading file: {0} ...", name));
                var file = File.ReadAllBytes(name);
                var size = file.Length;
                var sizeString = size.ToString("x");
                var prefixBlankCount = 12 - sizeString.Length;
                for (int i = 0; i < prefixBlankCount; i++)
                {
                    tocWriter.Write(' ');
                }
                tocWriter.Write(sizeString);
                tocWriter.Write(' ');
                tocWriter.Write(nameSizeHex);
                tocWriter.Write(' ');
                tocWriter.Write(name);
                tocWriter.Write("\n");
                var bytes = new byte[size];
                datWriter.Write(file);
                var suffixBlankCount = 4 - size % 4;
                if (suffixBlankCount < 4)
                {
                    for (int i = 0; i < suffixBlankCount; i++)
                    {
                        byte blankByte = 0;
                        datWriter.Write(blankByte);
                    }
                    Console.WriteLine(string.Format("Added {0} blank byte for {1} bytes.", suffixBlankCount, size));
                }
                Console.WriteLine(string.Format("Successfully written {0}.", name));
            }
            tocWriter.Close();
            datWriter.Close();
        }
    }
}
```

はいおつかれさまでした！
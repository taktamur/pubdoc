---
title: "NerdFontとRictyFontを合成するときにエラーが出た話"
emoji: "✏️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["NerdFont","RictyFont","FontForge","Starship","Homebrew"]
published: true
---
# どんなエラーが出たのか？
 [Ricty に Nerd Font を合成する](https://shnsprk.com/entry/2022/03/28/090000) や [Ricty DiminishedとNerd Fontsを合成する方法(Mac)](https://qiita.com/uhooi/items/dc9a9657f1706283753b) を見ながらフォントの合成をやろうとしたけど、うまく行かなかった。

```sh
❯ ./font-patcher --complete RictyDiminished-BoldOblique.ttf
Nerd Fonts: FontForge module could not be loaded. Try installing fontforge python bindings [e.g. on Linux Debian or Ubuntu: `sudo apt install fontforge python3-fontforge`]
```

環境としてはMacOS(12.6)で、homebrewを使ってfontforgeをインストールしている。



# 対処法  fontforgeコマンドでscript指定する
https://github.com/ryanoasis/nerd-fonts#font-patcher

には2つの方法が紹介されている
```
-   Usage:
    ./font-patcher PATH_TO_FONT
-   Alternative usage: Execute the patcher with the FontForge binary using the script flag:
    fontforge -script font-patcher PATH_TO_FONT
```
１つ目の方法だとうまく行かないけど、2つめのfontforgeコマンドにscriptを渡す方法だと、うまく通った。
```sh
❯ fontforge -script ./font-patcher --complete RictyDiminished-BoldOblique.ttf
Copyright (c) 2000-2023. See AUTHORS for Contributors.
 License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
 with many parts BSD <http://fontforge.org/license.html>. Please read LICENSE.
 Version: 20230101
 Based on sources from 2023-01-01 05:27 UTC-D.
Nerd Fonts Patcher v2.3.0-RC (3.3.0) executing
....(以下ビルドが進む)....
```


# ではなぜ、./font-patcherでエラーが起きてたのか？

 ./font-patcherはpythonで動くけど、そのpythonのバージョンにfontforgeのbindingが無いのでエラーになっていた。

自分のMacには色々なバージョンのpythonが入ってるので、それぞれで試してみる。
python(2系),  python3(3.9)ではエラーとなるが、python3.11 では通る。

```python
import fontforge
```

```sh
~/tmp via 🐍 v2.7.18
❯ python --version
Python 2.7.18
❯ python ./test.py
Traceback (most recent call last):
  File "./test.py", line 1, in <module>
    import fontforge    
ImportError: No module named fontforge

~/tmp via 🐍 v2.7.18
❯ python3 --version
Python 3.9.6

~/tmp via 🐍 v2.7.18
❯ python3 ./test.py
Traceback (most recent call last):
  File "/Users/tak/tmp/./test.py", line 1, in <module>
    import fontforge    
ModuleNotFoundError: No module named 'fontforge'

~/tmp via 🐍 v2.7.18
❯

~/tmp via 🐍 v2.7.18
❯ python3.11 --version
Python 3.11.1

~/tmp via 🐍 v2.7.18
❯ python3.11 ./test.py

~/tmp via 🐍 v2.7.18
❯
```

なので自分の環境では、python3.11で./font-patcherを動かせばエラーは発生しない。
```
❯ python3.11 ./font-patcher --complete RictyDiminished-BoldOblique.ttf
```

このpython3.11はhomebrewでfontforgeを入れたときに、依存関係として指定されているpythonのバージョン。
```
❯ brew deps fontforge | grep python
Warning: Treating fontforge as a formula. For the cask, use homebrew/cask/fontforge
python@3.11
❯
❯ brew ls fontforge | grep python
Warning: Treating fontforge as a formula. For the cask, use homebrew/cask/fontforge
/usr/local/Cellar/fontforge/20230101/lib/python3.11/site-packages/fontforge.so
/usr/local/Cellar/fontforge/20230101/lib/python3.11/site-packages/psMat.so

```
fontforge.soやpsMat.soというのが、pythonのbridgeというやつだろうか。

なのでhomebrewでFontForgeをインストールした時に、依存関係で一緒に入るPythonを使えば、エラーが出ることが無い。
また、別のPythonバージョンでFontForgeを動かす場合には、この辺のファイルを用意すれば良さそう。
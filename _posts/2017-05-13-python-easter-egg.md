---
layout: post
status: publish
published: true
title: Python Easter Egg
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 111
wordpress_url: http://unofficialism.info/posts/?p=111
date: '2017-05-13 10:22:30 +0900'
date_gmt: '2017-05-13 01:22:30 +0900'
categories:
- TIPS
tags:
- Python
---

Python Easter Egg = Pythonの隠しクレジットとはいってもPython基礎本などでよく紹介されているものなので既にご存知かもしれないが背景が面白いのでここで紹介。

Pythonにはthisモジュールという「[The Zen of Python](http://www.python.org/doc/humor/#the-zen-of-python)」(Note 1)を出力するだけのモジュールがある。このモジュール、中身(Note 2)を見てみると分かるが、総ステップにしてわずか28行、ROT13暗号化(Note 3)された文字列を復号化するだけの単純で取るに足らないものかもしれないがこのモジュールが作られた背景は面白い。Barry Warsaw氏が記事「[import this and The Zen of Python](http://www.wefearchange.org/2010/06/import-this-and-zen-of-python.html)」でthisモジュールが誕生にまつわる面白い話を紹介している。

**「[import this and The Zen of Python](http://www.wefearchange.org/2010/06/import-this-and-zen-of-python.html)」の一部簡訳**

> 2001年秋、Foretec Seminar社はのInternational Python Conference #10(以下IPC10、Pyconの前身となるカンファレンス)の準備をしておりPythonコミュニティからそのカンファレンスのスローガンを求めていた。スローガンはTシャツにもプリントされる予定だった。Guideや、Fred、Jeremyや著者達はかつてはForetec Seminar社に所属していたがPythonlabsを結成する2000年に同社を去っている。そしてPythonlabsはPythonコミュニティからのスローガン応募の審査と勝者の選定を担当することになった。応募は500くらいあったが、どれもひどいものだった。Timと著者は1つに絞られるまで何度となく選別作業を行い
> 
> 最終的に"import this"を選んだ。理由は"import this"という言葉の持つふざけた、小バカにしたようなトーンが好きだったからという。
> 
> 著者たちはこの"import this"をスローガンに選んですぐにthisモジュール(this.py)を実装した。モジュールは「The Zen of Python」を出力するだけのものだったが途中TimやGuidoの提案でrot13で暗号化して内容を少し難読化する工夫がされたりもした。IPC10が終わってすぐ、彼らはこのイベントを記念してthisモジュールをPython2.2.1ブランチにコミットした。この時、著者の提案で他の誰にも知られないようにするためにソース管理システムのチェックイン通知機能を停止し、こっそりこのモジュールをPython2.2.1のブランチに含めたのだ。これらのことは彼ら以外に誰にも知らせず内緒で行われた。著者いわく、この彼らの仕込んだeaster egg（thisモジュールのこと。ソフトウェアでいうeaster eggとは隠しコマンドとか、隠しクレジットのようなもの）が誰かに見つかるまではしばらく時間がかかったそうだ。

Barry Warsaw氏が同記事を「That was all back in the day when the Python community had a sense of humor」という一文で締めくくっているように、この記事を読むと当時のPythonコミュニティがいかにユーモア溢れたものだったのかが感じられる。[phython-2.2.1](http://www.python.org/download/releases/2.2.1/)がリリースされたのは2002年4月10日で、それからどれくらい経ってこのthisモジュールが発見されたのか分からないが初めて発見した人は絶対ほっこりしたことだろう。

## Note 1: import this

「[The Zen of Python](http://www.python.org/doc/humor/#the-zen-of-python)」はPythonハッカー、Tim Petersによって書かれた有名な文章でPython設計哲学を要約したようなものと言われている。 Barry Warsaw氏の[記事](http://www.wefearchange.org/2010/06/import-this-and-zen-of-python.html)によると起源はTim Peters氏による1999年6月4日のPython-listへの[この投稿](https://mail.python.org/pipermail/python-list/1999-June/001951.html)のようだ。以下、Pythonインタラクティクモードでimport thisを実行し「The Zen of Python」を表示させた内容：

```shell
$ python
Python 3.4.3 (default, Oct 14 2015, 20:28:29)
[GCC 4.8.4] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
>>>
```

## Note 2: this.py

this.pyの中身。 意味不明なコードをROT13 (Note 3)で複合化することで「The Zen of Python」を出力している。
`/usr/lib/python3.4/this.py`

```python
s = """Gur Mra bs Clguba, ol Gvz Crgref

Ornhgvshy vf orggre guna htyl.
Rkcyvpvg vf orggre guna vzcyvpvg.
Fvzcyr vf orggre guna pbzcyrk.
Pbzcyrk vf orggre guna pbzcyvpngrq.
Syng vf orggre guna arfgrq.
Fcnefr vf orggre guna qrafr.
Ernqnovyvgl pbhagf.
Fcrpvny pnfrf nera'g fcrpvny rabhtu gb oernx gur ehyrf.
Nygubhtu cenpgvpnyvgl orngf chevgl.
Reebef fubhyq arire cnff fvyragyl.
Hayrff rkcyvpvgyl fvyraprq.
Va gur snpr bs nzovthvgl, ershfr gur grzcgngvba gb thrff.
Gurer fubhyq or bar-- naq cersrenoyl bayl bar --boivbhf jnl gb qb vg.
Nygubhtu gung jnl znl abg or boivbhf ng svefg hayrff lbh'er Qhgpu.
Abj vf orggre guna arire.
Nygubhtu arire vf bsgra orggre guna *evtug* abj.
Vs gur vzcyrzragngvba vf uneq gb rkcynva, vg'f n onq vqrn.
Vs gur vzcyrzragngvba vf rnfl gb rkcynva, vg znl or n tbbq vqrn.
Anzrfcnprf ner bar ubaxvat terng vqrn -- yrg'f qb zber bs gubfr!"""

d = {}
for c in (65, 97):
    for i in range(26):
        d[chr(i+c)] = chr((i+13) % 26 + c)

print "".join([d.get(c, c) for c in s])
```

## Note 3: ROT13

[ROT13](http://en.wikipedia.org/wiki/Rot13)は定められた置き換えマップにもとづいて文字を置き換えるだけの単純な暗号方式。次の変換マップに基づいて文字を変換するので例えばA&rarr;N、B&rarr;O、C&rarr;Pのように変換される。
```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
                 ↑↓
NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm
```

ちなみにthis.pyではこの変換マップをモジュール中で生成しているがPython2系、3系ではROT13の実装は標準で組み込まれているので次のように直接decode関数に'rot13'を指定することでROT13で暗号化された文字列sを複合化することができる。
```python
>>> this.s
"Gur Mra bs Clguba, ol Gvz Crgref\n\nOrnhgvshy vf orggre guna htyl.\nRkcyvpvg vf orggre guna vzcyvpvg.\nFvzcyr vf orggre guna pbzcyrk.\nPbzcyrk vf orggre guna pbzcyvpngrq.\nSyng vf orggre guna arfgrq.\nFcnefr vf orggre guna qrafr.\nErnqnovyvgl pbhagf.\nFcrpvny pnfrf nera'g fcrpvny rabhtu gb oernx gur ehyrf.\nNygubhtu cenpgvpnyvgl orngf chevgl.\nReebef fubhyq arire cnff fvyragyl.\nHayrff rkcyvpvgyl fvyraprq.\nVa gur snpr bs nzovthvgl, ershfr gur grzcgngvba gb thrff.\nGurer fubhyq or bar-- naq cersrenoyl bayl bar --boivbhf jnl gb qb vg.\nNygubhtu gung jnl znl abg or boivbhf ng svefg hayrff lbh'er Qhgpu.\nAbj vf orggre guna arire.\nNygubhtu arire vf bsgra orggre guna *evtug* abj.\nVs gur vzcyrzragngvba vf uneq gb rkcynva, vg'f n onq vqrn.\nVs gur vzcyrzragngvba vf rnfl gb rkcynva, vg znl or n tbbq vqrn.\nAnzrfcnprf ner bar ubaxvat terng vqrn -- yrg'f qb zber bs gubfr!"

>>> this.s.decode('rot13')
u"The Zen of Python, by Tim Peters\n\nBeautiful is better than ugly.\nExplicit is better than implicit.\nSimple is better than complex.\nComplex is better than complicated.\nFlat is better than nested.\nSparse is better than dense.\nReadability counts.\nSpecial cases aren't special enough to break the rules.\nAlthough practicality beats purity.\nErrors should never pass silently.\nUnless explicitly silenced.\nIn the face of ambiguity, refuse the temptation to guess.\nThere should be one-- and preferably only one --obvious way to do it.\nAlthough that way may not be obvious at first unless you're Dutch.\nNow is better than never.\nAlthough never is often better than *right* now.\nIf the implementation is hard to explain, it's a bad idea.\nIf the implementation is easy to explain, it may be a good idea.\nNamespaces are one honking great idea -- let's do more of those!"
```

おわり

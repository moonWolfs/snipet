# snipet

## DB

### 重複値と件数洗い出し

```SQL
SELECT
  重複値,
  COUNT(重複値)
FROM
  tableName
GROUP BY
  重複値
HAVING COUNT(重複値) > 1
```

### SQLコマンドラインから結果をファイル出力

```SQL
--SPOOL パス¥ファイル名
--  SELECT文など
--  実行
--SPOOL OFF
--例：
SPOOL desktop¥test.csv
SELECT * FROM PERSONAL
SPOOL OFF
```

## 正規表現

### 用語

__メタ文字__
特別な意味のある文字

__リテラル__
ふつうの文字

__パターン__
演算子のこと

__接頭辞・Prefix__
文字の頭につける値

__接尾辞・Suffix__
文字の後ろにつける値

__部分文字列・Substring__
文字の間につける値

__完全一致__
完全に一致する

__部分一致__
一部分だけ一致する

__前方一致__
前方に一致する

__後方一致__
後方に一致する

### アンカー

文字位置にマッチする

|moji|mean|
|----|----|
|^|行頭|
|$|行末|
|¥A|テキストの先頭|
|¥z|テキストの終端|
|¥b|文字列の間|
|¥B|文字列以外|

### エスケープシーケンス

バックスラッシュを頭につけて特殊文字を表す

|moji|mean|
|----|----|
|¥t|タブ|
|¥n|改行|
|¥s|スペース|
|¥000|8進ASCII|
|¥x00|16進ASCII|
|¥d|¥[0-9¥]|
|¥D|¥[^0-9¥]|
|¥w|¥[A-Za-z0-9¥]|

### キャプチャ

正規表現に¥(¥)をつかうと、その部分に連番が振られて、
変数のように取り出す事ができる。
＊エンジンやプログラム言語によって変数にできたり使えなかったりする。

```reg
2021/10/23
この文字列の数字部分を以下のようにキャプチャする
(¥d{4})(¥d{2})(¥d{2})
前から順に以下のようになる
$1 = 2021
$2 = 10
$3 = 23
これを使って置換することができる
----------------------------
検索 (¥d{4})(¥d{2})(¥d{2})
----------------------------
置換 $3,$1,$2
----------------------------
結果 23/2021/10
```

### 基本

__or__
"文字１|文字２"で表す
いずれかの文字が対象になる

a|b|c
とすると
a,b,cのいずれか１文字にマッチ

__範囲指定__
"¥[0-4¥]"で表す
範囲内の文字が対象になる

[0-4]
とすると
0,1,2,3,4のいずれか１文字にマッチ
0|1|2|3|4
と同じ結果になる

__量指定（繰り返し）__
"{繰り返し数}"で表す
指定した繰り返し数の文字が対象になる

t{2}
とすると
tt
となっている文字にマッチ

¥[0-4¥]
とすると
0-4のいずれかが３回続く文字にマッチ
対象が以下ならば
1325592366499
132のみがマッチする、23、4は３回続いてないのでマッチしない

__否定__
"^"で表す

¥[A-Z¥]
とすると
アルファベット大文字を除く事ができる

### 量指定

#### 欲張り量指定

最長の文字にマッチしようとすること。
この例では一番最後のxが終端になってマッチする。

```txt
----------------------
文字  0aaYYxddx8bdx99
----------------------
検索  Y.+x
----------------------
結果  YYxddx8bdx
----------------------
```

#### 控えめな量指定・最小量指定

最短の文字にマッチしようとすること。
この例では一番最初のxが終端になってマッチする。

```txt
----------------------
文字  0aaYYxddx8bdx99
----------------------
検索  Y.+?x
----------------------
結果  YYx
----------------------
```

#### 範囲量指定

繰り返しの繰り返し。
n回からm回の繰り返し。

x{2,3}
とすると
xx|xxx|xxxx|
と同等の意味になる。

最大回数をなくすと無限大になる。
x{0,}
とすると
x*
とおなじになる。

|exp|count|equal pattern|
|---|-----|-------------|
|x{0,}| 0 >= | x* |
|x{1,}| 1 >= | x+ |
|x{0,1}| 0 or 1 |x?|
|x{2}| 2 = | 22 |
|x{1,3}| 1 >= and 3 <= | x¥|xx¥|xxx|

### 先読み・後読み

()[https://abicky.net/2010/05/30/135112/]

#### 先読み

指定した文字の直後に
指定した文字があるパターンにマッチする。
たとえば
d(?=ay)
ならば、dの直後にayがある文字(ayは含まない)にマッチする。


文字列
abracadabra

検索値
a(?=..a)

結果
**a**bracad**a**bra

便宜的に最初のaをa1,次をa2として
**a1**br**a2**
a1から初めて３文字目がa2なのでマッチする。

#### 否定先読み

先読みの否定。
指定した文字の直後に
指定した文字が__ない__パターンにマッチする。

たとえば
(?!0120)¥d{4}
ならば、直後に0120がこない４桁の数字にマッチする。

#### 後読み

指定した文字の__直前__に
指定した文字があるパターンにマッチする。
たとえば
(?<=no)more
ならば、直前にnoがある(noは含まない)にマッチする。

例
aamoreaaano**more**aaa

#### 否定後読み

指定した文字の直前に
指定した文字が__ない__パターンにマッチする。
たとえば
(?<!no)more
ならば、直前にnoがない(noは含まない)にマッチする。

例
aa**more**aaanomoreaaa

## 小技

```py
# ファイルの文字コードをutf-8からshift-jisに変換する
# 使い方: python3 utf8_to_sjis.py <変換したいファイル名>
# 例: python3 utf8_to_sjis.py test.txt
import sys
import codecs

def main():
    if len(sys.argv) < 2:
        print("usage: python3 utf8_to_sjis.py <filename>")
        sys.exit(1)
    filename = sys.argv[1]
    with codecs.open(filename, "r", "utf-8") as f:
        text = f.read()
    with codecs.open(filename, "w", "shift-jis") as f:
        f.write(text)

if __name__ == "__main__":
    main()
```
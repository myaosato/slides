# オレオレLisp<br>その名はARRP

Satoaki MIYAO

第6回関西Lispユーザ会

2018/06/23 @株式会社アックス

---

## Who am I?
* Satoaki MIYAO
  * myaosato (github)
  * @myao_s_moking(twitter)
* Lisper ３年生
* ５月から、お仕事はPHPer

---

## 紀貫之(土佐日記)

* 男もすなる日記といふものを、女もしてみむとてするなり。

---

## 最近の私

* Lisperもすなる処理系自作といふものを、私もしてみむとてするなり。

---

## 自作Lisp処理系歴

* JAVA製
  * 働き出す直前にJAVAの練習で。
  * 純LISPに近い感じのもの。
  * 一応動く? (忘れた)
  * git使い出す前 ソース紛失


>>>

## 自作Lisp処理系歴

* Julia製
  * レキシカルスコープとか考え出す。
  * 動きはする。
  * でも、思ってた仕様と違う。バグだらけ
  * myaosato/my-first-lisp (github)

>>>

## 自作Lisp処理系歴

* JavaScriptで挑戦 (今回)

---

## ARRP

* Array Processor
* https://github.com/myaosato/arrp

---

## Motive and Target

* 仕事でJavaScript使う。
  * 嫌いじゃないけど、だんだんLISPの文法で書きたくなる。
* JavaScriptの代わりとなるLISPが欲しい。

---

## Results

* それなりに、動くものは作れた。
* でも、まだJavaScriptの代わりとなるレベルではない。

---

## Feature

* Lisp-1
* Lexical scope

>>>

## Feature

* if, cond
* lambda
* let
* progn
* set!, set-g!, defun!

>>>

## Feature

* multiple-value-list

~~~LISP
ARRP-USER> (rem 8 5)
1
3
ARRP-USER> (multiple-value-list (rem 8 5))
(1 3)
~~~

>>>

## Feature

* Macro
  * quote(')
  * quasi-quote(\`)
  * comma(,)
  * comma at(,@)
* gensym

>>>

## Feature

* simple package system
  * just provide a way to change namespace

~~~LISP
ARRP-USER> (current-package)
"ARRP-USER"
ARRP-USER> (set-g! hoge 42)
42
ARRP-USER> hoge
42
ARRP-USER> (change-package! "foo")
"FOO"
FOO> (set-g! hoge 41)
41
FOO> hoge
41
FOO> arrp-user:hoge
42
FOO> (change-package! "arrp-user")
"ARRP-USER"
ARRP-USER> hoge
42
ARRP-USER> foo:hoge
41
~~~


---

## Feature From JS

* リテラル
  * 数値
  * 文字列
  * NaN
  * undefined
  * null
  * true
  * false

>>>

## Feature From JS

* Array
* ArrayBuffer
* DataView

>>>

## Feature From JS

* Number
* Math
* Date
* JSON
* Intl

>>>

## Feature From JS

* String
* RegExp

>>>

## Feature From JS

* Map
* Set

---

## Not Yet<br>Implemented

* Promise
* Proxy
* Generator
* Web API
* ...

---

## Demonstration

>>>

## Number

~~~LISP
ARRP-USER> 42
42
ARRP-USER> 0x2A
42
ARRP-USER> 0o52
42
ARRP-USER> 42.0
42
ARRP-USER> -42
-42
ARRP-USER> -4200e-2
42
ARRP-USER> +EPSILON+
2.220446049250313e-16
ARRP-USER> +MAX-SAFE-INTEGER+
9007199254740991
ARRP-USER> +MIN-SAFE-INTEGER+
-9007199254740991
ARRP-USER> +MAX-VALUE+
1.7976931348623157e+308
ARRP-USER> +MIN-VALUE+
5e-324
ARRP-USER> (is-NaN NaN)
true
ARRP-USER> (is-finite 42)
true
ARRP-USER> (is-integer 42.0)
true
ARRP-USER> (is-safe-nteger 0)
true
ARRP-USER> (parse-float "42.0")
42
ARRP-USER> (parse-int "42")
42
ARRP-USER> (to-exponential 42 3)
"4.200e+1"
ARRP-USER> (to-fixed 42 1)
"42.0"
ARRP-USER> (to-precision 42 4)
"42.00"
~~~

>>>

## threading macro

~~~LISP
ARRP-USER> (defmacro! -> (prev &rest remained)
..........   (if (= (length remained) 0)
..........       prev
..........       `(-> ,(concat (array (first (first remained)))
..........                     (concat (array prev) (rest (first remained))))
..........            ,@(rest remained))))
#<Arrp Macro>
ARRP-USER>
"#<No Input>"
ARRP-USER> (-> (+ 1 2)
..........     (+ 3)
..........     (+ 4))
10
~~~

---

## Thank you

* https://github.com/myaosato/arrp

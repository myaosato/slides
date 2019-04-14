# FizzBazzを書こう<br>with Common Lisp


## 第10回 関西Lispユーザ会

Miyao Satoaki 

---

## 自己紹介

```
(defvar *me*
  '(:job ("プログラマ" ("PHP" "JavaScript"))
    :hobby ("数学" ("読書" ("ミステリ" "SF")) "ゲーム")
    :dialect-of-lisp "Common Lisp"))
```

---

## Common Lisp で FizzBuzz

- 先頭から考えて、1から100に対応する要素数100個のリスト
    - 3の倍数なら文字列"Fizz"
    - 5の倍数なら文字列"Buzz"
    - 3の倍数かつ5の倍数なら代わりに文字列"FizzBuzz"
    - そうでないなら、その数を10進法で表した数の文字列

>>>

## Common Lisp で FizzBuzz

```lisp
(defun run-fizzbuzz-cl ()
  (mapcar #'(lambda (n)
              (cond 
                ((= (mod n 15) 0)
                 "FizzBuzz")
                ((= (mod n 5) 0)
                 "Buzz")
                ((= (mod n 3) 0)
                 "Fizz")
                (t (princ-to-string n))))
          (loop for i from 1 to 100 collect i)))
```

>>>

## Common Lisp で FizzBuzz

```lisp
CL-USER> (run-fizzbuzz-cl)
("1" "2" "Fizz" "4" "Buzz" "Fizz" "7" "8" "Fizz" "Buzz" "11" "Fizz"
 "13" "14" "FizzBuzz" "16" "17" "Fizz" "19" "Buzz" "Fizz" "22" "23"
 "Fizz" "Buzz" "26" "Fizz" "28" "29" "FizzBuzz" "31" "32" "Fizz" "34"
 "Buzz" "Fizz" "37" "38" "Fizz" "Buzz" "41" "Fizz" "43" "44"
 "FizzBuzz" "46" "47" "Fizz" "49" "Buzz" "Fizz" "52" "53" "Fizz"
 "Buzz" "56" "Fizz" "58" "59" "FizzBuzz" "61" "62" "Fizz" "64" "Buzz"
 "Fizz" "67" "68" "Fizz" "Buzz" "71" "Fizz" "73" "74" "FizzBuzz" "76" 
 "77" "Fizz" "79" "Buzz" "Fizz" "82" "83" "Fizz" "Buzz" "86" "Fizz" 
 "88" "89" "FizzBuzz" "91" "92" "Fizz" "94" "Buzz" "Fizz" "97"
 "98" "Fizz" "Buzz")
```

>>>

## Common Lisp で FizzBuzz

- ループ処理(mapcar)
- 分岐処理(cond)
- 処理の順番等(今回で言うと15が先など)

---

# FizzBazzを書こう<br><span style="color:red;">ただしラムダ計算で</span><br>with Common Lisp


## 第10回 関西Lispユーザ会

Miyao Satoaki

---

## 今回のネタ本

- アンダースタンディング コンピュテーション<br>――単純な機械から不可能なプログラムまで
- https://www.oreilly.co.jp/books/9784873116976/
- 6.1 ラムダ計算をまねる
    - RubyでFizzBuzz
    - ただし引数の数を1つに限定したProcのみを用いる

---

## 今回話すこと

- ラムダ計算でのFizzBuzzをCommon Lispでまねる
    - CLでの実装の話
    - チャーチ数の話
    - Y(Z)コンビネータの話すこし
    - デモ

>>>

## 今回話さないこと

- ラムダ計算が何か
- ラムダ計算の実装

---

## ラムダ計算

```lisp
(defmacro -> (param &body body)
  `(lambda (,param) (declare (ignorable ,param)) ,@body))
```

>>>

## カリー化の逆と関数合成(便利のため)

```lisp
(defmacro ! (prev &rest rest)
  (if (= (length rest) 0)
      prev
      `(! (funcall ,prev ,(car rest))
           ,@(cdr rest))))

(defmacro _ (func &rest rest)
  (if (= (length rest) 1)
      (list '! func (car rest))
      `(! ,func (_ ,@rest))))
```

---

## チャーチ数

```lisp
(define-symbol-macro zero (-> p (-> x x)))

(define-symbol-macro one (-> p (-> x (! p x))))

(define-symbol-macro two (-> p (-> x (_ p p x))))

(define-symbol-macro three (-> p (-> x (_ p p p x))))

(define-symbol-macro five (-> p (-> x (_ p p p p p x))))
```

>>>

## チャーチブール値 述語

```lisp
(define-symbol-macro true (-> x (-> y x)))

(define-symbol-macro false (-> x (-> y y)))

(define-symbol-macro _if (-> x x))

(define-symbol-macro zero? (-> p (! p (-> x false) true)))

```

>>>

## チャーチペア

```lisp
(define-symbol-macro pair (-> x (-> y (-> f (! f x y)))))

(define-symbol-macro left (-> p (! p true)))

(define-symbol-macro right (-> p (! p false)))
```

---

## インクリメント デクリメント

```lisp
(define-symbol-macro increment (-> n (-> p (-> x (! p (! n p x))))))

(define-symbol-macro slide 
  (-> p (! pair (! right p) (! increment (! right p)))))

(define-symbol-macro decrement 
  (-> n (! left (! n slide (! pair zero zero)))))
```

>>>

## 算術演算

```lisp
(define-symbol-macro add (-> m (-> n (! n increment m))))

(define-symbol-macro sub (-> m (-> n (! n decrement m))))

(define-symbol-macro mul (-> m (-> n (! n (! add m) zero))))

(define-symbol-macro pow (-> m (-> n (! n (! mul m) one))))
```

>>>

## 比較

```lisp
(define-symbol-macro less-or-equal?
  (-> m (-> n (! zero? (! sub m n)))))
```

---

## しかし、再帰はどうするか

- 名前をつけているのはあくまでも省略表現
    - メタ言語であるCommon Lispの機能である
    - 展開したら無限に長くなってしまうのでこまる
- しかし、実は再帰を書くことが出来る

>>>

## Y コンビネータ

```lisp
(define-symbol-macro y-comb
  (-> f 
      (! (-> x (! f x x)) 
         (-> x (! f x x)))))
```

>>>

## Z コンビネータ (正格評価用)

```lisp
(define-symbol-macro z
  (-> f 
      (! (-> x (! f (-> y (! x x y)))) 
         (-> x (! f (-> y (! x x y)))))))
```

---

## 剰余

```lisp
(define-symbol-macro modulo
  (! z (-> f
         (-> m
           (-> n
             (! _if (! less-or-equal? n m)
                (-> x (! f (! sub m n) n x))
                m))))))
```

---

## リスト処理

```lisp
(define-symbol-macro empty (! pair true true))

(define-symbol-macro unshift (-> l (-> x (! pair false (! pair x l)))))

(define-symbol-macro empty? left)

(define-symbol-macro _first (-> l (_ left right l)))

(define-symbol-macro _rest (-> l (_ right right l)))
```

- unshiftはCLのpush
- 後述のpushはCLのpushではない

>>>

## リスト処理 range(iota)

```lisp
(define-symbol-macro range 
  (! z
     (-> f 
       (-> m 
         (-> n
           (! _if (! less-or-equal? m n)
              (-> x (! unshift (! f (! increment m) n) m x))
              empty))))))

```

>>>

### リスト処理 fold(reduce)

```lisp
(define-symbol-macro fold 
  (! z 
     (-> f
       (-> l 
         (-> x 
           (-> g
             (! _if (! empty? l)
                x
                (-> y 
                  (! g (! f (! _rest l) x g) (! _first l) y)))))))))
```

>>>

### リスト処理 map(mapcar)

```lisp
(define-symbol-macro _map 
  (-> k 
    (-> f
      (! fold k empty
         (-> l (-> x (! unshift l (! f x))))))))
```

---

## 大きな数

```lisp
(define-symbol-macro ten (! mul two five))

(define-symbol-macro fifteen (! mul three five))

(define-symbol-macro hundred (! mul ten ten))
```

>>>

## 文字コード

```lisp
(define-symbol-macro bee ten)

(define-symbol-macro ef (! increment bee))

(define-symbol-macro i (! increment ef))

(define-symbol-macro u (! increment i))

(define-symbol-macro zed (! increment u))

```

>>>

## 文字列 fizz

```lisp
(define-symbol-macro fizz
  (! unshift
     (! unshift
        (! unshift
           (! unshift empty zed)
           zed)
        i)
     ef))
```

>>>

## 文字列 buzz

```lisp
(define-symbol-macro buzz
  (! unshift
     (! unshift
        (! unshift
           (! unshift empty zed)
           zed)
        u)
     bee))
```

>>>

## 文字列 fizzbuzz

```lisp
(define-symbol-macro fizzbuzz
  (! unshift
     (! unshift
        (! unshift
           (! unshift buzz zed)
           zed)
        i)
     ef))
```

>>>

## 整除算

```lisp
(define-symbol-macro div 
  (! z
     (-> f
       (-> m 
         (-> n 
           (! _if (! less-or-equal? n m)
              (-> x (! increment (! f (! sub m n) n) x))
              zero))))))
```

>>>

## push

```lisp
(define-symbol-macro _push
  (-> l
    (-> x
      (! fold l (! unshift empty x) unshift))))
```

>>>

## 文字列(数字列)

```lisp
(define-symbol-macro to-digits
  (! z 
     (-> f
       (-> n
         (! _push 
            (! _if (! less-or-equal? n (! decrement ten))
               empty
               (-> x (! f (! div n ten) x)))
            (! modulo n ten))))))

```

---

## ラムダ計算(Common Lisp) で FizzBuzz

```lisp
(define-symbol-macro run-fizzbuzz
  (! _map (! range one hundred) 
     (-> n 
       (! _if (! zero? (! modulo n fifteen))
          fizzbuzz
          (! _if (! zero? (! modulo n five))
             buzz
             (! _if (! zero? (! modulo n three))
                fizz
                (! to-digits n)))))))

```

>>>

## ラムダ計算 v.s. FizzBuzz

```lisp
(define-symbol-macro run-fizzbuzz
  (! _map (! range one hundred) 
     (-> n 
       (! _if (! zero? (! modulo n fifteen))
          fizzbuzz
          (! _if (! zero? (! modulo n five))
             buzz
             (! _if (! zero? (! modulo n three))
                fizz
                (! to-digits n)))))))

```

```lisp
(defun run-fizzbuzz-cl ()
  (mapcar #'(lambda (n)
              (cond 
                ((= (mod n 15) 0) "FizzBuzz")
                ((= (mod n 5) 0) "Buzz")
                ((= (mod n 3) 0) "Fizz")
                (t (princ-to-string n))))
          (loop for i from 1 to 100 collect i)))
```

---

## デモの時間

- 時間あれば

---

## まとめ

- ラムダ計算FizzBuzz
    - 意外となんとでもなる
    - パズルみたいで楽しい
- Common Lispでの実装
    - ignoreじゃなくてignorable
    - Lisp-2なので...
    - 副産物 マクロを深く展開
        - https://github.com/myaosato/my-cl-etude/
        - deep-macroexpandディレクトリ

---

## 今日のコードはこちら

- https://github.com/myaosato/my-cl-etude/
    - lambda-fizzbuzzディレクトリ
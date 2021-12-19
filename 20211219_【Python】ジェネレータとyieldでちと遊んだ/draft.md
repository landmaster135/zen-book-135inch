# はじまり
どこかのソースで`yield()`を使ったソースを見つけたので、どんな動きをするんだろうかと遊んでみました。

# 基本的な動き
以下の`generator()`のように`yield()`を使用して値を返すやつがジェネレータと呼ばれるやつです。returnだと関数が終了してしまいますが、yieldだと関数が一時停止してくれる。
~~~python
def generator():
    yield('1')
    yield('2')
    yield('3')
    
g = generator()
for i in generator():
    print(i, end=', ')
for i in g:
    print(i, end=', ')

'''
# result
1, 2, 3, 1, 2, 3, 
'''
~~~

# StopIterationエラー
次はこんな風に書いてみる。nextで次の要素を取得します。
~~~python
def generator():
    yield('1')
    yield('2')
    yield('3')
    
g = generator()
for i in generator():
    print(i, end=', ')
for i in g:
    print(i, end=', ')
print(next(generator()), end=', ')
print(next(g), end=', ')

'''
# result
1, 2, 3, 1, 2, 3, 1, 
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-50-8b67aae52322> in <module>
     10     print(i, end=', ')
     11 print(next(generator()), end=', ')
---> 12 print(next(g), end=', ')

StopIteration: 
'''
~~~

どうやら、関数をそのままジェネレータに使用すると、再度関数を頭から実行してくれますが、別の変数に代入すると、関数が終了したら再定義し直さなければならないようです。

~~~python
def generator():
    yield('1')
    yield('2')
    yield('3')
    
g = generator()
for i in generator():
    print(i, end=', ')
for i in g:
    print(i, end=', ')
print(next(generator()), end=', ')
g = generator()
print(next(g), end=', ')

'''
# result
1, 2, 3, 1, 2, 3, 1, 1, 
'''
~~~

# forの処理をまとめられて楽そう
例えば、こう書いたとします。
~~~python
def generator():
    for i in range(1, 7):
        yield(str(i))
    
g = generator()
for i in g:
    if int(i) % 3 == 0:
        print(i + '!!', end=', ')
    else:
        print(i, end=', ')
g = generator()
for i in g:
    if int(i) % 5 == 0:
        print(i + '!!', end=', ')
    else:
        print(i, end=', ')

'''
# result
1, 2, 3!!, 4, 5, 6!!, 1, 2, 3, 4, 5!!, 6, 
'''
~~~

このコードだと分岐が２つに分かれていてテストが面倒くさそうですね。イテレータだと上記の書き方じゃないと実現できないかも。
でも、ジェネレータの中で、処理を完結させれば・・・

~~~python
def generator(multiple):
    for i in range(1, 7):
        if int(i) % multiple == 0:
            yield(str(i) + '!!')
        else:
            yield(str(i))
    
g = generator(3)
for i in g:
    print(i, end=', ')
g = generator(5)
for i in g:
    print(i, end=', ')

'''
# result
1, 2, 3!!, 4, 5, 6!!, 1, 2, 3, 4, 5!!, 6, 
'''
~~~
コードも短くなって、`generator(multiple)`の関数だけテストすれば済みそうですね。

# yield fromでイテレータからyield
`yield from`を使用して三項演算子も使えば、One-Linerで書けそうです。
~~~python
def generator(multiple):
    # <true case> if <conditional> else <false case> for <loop variable> in <iterator>
    yield from [str(i) + '!!' if i % multiple == 0 else str(i) for i in range(1, 7)]
    
g = generator(3)
for i in g:
    print(i, end=', ')
g = generator(5)
for i in g:
    print(i, end=', ')

'''
# result
1, 2, 3!!, 4, 5, 6!!, 1, 2, 3, 4, 5!!, 6, 
'''
~~~

# ジェネレータをnextするとき
yieldを調べた際に、`<generator>.next`みたいな記述が散見されたのですが、それって果たして動くんでしょうか？　僕は試しても全然動かず・・・、これだと動きました。
~~~python
def generator(multiple):
    # <true case> if <conditional> else <false case> for <loop variable> in <iterator>
    yield from [str(i) + '!!' if i % multiple == 0 else str(i) for i in range(1, 7)]
    
g = generator(3)
print(g.__next__(), end=', ')
print(next(g), end=', ')
print(g.__next__(), end=', ')

print(g.next(), end=', ')

'''
1, 2, 3!!, 
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-53-5e77f1ac141f> in <module>
      8 print(g.__next__(), end=', ')
      9 
---> 10 print(g.next(), end=', ')

AttributeError: 'generator' object has no attribute 'next'

'''
~~~
Pythonのリファレンスだとv3.5でも`__next__()`が書いてありましたが、けっこう古いバージョンの表記なのでしょうか？
[Python公式リファレンス_v3.10b2](https://docs.python.org/ja/3.10/reference/expressions.html#yieldexpr "Python reference")

# send()のときの挙動
`send()`がかなりややこしい挙動をしていました。以下の感じの挙動をします。
`send()`を入れない場合の挙動
~~~python
def generator():
    for i in range(1, 7):
        print('generator: ' + str(i))
        res = yield i
        
g = generator()
print('aaaaa')
for i in g:
    print('bbbbb')
    print('main: ' + str(i))

'''
# result
aaaaa
generator: 1
bbbbb
main: 1
generator: 2
bbbbb
main: 2
generator: 3
bbbbb
main: 3
generator: 4
bbbbb
main: 4
generator: 5
bbbbb
main: 5
generator: 6
bbbbb
main: 6
'''
~~~

`send()`を入れた場合の挙動
~~~python
def generator():
    for i in range(1, 7):
        print('generator: ' + str(i))
        res = yield i
        
g = generator()
print('aaaaa')
for i in g:
    print('bbbbb')
    print('main: ' + str(g.send(i)))

'''
# result
aaaaa
generator: 1
bbbbb
generator: 2
main: 2
generator: 3
bbbbb
generator: 4
main: 4
generator: 5
bbbbb
generator: 6
main: 6
'''
~~~

どうやら、`g`からループ変数に代入する時と、`send()`を発動する時にジェネレータを起動するようですね。そして、`send()`の際には、再度動かした時に一時停止した`yield`の返り値が入るようです。

そして、どんな返り値が`res`に入っているのかを見てみると・・・
~~~python
def generator():
    res = None
    for i in range(1, 7):
#         print('generator: ' + str(i))
        print('res: ' + str(res))
        res = yield i
        
g = generator()
print('aaaaa')
for i in g:
    print('bbbbb')
    print('main: ' + str(g.send(i)))
'''
# result
aaaaa
res: None
bbbbb
res: 1
main: 2
res: None
bbbbb
res: 3
main: 4
res: None
bbbbb
res: 5
main: 6
'''
~~~
`g`からループ変数に代入する時には、返り値を貰っていないので`res`はNoneになっていますね。
つまり、ループの中に1つ`send()`を入れると、ジェネレータを1つ飛ばしにiterationしてしまうみたいですね・・・。これって、一体どういう用途で使うんでしょうか（笑）
とりあえず、動きは分かったので、一旦閉じます・・・。

# おしまい
yieldで遊んでみました。楽しかった。

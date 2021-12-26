# ポインタ

変数のアドレスを格納することができ、そのメモリに保存されている値を直接操作することができる。
定義した変数のアドレスはアドレス演算子「&」で取得することができる。
またポインタは型のあとに間接演算子「*」を置くことで定義することができる。 ポインタが指しているデータは参照外し（参照はがし）「\*」を置くことでアクセスできる。

```cpp
int a = 10;
int* b = &a;

std::cout << a << std::endl;  // 10 
std::cout << *b << std::endl; // 10

std::cout << &a << std::endl; // 0x7ffee47dc7ec
std::cout << b << std::endl;  // 0x7ffee47dc7ec
```

ポインタの使い方で有名なものは、関数の引数として与える使い方である。関数の引数は値渡しなので、引数で与えたデータを変更することはできない。そこでアドレスを与えることで、そのデータを直接扱うことができ、関数でデータの書き換えを行うことが出来るようになる。


# 配列

- https://www.renesas.com/jp/ja/support/engineer-school/mcu-programming-peripherals-05

メモリ上に連続して定義される。

以下の例で配列のアドレス空間を確認する。`int`型（4byte）の配列を定義し、各要素のアドレスを出力する。

```cpp
int a[10];

for (int idx=0; idx<10; idx++){
    std::cout << "[" << idx << "] " << &a[idx] << std::endl;
}
```

結果は以下のようになり、

```cpp
[0] 0x7ffee19697c0
[1] 0x7ffee19697c4
[2] 0x7ffee19697c8
[3] 0x7ffee19697cc
[4] 0x7ffee19697d0
[5] 0x7ffee19697d4
[6] 0x7ffee19697d8
[7] 0x7ffee19697dc
[8] 0x7ffee19697e0
[9] 0x7ffee19697e4
```

アドレスが連続していることが分かる。ここでは `int`型配列なので下一桁が4byteずつインクリメントされているのが分かる（画面に出力されているのは不連続だが、その4byteを踏まえて連続であることが分かる）。


# 参照

ポインタは変数に別名をつけていることに相当するが、指しているデータへのアクセス（参照剥がし）など、色々と作法がややこしい場合がある。
そこでこの「参照」というものがC++から導入された。

## 初期化が必要

ポインタ変数と異なり定義の際に初期化が必要。以下の例はコンパイルエラーになる。

```cpp
int &a; 

> main.cpp:6:8: error: declaration of reference variable 'a' requires an initializer
```

使い方としては変数を代入するようにして使用する。変数に別名をつけているように扱えるので、初期化した参照`b`は同じアドレスを指し、また値を変える際も特殊なアクセスの必要はなく、通常の変数の様に扱えば良い。

```cpp
int a = 10;
int& b = a;

std::cout << b << std::endl; // 10
std::cout << &a << std::endl; // 0x7ffee37a97ec
std::cout << &b << std::endl; // 0x7ffee37a97ec

b += 1;
std::cout << a << std::endl; // 11 （参照bを通して、変数aに1が追加されている）
```

また途中で参照する変数を変えることはできない。

```cpp
int a = 10;
int b = 100;

// 変数aを指す参照として定義する
int& c = a;
c += 1;
std::cout << a << std::endl; // 11

// 変数bを指す参照、となるわけではない！
// 単に変数aに、100を代入しているだけ
c = b;
c += 1;
std::cout << b << std::endl; // 100 （c+=1; で変数bの値は変わらない）
```

以上のように参照はポインタの一種であり、取り扱いが簡単であるというメリットがある。
ただし途中で参照先を変更できないなど、細かい制限もある。




# スマートポインタ

C++11から、スマートポインタが導入された。メモリの動的確保の利用の際に生じる諸問題を自動で解決するための仕組みである。従来のポインタは「生のポインタ」と呼ばれたりする。

これまではユーザーが自分で動的確保 `new`を行い、正しく解放 `delete` する必要があった。メモリの解放忘れでメモリリークが起こり、これは非常に凶悪なバグとなる（長期運用して初めて顕著化する問題である。。。）。これらを解決してくれるのがスマートポインタである。

- ex. 二重解放によるバグ

```cpp
int* a = new int;
int b = 10;
a = &b;

delete a;
delete a;

> a.out(75859,0x108b96e00) malloc: *** error for object 0x7ffeecff37dc: pointer being freed was not allocated
> a.out(75859,0x108b96e00) malloc: *** set a breakpoint in malloc_error_break to debug
> Abort trap: 6
```

また以下の例だと、二重開放になるため実行時エラーがでる。つまり、すでに元のポインタが破棄されているため、二重開放となってしまっているのである。

```cpp
int* a = new int(100);
int* b = a;

delete a; // これだけではエラーが出ない
delete b; // 二重開放でエラー
```

スマートポインタは所有権を正しく扱えるようにすることで、どの変数が所有権を持っているかを考えなくて住むようになる。

スマートポインタは動的確保と解放を管理するための枠組みなので、そもそも既存の変数へのアドレスを管理するというのは設計から外れている（と思う）。




## unique_ptr

あるメモリに対する所有権を持つポインタが唯一つであることを保証する。スマートポインタが破棄される際にデストラクタで開放される。そのため明示的に `delete` する必要がない。


- 初期化の方法一覧

```cpp
std::unique_ptr<int> x(new int(100));

std::unique_ptr<int> x;
x.reset(new int(100));

// C++14
std::unique_ptr<int> x = std::make_unique<int>(100);
```

また、「ただ一つ（unique）」を規定するために、コピー不可能なクラスとして実装されている。次のようにコピーコンストラクタを呼ぶような実装はコンパイルエラーになる。

```cpp
std::unique_ptr<int> a(new int(10));
std::unique_ptr<int> b(a); // call to implicitly-deleted copy constructor of 'std::unique_ptr<int>'
```

所有権を移すときはコピーコンストラクタを経由した実装ではなく、ムーブの実装は可能となっている。

```cpp
std::unique_ptr<int> a(new int(10));
std::unique_ptr<int> b(std::move(a)); // コンパイルOK
```

ムーブしたあとの `a` は何も指していないポインタになる。そのため、所有権を移動したあとに参照剥がしをするとセグメンテーション違反になる。

```cpp
std::unique_ptr<int> a(new int(10));
std::cout << *a << std::endl; // 10

std::unique_ptr<int> b(std::move(a));
std::cout << *b << std::endl; // 10
std::cout << *a << std::endl; // Segmentation fault: 11
```


## shared_ptr

## weak_ptr


# 用語

```cpp
int a = 0; // 初期化
a = 10;    // 代入
```

# コピーコンストラクタ

コピーコンストラクタは、同じクラスのオブジェクトを使って初期化するときに呼ばれる。

```cpp
class CTest { 
  public:
    CTest(){};
    CTest(CTest& c) { std::cout << "Copy constructor is called!" << std::endl;}
};

int main(){
  CTest a;
  CTest b(a);  // コピーコンストラクタが呼ばれる
  
  CTest c = a; // コピーコンストラクタが呼ばれる
  
  CTest d;
  d = a;       // コピーコンストラクタは呼ばれない（代入演算子が呼ばれる）
}
```

最後の代入では代入演算子が呼ばれる。そのためコピーコンストラクタは呼ばれないが、このときにも
コピーコンストラクタのような挙動をさせたいときには、演算子のオーバーロードを行う。

## 動作

コピーコンストラクタを定義しない場合は、ただ初期化がされるだけで、メンバ変数の状態などは引き継がない。これをメンバ変数までコピーするためには、そのような動作を定義するためのコピーコンストラクタを実装する。

```cpp
class CTest { 
  public:
    CTest(){};

    void set(int num) { private_member = num;}
    int get() { return private_member;}

  private :
    int private_member;
};


int main(){
  CTest a;
  a.set(10);

  CTest c = a; // コピーコンストラクタが呼ばれる

  std::cout << a.get() << std::endl; // 10
  std::cout << c.get() << std::endl; // -527235072（動作未定義、実行ごとに値が違う）
}
```

以下のようなコピーコンストラクタを定義すると、上記の `a.set(10);` も `CTest c = a;` でコピーされて引き継がれる。

```cpp
CTest(CTest& c){ private_member = c.private_member; };
```

# 演算子オーバーロード

ただしこのままの実装では、代入に関してもデフォルトの実装で値のコピーはされない。

```cpp
CTest a;
a.set(10);

CTest b;
b = a;

std::cout << b.get() << std::endl; // 10は表示されない
```

自作クラスの代入に関しても動作を個別に指定するには、演算子のオーバーロードで実装する。

```cpp
class CTest { 
  public:
    CTest(){};
    // 演算子のオーバーロード
     void operator = (CTest r) {
       private_member = r.private_member;
     };

    void set(int num) { private_member = num;}
    int get() { return private_member;}

  private :
    int private_member;
};


int main(){
  CTest a;
  a.set(10);

  CTest b;
  b = a; // 代入演算子が呼ばれる 

  std::cout << b.get() << std::endl; // 10 
}
```


# コピーの禁止

ここまででコピーコンストラクタの振る舞いをユーザー独自で定義できることを議論してきたが、逆にコピーコンストラクタの使用の禁止を明示できる機能もある。想定としてはコピーできない、インスタンス化したときにその値をもつオブジェクトは唯一つであるような実装をしたい、という環境である。

```cpp
class CTest { 
  public:
    CTest(){};
    CTest(CTest& c) = delete;
};


int main(){
  CTest a;
  CTest b = a; // コンパイルエラー
               // error: call to deleted constructor of 'CTest'
}
```

`delete` を使用することで、コピーコンストラクタを使用する実装でコンパイルエラーを出すことができる。そのため、コピー禁止の実装をより厳格に定義することができる。


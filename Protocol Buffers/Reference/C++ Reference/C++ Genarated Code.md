# C++ 生成コード

## package

```proto
package foo.bar;
```

このように書くことで、ファイル内の内容が`foo::bar`名前空間に配置される。

## message

```proto
message Foo {}
```

このように書くことで、`google::protobuf::Message` を基底クラスに持つ`Foo`というクラスが生成される。`Foo`クラスは基底クラスのメソッドをオーバーライドしており、各メソッドは最速で動作するようにチューニングされている。

しかし、以下のコードを書いた場合は、

```proto
option optimize_for = CODE_SIZE;
```

`Foo`が動作する必要最小限のメソッドのみがオーバーライドされ、生成されるコードのサイズが大幅に縮小される。ただし未実装のメソッドについては基底クラスのリフレクションベースの動作になるので、パフォーマンスは低下する。

.protoファイルに以下が含まれている場合は、

```proto
option optimize_for = LITE_RUNTIME;
```

ディスクリプタやリフレクションのサポートがなくなる反面、`libprotobuf.so` (Windowsでは`libprotobuf-lite.lib`)の代わりに、遥かにサイズの小さい`libprotobuf-lite.so` (Windowsでは`libprotobuf-lite`)とリンクするだけで良くなる。携帯電話などのリソースに制約のあるシステムに適している。

`Foo`から派生した独自のサブクラスを作成することはできない。このクラスをサブクラス化して仮想メソッドをオーバーライドしようとしても、オーバーライドは無視される。これは生成された多くのメソッドが、パフォーマンス向上のために非仮想化されているため。


`Message` 基底クラスには、メッセージ全体の確認、操作、読み出し、書き込みを行うためのメソッドが定義されている（バイナリ文字列からのパース、バイナリ文字列へのシリアライズを含む）。

- `bool ParseFromString(const string& data)` : 引数として渡されたシリアル化済みのバイナリ文字列（ワイヤーフォーマットと呼ぶ）を、パースする。
- `bool SerializeToString(string* output) const` : 引数として渡されたメッセージをバイナリ文字列にシリアライズする。
- `string DebugString()` : proto の `text_format` 表現を文字列で返す(デバッグにのみ使用されるべき)。

これらのメソッドに加え、`Foo`クラスでは以下のメソッドを定義している。

- `Foo()` : コンストラクタ。
- `~Foo()` : デストラクタ。
- `Foo(const Foo& other)` : コピーコンストラクタ。
- `Foo(Foo& &other)` : ムーブコンストラクタ。
- `Foo& operator=(const Foo& other)` : 代入演算子。
- `Foo& operator=(Foo&& other)` : ムーブ代入演算子。
- `void Swap(Foo* other)` : 他のメッセージと内容を入れ替える。
- `const UnknownFieldSet& unknown_fields() const` : このメッセージのパース中に遭遇した未知のフィールドのセットを返す。
- `UnknownFieldSet* mutable_unknown_fields()` : このメッセージの解析中に遭遇した未知のフィールドのミュータブルセットへのポインタを返す。

また、このクラスでは以下の静的メソッドを定義している。

- `static const Descriptor* descriptor()` : 型の記述子を返す。これは、どのフィールドを持ち、それらの型が何であるかなど、型に関する情報を含んでいる。これは、プログラム的にフィールドを検査するために、リフレクションと共に使用することができる。
- `static const Foo& default_instance()` : 新しく構築された`Foo`のインスタンスと同一の`Foo`の`const`シングルトン・インスタンスを返す（したがって、すべての特異フィールドは設定されておらず、すべての反復フィールドは空になっている）。メッセージのデフォルト・インスタンスは、その `New()` メソッドを呼び出すことで、ファクトリとして使用できることに注意。

# ネスト

メッセージは、ネストすることができる。

```proto
message Foo {
  message Bar { }
}
```

この場合、コンパイラは`Foo`と`Bar`の2つのクラスを生成する。また，`Foo` の内部には以下の `typedef` が生成される。

```c++
typedef Foo_Bar Bar;
```

これにより，`Foo::Bar` のようにネストされたクラスとして使うことができる。しかし、C++ではネストした型を前方宣言できない。よって、もし`Bar`を別のファイルで前方宣言したい場合は、`Foo_Bar`と記載する必要がある。

## フィールド

前のセクションで説明したメソッドに加え、`message`内の各フィールドに関するアクセサも生成される。メソッド名は、`has_foo()` や `clear_foo()` のように、小文字のsnake_caseである。

また、フィールド番号の定数も生成する。定数名は「`k`＋フィールド名（CamelCase）＋フィールド番号」である。例えば、`optional int 32 foo_bar = 5;`であれば、`static int kFooBarFieldNumber = 5;` が生成される。

`const` 参照やポインタを返すアクセサについて。`message`に対して何らかの変更を加えるとそれらが無効になる可能性があるので注意。

### 単一の数値フィールド(proto2)

以下のフィールド定義に対して

```proto
optional int32 foo = 1;
```

コンパイラは以下のアクセサを生成する。

- `bool has_foo() const` : フィールドに値がセットされていれば`true`
- `int32 foo() const` : フィールドの値を返す。フィールドに値がセットされていなければデフォルト値を返す。
- `void set_foo(int32 value)` : フィールドに値をセットする。
- `void clear_foo()` : フィールドにセットされた値をクリアする。このあと`has_foo()` は `false` を返す。

その他の数値型・論理型（bool）についても同様。`optional`の代わりに`required`とした場合も同様。

### 単一の数値フィールド(proto3)

以下のフィールド定義に対して

```proto
int32 foo = 1;
```

コンパイラは以下のアクセサを生成する。

- `int32 foo() const` : フィールドの値を返す。フィールドに値がセットされていなければ`0`を返す。
- `void set_foo(int32 value)` : フィールドに値をセットする。
- `void clear_foo()` : フィールドにセットされた値をクリアする。

その他の数値型・論理型（bool）についても同様。


### 単一の文字列フィールド(proto2)

以下のいずれかのフィールド定義に対して

```proto
optional string foo = 1;
required string foo = 1;
optional bytes foo = 1;
required bytes foo = 1;
```

コンパイラは以下のアクセサを生成する。

- `bool has_foo() const` : フィールドに値がセットされていれば`true`
- `const string& foo() const` : フィールドの値を返す。フィールドに値がセットされていなければデフォルト値を返す。
- `void set_foo(const string& value)` : フィールドに値をセットする。
- `void set_foo(string&& value)` (C++11 以降) : フィールドに値をセットする＋ムーブする。
- `void set_foo(const char* value)` : フィールドに値をセットする。NULL終端している必要がある。
- `void set_foo(const char* value, int size)` : フィールドに値をセットする。サイズを明示するので、NULL終端していなくてもよい。
- `string* mutable_foo()` : ポインタを取得する。値がセットされていなければ、空文字列（デフォルト値ではない）をセットしてそのポインタを返す。つまりこれを呼び出したあと、`has_foo()`は`true`を返すようになる。
- `void clear_foo()` : フィールドにセットされた値をクリアする。このあと`has_foo()` は `false` を返す。
- `void set_allocated_foo(string* value)` : フィールドに値をセットする。前の値が存在すれば、そのメモリも解放する。`NULL`を渡すと`clear_foo()`と同じ動きになる。
- `string* release_foo()` : ポインタを取得し、値がセットされていない状態にする。つまりこれを呼び出したあと、`has_foo()`は`false`を返すようになる。

### 単一の文字列フィールド(proto3)

以下のいずれかのフィールド定義に対して

```proto
string foo = 1;
bytes foo = 1;
```

コンパイラは以下のアクセサを生成する。

- `const string& foo() const` : フィールドの値を返す。フィールドに値がセットされていなければ、空文字列を返す。
- `void set_foo(const string& value)` : フィールドに値をセットする。
- `void set_foo(string&value)` (C++11 以降) : フィールドに値をセットする＋ムーブする。- `void set_foo(const char* value)` : フィールドに値をセットする。NULL終端している必要がある。
- `void set_foo(const char* value, int size)` : フィールドに値をセットする。サイズを明示するので、NULL終端していなくてもよい。
- `string* mutable_foo()` : ポインタを取得する。値がセットされていなければ空文字列を返す。
- `void clear_foo()` : フィールドにセットされた値をクリアする。
- `void set_allocated_foo(string* value)` : フィールドに値をセットし、前の値が存在すれば、そのメモリも解放する。`NULL`を渡すと`clear_foo()`と同じ動きになる。
- `string* release_foo()` : ポインタを取得し、値がセットされていない状態にする。

### 単一のenumフィールド(proto2)

### 単一のenumフィールド(proto3)

### 単一のmessageフィールド

### repeatedがついた数値フィールド

### repeatedがついた文字列フィールド

### repeatedがついたenumフィールド

### repeatedがついたmessageフィールド

### oneofがついた数値フィールド

### oneofがついた文字列フィールド

### oneofがついたenumフィールド

### oneofがついたmessageフィールド

### mapフィールド

## Any

## Oneof

## enum

## arena allocation

## services

### interface

### stub

## plugin 挿入ポイント

C++ コード生成器の出力を拡張したいコード生成器プラグインは、与えられた挿入ポイント名を使用して、以下のタイプのコードを挿入することができます。各挿入ポイントは、特に断りのない限り、.pb.cc ファイルと .pb.h ファイルの両方に表示されます。

- includes。インクルードディレクティブ。
- namespace_scope。ファイルのパッケージ/名前空間に属し、特定のクラスには属さない宣言。他のすべての名前空間スコープのコードの後に表示されます。
- global_scope。ファイルの名前空間の外側で、トップレベルに属する宣言。ファイルの一番最後に表示されます。
- class_scope:TYPENAME。メッセージクラスに属するメンバー宣言。TYPENAME は完全なプロト名で、たとえば package.MessageType のようになります。クラス内の他のすべての public 宣言の後に表示されます。この挿入ポイントは、.pb.h ファイルにのみ表示されます。

プロトコルバッファの将来のバージョンで実装内容が変更される可能性があるため、標準コード生成ツールで宣言されたクラスのプライベートメンバーに依存するコードを生成しないでください。
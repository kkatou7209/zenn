---
title: "値オブジェクトをどのように実装するか：複数言語でのアプローチ"
emoji: "💭"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["値オブジェクト", "ドメイン駆動開発", "DDD"]
published: true
---

:::message
誤字や脱字、説明や引用の誤りなどに気づいた方がいれば気兼ねなくコメントにてご連絡ください。

コメント内での議論は一向に構いませんが、筆者以外に対する読者への誹謗中傷などはご遠慮ください。

この記事に掲載されているコードは自由に利用してもらってかまいません。

リンクの共有・引用なども大歓迎です。
:::

# 概要

- 値オブジェクトとは何か
- さまざまな言語での実装方法

# 対象読者

- ドメイン駆動開発に関心がある方
- ドメイン駆動開発を多少学んだことがある方

# 値オブジェクトとは何か

値オブジェクトはドメイン駆動開発の文脈において以下の様に定義づけられています。

> - そのドメイン内の何かを計測したり定量化したり、あるいは説明したりする。
> - 状態を不変に保つことができる。
> - 関連する属性を不可欠な単位として組み合わせることで、概念的な統一体を形成する。
> - 計測値や説明が変わったときには、全体を完全に置き換えられる。
> - 値が等しいかどうかを、他と比較できる。
> - 協力関係にあるその他の概念に、副作用のない振る舞いを提供する
> 
> *ヴォーン・ヴァーノン『実践ドメイン駆動設計』[^1]pp.211*

この記事ではこの様な仕様を各主要言語でどのように実装できるか考えていきます。

上記の説明を簡便にまとめると、値オブジェクトには以下のような特徴があります。

- 特定の概念を表す
- 不変性を持ち、副作用がない
- 同値比較が可能である
- 他と完全に置き換えられる（ライフサイクルがない）
- 複数の値オブジェクトで統一的な値オブジェクトを形成できる

## 特定の概念を表す

値オブジェクトにはそれぞれ対応する概念が存在します。

そのため、異なる概念間での比較などができないよう、同じ概念間での比較を保証する仕組みが必要になります。

また、その概念として正しい値を保持するために、値を検証する仕組みも必要です。

## 不変性を持ち、副作用がない

値オブジェクトでは（クラス等で実装するとして）自身のフィールドを変更したりすることはありません。値オブジェクト自体が値だからです（ここでは値型を念頭にはおいていません）。

不変性を確保するために全てのフィールドはプライベートに保つ必要があります。

そのため、値オブジェクトの利用時に副作用が発生することはありません。

## 同値比較が可能である

値オブジェクトは参照や識別子ではなく、自身が保持する値でのみ比較を行います。

しかしこれは保持している値の型と比較対象の型が一致する場合は比較できるということではありません。

値オブジェクト自体は一つの概念を表しますので、異なる概念との比較が行えてはいけません（値段とゲームスコアを比較するなど）。

## 他と完全に置き換えられる

値オブジェクトが変更される際は必ず新しい値を保持した別の値オブジェクトと置き換えられ、元のオブジェクトは破棄されます。

これはエンティティの様に識別子を基準としたライフサイクルを持たないためです。

## 複数の値オブジェクトで統一的な値オブジェクトを形成できる

値オブジェクトは複数の値オブジェクトを一つにまとめて、新しい値オブジェクトを構築できます。

『実践ドメイン駆動設計』では通貨の値（MonetaryValue）を例に解説しています。通貨の値、例えば「400ドル」は、「400」という値と「ドル」という値に分割でき、通貨の値はこの二つの統一体であると言えます。

# 各言語での実装

以上の前提を踏まえた上で、各言語での実装方法を検証します。

今回は各言語間での比較検証が行いやすいように同じ概念の値オブジェクトを実装します。

あまり複雑な概念は扱いたくないので、「年齢」を例に考えてみましょう。

年齢では以下の条件が求められることを仮定します。

- 負数でない、0以上の整数であること
- 120以下であること

## TypeScript での実装

```ts
class Age { // 1
    
    private _value: number; // 2

    public constructor(value: number) { // 3

        if (isNaN(value)) {
            throw ...
        }

        if (!isFinit(value)) {
            throw ...
        }

        if (isFloat(value)) {
            throw ...
        }

        if (value < 0) {
            throw ...
        }

        if (value > 120) {
            throw ...
        }

        this._value = value;
    }

    public value = () => this._value; // 4

    public equals = (other: Age): boolean => { // 5

        if (!(other instanceof Age)) {
            throw ...
        }

        return this._value === other._value;
    }

    public copy = () => new Age(this._value); // 6
}
```

### 1. クラスを使う

同一概念であるか確認するために TypeScript（もとい JavaScript）ではクラスを使う必要があります。

### 2. プライベートフィールド

不変性を保証するため、値は変更できないようにプライベートで設定します。

実行時に変更されない様にするため　`Object.freeze` を利用する手も考えられますが、複雑になりそうなので利用は慎重になる必要があります。

### 3. 値のバリデーション


まず、数値型であるか判別し、`NaN` あるいは `Infinity`/`-Infinity` ではないかをチェックします。

TypeScript では数値は整数も浮動小数点数もまとめて `number` 型で扱われますので、浮動小数点数でないか `isFinit` で確認します。

最後に負数でなく、120以下であるかを確認します。

ここではコンストラクタ内でバリデーションを行いましたが、バリデーション処理だけを静的メソッドに切り出して汎用的に利用する方法も考えられます。

### 4. 値の取得メソッド

内部値を取得できるように `value` メソッドを追加します。`valueOf` や `[Symbol.primitive]` を使う方法も考えられますが、コード上で値オブジェクトをであることがわかりやすくするために、暗黙の変換は行いません。

### 5. 比較メソッド

比較だけなら `value` で値を取り出せば良いと考えるかもしれませんが、値オブジェクトでは同一の概念かどうかも重要になります。

なので `equals` メソッドでは `Age` のインスタンス化どうかも判定した上で内部値の比較を行います。

### 6. 複製メソッド

TypeScript のクラスには代入時の挙動を制御する方法がありませんので、明示的に複製用のメソッドを作成し、クローンを生成することで不変性を確保します。

利用側は値オブジェクトを渡す際、明示的に `copy` メソッドを呼ぶ必要があります。

### 利用サンプル

```ts
const age = new Age(20);

const invalidAge = new Age(-20); // error

const otherAge = new Age(30);

console.log(age.equals(otherAge)); // => false

age.equals(20); // error

const copy = age.copy();

console.log(age === copy) // => false

console.log(age.equals(copy)) // => true
```

:::message
**コラム：TypeScript でも値オブジェクトを使うべきか**

Node.js でバックエンドをの実装をドメイン駆動開発で行う場合は値オブジェクトはもちろん利用すべきです。

ですが、フロントエンドの場合はどうでしょうか。筆者は「値オブジェクトは積極的に使うべきだ」と考えます。

とは言いつつ、フロントエンドでもドメイン駆動開発を行うべきだとは考えません。UIはドメインの関心事ではないからです。

しかし値オブジェクトという仕組み自体は有用です。ある程度の冗長性を天秤にかけても、コードの可読性の向上とオブジェクト自体による値の検証の仕組みは保守性の観点から採用する価値があると思います。
:::

## Java による実装

```java
public record Age(int value) { // 1
    
    public Age { // 2
        if (value < 0) {
            throw ...
        }
        if (value > 120) {
            throw ...
        }
    }

    public Age copy() { // 3
        return new Age(value);
    }
}
```

### 1. Record

不変にするために Record を使います。

Record にすることで `Object.equals()` と `Object.hashCode()` も自動的にオーバーライドされるので同値比較も可能になります。

:::message
Record の利用には　Java 16 以上が必要です。
:::

### 2. 暗黙的コンストラクタでバリデーション

TypeScript と同じようにコンストラクタでバリデーションします。

### 3. 複製用メソッド

`record` は参照型なので明示的にコピーするメソッドを用意します。

### 利用サンプル

```java
Age age = new Age(20);

Age invalidAge = new Age(-20); // Exception

Age otherAge = new Age(30);

System.out.println(age.equals(otherAge)); // => false

Age copy = age.copy();

System.out.println(age == copy); // => false

System.out.println(age.equals(copy)) // => true
```

## C# による実装

```csharp
public readonly struct Age // 1
{
    public readonly int value;

    public Age(int value) // 2
    {
        if (value < 0) {
            throw ...
        }
        if (value > 120) {
            throw ...
        }
        this.value = value;
    }
}
```

### 1. 構造体を使う

値型である構造体を使用し、不変性を確保するために `readonly struct` を使います。

フィールドの値が同じであれば `Equals` メソッドは `true` を返します。

### 2. バリデーション

コンストラクタを使ってバリデーションを行います。

### 利用サンプル

```csharp
Age age = new Age(20);

Age invalidAge = new Age(-20); // Exception

Age otherAge = new Age(30);

Console.WriteLine(age.Equals(otherAge)); // => false

Age copy = age;

Console.WriteLine(Object.ReferenceEquals(age, otherAge)); // => false

Console.WriteLine(age.Equals(copy)); // => true
```

### Go による実装

```go
type Age struct { // 1
    value int
}

func NewAge(value int) Age { // 2
    if value < 0 {
        panic("...")
    }
    if value > 120 {
        panic("...")
    }
    return Age{value: value}
}

func (a Age) Value() int { // 3
    return a.value
}
```

### 1. 構造体を使う

Goにおいて構造体は値型ですのでコピー用のメソッドは必要ありません。

不変性のためにフィールドはプライベートにします。

また、フィールドの値が同じ場合、等価演算子で `true` が返ってくるので比較等のメソッドは必要ありません。

### 2. 生成メソッド

オブジェクト生成用のメソッドを用意し、その中でバリデーションを行います。

### 3. 内部値取得メソッド

内部値を取得するためのメソッドを用意します。


### 利用サンプル

```go
age := NewAge(20)

invalidAge := NewAge(-20) // panic

otherAge := NewAge(30)

fmt.Println(age.Equals(otherAge)) // => false

copy := age

fmt.Println(copy == age) // => true
```

## Rust による実装

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)] // 1
pub struct Age {
    value: i32,
}

impl Age {

    pub fn new(value: i32) -> Self { // 2
        if value < 0 {
            panic!("...");
        }

        if value > 120 {
            panic!("...");
        }
        
        Age { value }
    }
}
```

### 1. 構造体を使う

Goと同様、構造体を使います。

そのままでは代入した際に所有権が写ってしまうので、`Clone` と `Copy` を実装します。

`PartialEq` と `Eq` を実装することでフィールドの値を含めた同値比較が可能です。別の型同士では比較できません。

### 2. 生成用メソッド

インスタンス化用のメソッドを用意し、その中でバリデーションします。

### 利用サンプル

```rust
let age = Age::new(20);

let invalidAge = Age::new(-20); // panic

let otherAge = Age::new(30);

println!("{}", age == otherAge); // => false

let copy = age;

println!("{}", age == copy); // => true
```

:::message
**コラム：トレイトについて**

今回は内部値が `i32` だったため簡単に実装できましたが、`f64` や `String` の場合はトレイトをそのまま使用できない場合があります。

例えば `f64` は `Eq` を実装していないのでそのまま `Eq` を `derive()` に渡すことができません。

実際の開発で値オブジェクトを実装する際はいろいろ工夫が必要になりそうです。
:::

## Ruby による実装

```ruby
class Age # 1

  def initialize(value) # 2
    if !value.is_a?(Integer)
      raise ...
    end
    if value < 0
      raise ...
    end
    if value > 120
      raise ...
    end
    @value = value
  end

  attr_reader :value # 3

  def equals(other) # 4
    if !other.instance_of?(Age)
        return false
    end
    return other.value == @value
  end

  def copy() # 5
    return Age.new(@value)
  end
end
```

### 1. クラスをつかう

値オブジェクトの表現にクラスを使います。

インスタンスフィールドが自動的にプライベートになるため、不変性を確保できます。

### 2. コンストラクタ

コンストラクタ内でバリデーションを行います。

引数の型が確定していないので `Integer` であることを確認します。

### 3. アクセス用フィールド

ゲッターアクセサを実装し、値は取得できても設定はできない様にします。

### 4. 同値性比較メソッド

内部値の比較のためのメソッドを用意します。

比較対象の型は `instance_of?` で厳密にチェックします。

### 5. 複製用メソッド

クラスが参照型のため、複製用のメソッドを用意します。

### 利用サンプル

```ruby
age = Age.new(20)

invalid_age = Age.new(-20) # Error

other_age = Age.new(30)

puts age.equals(other_age) # => false

copy = age.copy()

puts copy == age # => false

puts copy.equals(age) # => true
```

## Python による実装

```python
from typing import Self

class Age: # 1

    __value: int # 2
    
    def __init__(self, value: int): # 3
        if type(value) != int:
            raise ...

        if value

        self.__value = value

    def value(self) -> int: # 4
        return self.__value

    def equals(self, other: Age) -> bool: # 5
        if type(other) != Age:
            return False
        return self.__value == other.__value

    def copy(self) -> Self: # 6
        return Age(self.__value)
```

### 1. クラスの使用

値オブジェクトの表現にクラスを用います。

### 2. フィールド

不変性を保証するために内部値のフィールドをプライベートにします。

### 3. コンストラクタ

コンストラクタ内で値をバリデーションします。

値の型が確定していないため、`type()` で方を確認します。

### 4. 内部値取得メソッド

内部値を取得するためのメソッドを用意します。

### 5. 同値性比較メソッド

内部値を比較するためのメソッドを用意します。

型が不確定のため `type()` を使って同じ方かどうかを確認します。

### 6. 複製用メソッド

代入時に複製を作成するためメソッドを追加します。

### 利用サンプル

```python
age = Age(20)

invalid_age = Age(-20) # Exception

other_age = Age(30)

print(age.equals(other_age)) # => False

copy = age.copy()

print(age == copy) # => False

print(age.equals(copy)) # => True
```

# 最後に

個人的には Go と C# の実装がスッキリしていて扱いやすいと思いました。

Java は Record が使えれば楽ですが、バージョン 16 以下の場合は `final class` などで実装するほかありません。

Rust は内部値によって実装に差が出てくるでしょう。

TypeScript、Python、Ruby においては実装がかなり冗長になってしまいました。

今回、PHPを取り上げませんでしたが（というか忘れてました）実装方法としてほとんど TypeScript に近い形になると思います。

なお、C と　C++ についてですが、C はオブジェクト指向言語でないため、C++ はアプリケーション開発で採用されることが少ないため割愛しました。

---

[^1]: Vaghn Vernon, *"Implementing Domain-Driven Design"*, Addison-Wesley Professional, 2013 (Vaghn Vernon (2024). *実践ドメイン駆動設計：エリック・エヴァンスが確立した理論を実際の設計に応用する*. 高木正弘. 翔泳社)
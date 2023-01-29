---
title: Java で静的 null 検査をするためのアノテーションまとめ (2023)
description: 2023年の @Nullable、@Nonnull アノテーション事情まとめ
tags:
  - Java
updated_at: 2023-01-29
---

`NullPointerException` を防ぎたいなら静的解析をやれ！

## TL;DL

- [JSpecify](https://jspecify.dev/) の進捗を待とう
- 現時点では、以下の基準で好きなものを選ぶのがいい
  - Checker Framework に依存しても問題ない場合は [Checker Qual](https://mvnrepository.com/artifact/org.checkerframework/checker-qual)
  - Checker Framework に依存したくない場合
    -  IntelliJ IDEA を使ってる人は [JetBrains Java Annotations](https://github.com/JetBrains/java-annotations)
    - Eclipse を使ってる人は [JDT Annotations For Enhanced Null Analysis](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Ftasks%2Ftask-using_null_annotations.htm)
- Lombok、Hibernate Validator と併用する場合は名前の衝突も気にした方がいい
- JSR-305、Jakarta Annotations はやめた方がいい
  - [Which @NotNull Java annotation should I use? - Stack Overflow](https://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use) では `javax.annotation.*` か Checker Framework を奨めている
  - Jakarta Annotations に対する言及が抜けている
  - Java 17 のバグで JSR-305、Jakarta Annotations は型変数に付けられない

![How standards proliferate](./images/how-standards-proliferate.png)

<div style="text-align: center;">
    &copy; <a href="https://xkcd.com/927/">xkcd</a>
</div>

## 想定読者

- Java の `NullPointerException` を防ぎたい人
- どのライブラリが提供する Nullable、Nonnull アノテーションを使えばいいか迷っている人

> 少ないかもしれないけど、誰かに深く刺さるんじゃないかな
> -- 山田リョウ

## ひとりごと

> それは10億ドルにも相当する私の誤りだ。null参照を発明したのは1965年のことだった。当時、私はオブジェクト指向言語 (ALGOL W) における参照のための包括的型システムを設計していた。目標は、コンパイラでの自動チェックで全ての参照が完全に安全であることを保証することだった。しかし、私は単にそれが容易だというだけで、無効な参照を含める誘惑に抵抗できなかった。これは、後に数え切れない過ち、脆弱性、システムクラッシュを引き起こし、過去40年間で10億ドル相当の苦痛と損害を引き起こしたとみられる。
> -- [Charles Antony Richard Hoare](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%B3%E3%83%88%E3%83%8B%E3%83%BC%E3%83%BB%E3%83%9B%E3%83%BC%E3%82%A2)

`null` が言語仕様になければ…

## 交わり

辛く苦しい `null` のある世界に飛び込む前に認識を合わせよう。

### Nullness アノテーションとは？

さまざまなライブラリで提供されている `null` 検証を**静的**に行うためのアノテーション全般のことを指す。

#### Nullness アノテーションの必要性

現代において Nullness アノテーションが存在する主な理由は次の2つである。

- 静的解析による `NullPointerException` の検知
- Kotlin との連携

1つ目は、IDE や静的解析ツールでの利用が目的だ。
2つ目は、Kotlin の言語コンセプトである [Null safety](https://kotlinlang.org/docs/null-safety.html) に関係する。
Java と Kotlin は相互に定義されたコードを呼び出すことができるが、Nullable、Nonnull 相当のアノテーションを Java 側のコードで適切に付与することで Kotlin 側でも null の可能性がある型として扱われる。
[JSR 305](https://jcp.org/en/jsr/detail?id=305) が起案された当時と異なるのは後者の部分だろう。

話は変わるが、プログラミングにおいてそもそも `null` は必要なのだろうか。
その答えはアントニー・ホーアの後悔からもわかるように No だ。
では、プログラミング言語において `null` (空の参照、未初期化) に対して取れる手段は何があるだろう。
その答えは、主に次の3つがあると私は考えている。

1. `null` を言語仕様として受け入れ、プログラマーが用心深く扱うことを前提とする
2. 変数の型に `null` になる可能性があるかを表す情報を埋め込み、矛盾するコードを**コンパイル時に弾く**
3. 言語仕様に `null` のような空の参照を表す値を含めない

それぞれ具体例として有名 (?) な言語を上げると以下のように分類される。

1. C、C++、Java、Go
2. TypeScript、Kotlin、Swift、Dart (2.12~)
3. Haskell、OCaml、Rust

最も賢いアプローチは `3.` だろう。
`2.` に属している言語は大体が他言語との連携や初期の仕様に引き連られた結果として言語仕様に `null` が残っているように思える。
この中だと愚かにもアントニー・ホーアの二の轍を踏んだのは初期の Dart の言語設計者だけのようだ。

本記事では、`NullPointerException` が発生する可能性があるコードが存在するか静的に検証することを 静的 null 検証と呼ぶことにする。
静的 null 検証は `1.` に属する言語を `2.` に近づける行為と考えられる。
ある変数に `null` が代入可能かどうかを型に含めるのではなくアノテーションとして宣言する。
言語仕様上は、null 参照にアクセスして `NullPointerException` が発生するようなコードもコンパイルできてしまうが、それを外部のツールで補うことでそのようなコードをコンパイルさせないようにするのだ。
それにより、疑似的に Java を完全ではないものの `NullPointerException` が発生しないことを保証しつつコーディングすることを目指す。

#### アノテーションを書ける場所

アノテーションは、言語仕様によって定められた場所に付与することができる。
付与することができる場所は、アノテーションを定義するとき `@Target` アノテーションを使って指定する。

```java
import java.lang.annotation.Target;

@Target({...}) // ElementType を並べて指定する
public @interface Annotation {
}
```

[ElementType](https://docs.oracle.com/javase/jp/17/docs/api/java.base/java/lang/annotation/ElementType.html) では、以下の列挙型定数が定義されている。

| ElementType        | 説明                                                                           |
|--------------------|--------------------------------------------------------------------------------|
| `TYPE`             | クラス、インターフェース(注釈インターフェースを含める)、列挙またはレコード宣言 |
| `FIELD`            | フィールド宣言(enum定数を含む)                                                 |
| `METHOD`           | メソッド宣言                                                                   |
| `PARAMETER`        | 仮パラメータ宣言                                                               |
| `CONSTRUCTOR`      | コンストラクタ宣言                                                             |
| `LOCAL_VARIABLE`   | ローカル変数宣言                                                               |
| `ANNOTATION_TYPE`  | 注釈インターフェース宣言                                                       |
| `PACKAGE`          | パッケージ宣言                                                                 |
| `TYPE_PARAMETER`   | 型パラメータ宣言                                                               |
| `TYPE_USE`         | 型の使用                                                                       |
| `RECORD_COMPONENT` | レコード・コンポーネント                                                                               |

レコードが新しく Java に追加されたため `RECORD_COMPONENT` が Java16 から追加されている。

アノテーションは、各 ElementType の説明で書かれている場所に付けることができる。
それぞれの詳細は、公式ドキュメント[^The Java Language Specification 9.6.4.1] [^The Java Language Specification 9.7.4]を参照してください。

[^The Java Language Specification 9.6.4.1]: [9.6.4.1. @Target - The Java Language Specification](https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.6.4.1)
[^The Java Language Specification 9.7.4]: [9.7.4. Where Annotations May Appear - The Java Language Specification](https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.7.4)

### 決定不能問題

もし、あなたがコンピュータサイエンスに関して学んだことがなければ、次のような疑問を持つかもしれない。

> アノテーションを付けなくても静的 null 検証ができるのではないか？

この疑問に対する回答は否定的で **No** だ。
**任意**のプログラムに対してこの手の問題を解くということは[停止性問題](https://ja.wikipedia.org/wiki/%E5%81%9C%E6%AD%A2%E6%80%A7%E5%95%8F%E9%A1%8C)に落とし込むことができ、解くことができない問題に分類される。

そのため、静的解析ツールに対して人間がアノテーションによりある変数に `null` が代入される可能性があるかを伝え、それを元に `NullPointerException` が発生する可能性があるコードの検出をする、といった手法になっている。

## 比較表

有名なライブラリとそのライブラリが提供する Nullable、Nonnull を表すためのアノテーションを表にまとめた。

| ライブラリ                                 | Nullable       | Nonnull    |
|--------------------------------------------|----------------|------------|
| JSR-305                                    | `@Nullable`    | `@Nonnull` |
| Jakarta Annotations                        | `@Nullable`    | `@Nonnull` |
| JDT Annotations For Enhanced Null Analysis | `@Nullable`    | `@NonNull` |
| NetBeans API Annotations Common            | `@NullAllowed` | `@NonNull` |
| JetBrains Java Annotations                 | `@Nullable`    | `@NotNull` |
| Checker Framework                          | `@Nullable`    | `@Nonnull` |
| SpotBugs Annotations                       | `@Nullable`    | `@NonNull` |

Android SDK にも同様のアノテーションが含まれているが、ここではプラットフォームに依存せず使われるであろうアノテーションのみを挙げている。
Nullable、Nonnull 系のアノテーションは、現時点 (2023/01/29) においてスタンダードはもちろんのこと、デファクトスタンダードすら存在していないため、上記のように複数のライブラリがアノテーションを提供しており、それらに依存せずに開発されているフレームワークでは各自が独自の定義をしているのが現状だ。
例えば、[Spring Boot](https://spring.pleiades.io/spring-framework/docs/current/javadoc-api/org/springframework/lang/package-summary.html) や [AssertJ](https://javadoc.io/doc/org.assertj/assertj-core/3.14.0/org/assertj/core/annotations/package-summary.html) が上げられる。

上記の表を見てこち亀の画像が頭に思い浮かんだ読者が多いのではないだろうか。

<img src="./images/kochikame.jpeg" alt="全部同じじゃないですか" width="360" style="display: block; margin: auto;" loading=lazy />
<div style="text-align: center;">
    &copy; こちら葛飾区亀有公園前派出所 141巻
</div>

悲しいことに実装を見るとそれぞれのアテノーションの定義が異なり、できることも少しずつ違うことがわかる。
どれを使っても同じであればどれほど楽だっただろう。

それでは、各ライブラリの定義を見ていこう。
それぞれの実装について問題点が存在するのだが、最初は実装を一通り眺めて後から指摘することにする。

## 実装

### JSR-305

- [JSR 305: Annotations for Software Defect Detection](https://jcp.org/en/jsr/detail?id=305)
- [Maven Repository](https://mvnrepository.com/artifact/com.google.code.findbugs/jsr305)
- [Javadoc](https://javadoc.io/doc/com.google.code.findbugs/jsr305/latest/index.html)

#### [`@Nullable`](https://javadoc.io/static/com.google.code.findbugs/jsr305/3.0.2/javax/annotation/Nullable.html)

```java
@Documented
@TypeQualifierNickname
@Nonnull(when=UNKNOWN)
@Retention(value=RUNTIME)
public @interface Nullable
```

#### [`@Nonnull`](https://javadoc.io/static/com.google.code.findbugs/jsr305/3.0.2/javax/annotation/Nonnull.html)

```java
@Documented
@TypeQualifier
@Retention(value=RUNTIME)
public @interface Nonnull
```

### Jakarta Annotations

- [公式サイト](https://jakarta.ee/specifications/annotations/)
- [Maven Repository](https://mvnrepository.com/artifact/jakarta.annotation/jakarta.annotation-api)
- [Javadoc](https://javadoc.io/doc/jakarta.annotation/jakarta.annotation-api/latest/jakarta.annotation/module-summary.html)

#### [`@Nullable`](https://javadoc.io/doc/jakarta.annotation/jakarta.annotation-api/latest/jakarta.annotation/jakarta/annotation/Nullable.html)

```java
@Documented
@Retention(RUNTIME)
public @interface Nullable
```

#### [`@Nonnull`](https://javadoc.io/doc/jakarta.annotation/jakarta.annotation-api/latest/jakarta.annotation/jakarta/annotation/Nonnull.html)

```java
@Documented
@Retention(RUNTIME)
public @interface Nonnull
```

### JDT Annotations For Enhanced Null Analysis

- [Eclipse Wiki](https://wiki.eclipse.org/JDT_Core/Null_Analysis)
- [Maven Repository](https://mvnrepository.com/artifact/org.eclipse.jdt/org.eclipse.jdt.annotation)
- [Javadoc](https://javadoc.io/doc/org.eclipse.jdt/org.eclipse.jdt.annotation/2.0.0/org/eclipse/jdt/annotation/package-summary.html)

#### [`@Nullable`](https://javadoc.io/doc/org.eclipse.jdt/org.eclipse.jdt.annotation/2.0.0/org/eclipse/jdt/annotation/Nullable.html)

```java
@Documented
@Retention(value=CLASS)
@Target(value=TYPE_USE)
public @interface Nullable
```

#### [`@NonNull`](https://javadoc.io/doc/org.eclipse.jdt/org.eclipse.jdt.annotation/2.0.0/org/eclipse/jdt/annotation/NonNull.html)

```java
@Documented
@Retention(value=CLASS)
@Target(value=TYPE_USE)
public @interface NonNull
```

### NetBeans API Annotations Common

- [Maven Repository](https://mvnrepository.com/artifact/org.netbeans.api/org-netbeans-api-annotations-common)
- [Javadoc](https://bits.netbeans.org/dev/javadoc/org-netbeans-api-annotations-common/overview-summary.html)

#### [`@NullAllowed`](https://bits.netbeans.org/dev/javadoc/org-netbeans-api-annotations-common/org/netbeans/api/annotations/common/NullAllowed.html)

```java
@Documented
@Target(value={FIELD,PARAMETER,LOCAL_VARIABLE})
@Retention(value=CLASS)
public @interface NullAllowed
```

#### [`@NonNull`](https://bits.netbeans.org/dev/javadoc/org-netbeans-api-annotations-common/org/netbeans/api/annotations/common/NonNull.html)

```java
@Documented
@Target(value={FIELD,METHOD,PARAMETER,LOCAL_VARIABLE})
@Retention(value=CLASS)
public @interface NonNull
```

### JetBrains Java Annotations

- [公式サイト](https://www.jetbrains.com/help/idea/annotating-source-code.html)
- [GitHub](https://github.com/JetBrains/java-annotations)
- [Maven Repository](https://mvnrepository.com/artifact/org.jetbrains/annotations)
- [Javadoc](https://javadoc.io/doc/org.jetbrains/annotations/latest/index.html)

#### [`@Nullable`](https://javadoc.io/doc/org.jetbrains/annotations/latest/org/jetbrains/annotations/Nullable.html)

```java
@Documented
@Retention(CLASS)
@Target({METHOD,FIELD,PARAMETER,LOCAL_VARIABLE,TYPE_USE})
public @interface Nullable
```

#### [`@NotNull`](https://javadoc.io/doc/org.jetbrains/annotations/latest/org/jetbrains/annotations/NotNull.html)

```java
@Documented
@Retention(CLASS)
@Target({METHOD,FIELD,PARAMETER,LOCAL_VARIABLE,TYPE_USE})
public @interface NotNull
```

### Checker Framework

- [公式サイト](https://checkerframework.org/)
- [Maven Repository](https://mvnrepository.com/artifact/org.checkerframework/checker-qual)

#### [`@Nullable`](https://checkerframework.org/api/org/checkerframework/checker/nullness/qual/Nullable.html)

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={TYPE_USE,TYPE_PARAMETER})
@SubtypeOf(value={})
@QualifierForLiterals(value=NULL)
@DefaultFor(types=java.lang.Void.class)
public @interface Nullable
```

#### [`@NonNull`](https://checkerframework.org/api/org/checkerframework/checker/nullness/qual/NonNull.html)

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={TYPE_USE,TYPE_PARAMETER})
@SubtypeOf(value=MonotonicNonNull.class)
@QualifierForLiterals(value=STRING)
@DefaultQualifierInHierarchy
@DefaultFor(value=EXCEPTION_PARAMETER)
@UpperBoundFor(typeKinds={PACKAGE,INT,BOOLEAN,CHAR,DOUBLE,FLOAT,LONG,SHORT,BYTE})
public @interface NonNull
```

### SpotBugs Annotations

- [Maven Repository](https://mvnrepository.com/artifact/com.github.spotbugs/spotbugs-annotations)
- [Javadoc](https://www.javadoc.io/doc/com.github.spotbugs/spotbugs-annotations/3.1.0/edu/umd/cs/findbugs/annotations/package-summary.html)

#### [`@Nullable`](https://www.javadoc.io/doc/com.github.spotbugs/spotbugs-annotations/3.1.0/edu/umd/cs/findbugs/annotations/Nullable.html)

```java
@Documented
@Target(value={FIELD,METHOD,PARAMETER,LOCAL_VARIABLE})
@Retention(value=CLASS)
@Nonnull(when=UNKNOWN)
@TypeQualifierNickname
public @interface Nullable
```

#### [`@NonNull`](https://www.javadoc.io/doc/com.github.spotbugs/spotbugs-annotations/3.1.0/edu/umd/cs/findbugs/annotations/NonNull.html)

```java
@Documented
@Target(value={FIELD,METHOD,PARAMETER,LOCAL_VARIABLE})
@Retention(value=CLASS)
@Nonnull(when=ALWAYS)
@TypeQualifierNickname
public @interface NonNull
```

## 問題点

全部同じに見える？これだからしろうとはダメだ！もっとよく見ろ。  
重要なのは `@Target`、その次に `@Retenion` だ。
それ以外のアノテーションは基本的に無視して問題ないと考えられる。

それでは、それぞれアノテーションの問題点を見ていこう。

### JSR-305、Jakarta Annotations

この二つのアノテーションの問題点は、`@Retention` が何故か `Runtime` になっている点と `TYPE_USE` が設定されていない点だ。

`TYPE_USE` が設定されていない理由は、型変数に対してアノテーションが付けられるようになったのが Java 8 からで、JSR-305 はそれ以前に作られたものだからだ。
また、Jakarta は恐らく JSR-305 を基本とし、FindBugs (SpotBugs) の定義にあえて手を加えないようにしたからだろう。
以下の Issue でも十分に議論はされておらず[^#89][^#50]、ちゃんとした仕様の策定は後述の JSpecify に任せる、というスタンスなのだろう。

[^#50]: [Add Nullable and NotNull annotations #89 - GitHub](https://github.com/jakartaee/common-annotations-api/issues/89)
[^#89]: [Please include JSR305 into the project #50 - GitHub](https://github.com/jakartaee/common-annotations-api/issues/50)

ここで `@Target` の Javadoc を読んだことがある人は `TYPE_USE` が `@Target` で指定されていないと型変数にアノテーションが付けられないことに違和感を覚えるのではないだろうか。
Java 17 の `@Target` の [Javadoc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/Target.html) では以下のような記述がある。

> If an @Target meta-annotation is not present on an annotation interface T, then an annotation of type T may be written as a modifier for any declaration.

この文を読むと `@Target` がないアノテーションはいたるところで付けられるように思える。
だが、待って欲しい。ここで Java 11 の `@Target` の [Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html) を見ると次のようになっている。

> If an @Target meta-annotation is not present on an annotation type T , then an annotation of type T may be written as a modifier for any declaration except a type parameter declaration.

Java 17 の Javadoc では消えてしまっているが Java 11 では *except a type parameter declaration* という文言が存在している[^他のJavaバージョン]。

[^他のJavaバージョン]: [Java 8](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Target.html) でも同様の記述がある。[Java 5](https://docs.oracle.com/javase/1.5.0/docs/api/java/lang/annotation/Target.html) では `TYPE_USE`、`TYPE_PARAMETER` がないため異なる説明がされている。

IntelliJ で利用する SDK に azul-17 を指定してコンパイルしようとしたところ、

```
error: annotation @Nullable not applicable in this type context
```

といったエラーが発生したため、`@Target` が付いていないアノテーションは型変数宣言で使えないといった仕様は Java 17 でも健在。
しかし、これはおかしいと思い調べてみるとどうやら言語仕様と実装の乖離があるようだ。
Javadoc の修正が行われて [commit](https://github.com/openjdk/jdk17/commit/ecc37b06f283c18ab4aa2b23562843bca14da85d) を辿ると [Issue](https://github.com/openjdk/jdk17/pull/256#issuecomment-902598483) でこのことが指摘されていた。
仕様変更を含む修正となってしまい Java 17 で直すには遅すぎるため、次の LTS バージョンである Java 21 では修正されることを期待して待つしかないが、この問題は忘れさられてしまったのか2023/01/29現在においても[修正されていない](https://github.com/openjdk/jdk/blob/jdk-21+7/src/jdk.compiler/share/classes/com/sun/tools/javac/comp/Check.java#L3474-L3494)。

話が大幅にズレてしまったが元の話題に戻すと `TYPE_USE` が Jakarta でも付いていないのは納得できるが `@Retention` に `Runtime` が設定されているのは何故だかわからない。
JSR-305 は静的解析だけでなく Runtime での解析も目的としていたのだろうか :thinking_face:。

どのような理由があるにしても JSR-305、Jakarta Annotations には `TYPE_USE` が付いていないため現代においては利用の候補から外れるだろう。

JRS-305、Jakarta Annotations を採用しにくい理由はもう一つある。
JSR-305 はリポジトリ名やアーティファクトIDは FindBugs 由来の命名だが、パッケージは `javax.annotation.*` となっている。
そして世間一般としては、Java EE で使われていた `javax` は、Jakarta EE で提供される `jakarta` に移行してく流れがある。
また、JSR-305 は Java 9 で登場したモジュールシステムの観点からも `javax.annotation.*` が衝突するという問題がある。
そのため、今日においては JSR-305 を使うくらいであれば Jakarta Annotations の方がいいが、各種ツールの Jakarta 対応が完了しきっていないという点を考慮すると、Jakarta Annotations も推奨することはできない。

### JDT Annotations For Enhanced Null Analysis、JetBrains Java Annotations、Checker Framework

JDT Annotations For Enhanced Null Analysis、JetBrains Java Annotations、Checker Framework は `@Target` に渡している `TargetElement` が異なるが、アノテーションを付与できる場所に違いはない (はず)。
`TYPE_USE` アノテーションは、[Javadoc](https://docs.oracle.com/javase/jp/17/docs/api/java.base/java/lang/annotation/ElementType.html) に以下のような説明がされている。

> 定数TYPE_USEはJLS 4.11に記載された15の型コンテキストと、2つの宣言コンテキスト(注釈型宣言を含む型宣言および型パラメータ宣言)に対応しています。
> 
> たとえば、型に@Target(ElementType.TYPE_USE)のメタ注釈が付けられた注釈は、フィールドの型(ネストされた型、パラメータ化された型、または配列型である場合はそのフィールドの型の内部)に記述でき、クラス宣言などの修飾子として指定することもできます。
> 
> TYPE_USE定数には、注釈型にセマンティックスを提供する型チェッカーの設計者の便宜を図って、型宣言および型パラメータ宣言が含まれています。たとえば、注釈型NonNullに@Target(ElementType.TYPE_USE)のメタ注釈が付けられている場合、型チェッカーによる@NonNull class C {...}の処理では、クラスCの変数はすべてnull以外であることを示しますが、他のクラスの変数がnull以外であるか否かは引き続き、変数の宣言に@NonNullが表示されているかどうかに基づいて決められます。

JLS は Java Language Specification の略で[ここ](https://docs.oracle.com/javase/specs/jls/se17/html/index.html)で閲覧できる。

JetBrains Java Annotations は、`TYPE_USE` 以外にも `METHOD`、`FIELD`、`PARAMETER`、`LOCAL_VARIABLE` を指定しているが、これは Java 8 より前のバージョンをサポートするためのようだ。
Java 8 より前のバージョンのためのライブラリ生成タスクで `ElementType.TYPE_USE` を消すためのコードが存在している[^JetBrains Java AnnotationsのJava 5対応タスク]。

[^JetBrains Java AnnotationsのJava 5対応タスク]: [JetBrains/java-annotations](https://github.com/JetBrains/java-annotations/blob/24.0.0/java5/build.gradle#L36-L40)

Checker Framework は `TYPE_USE` と `TYPE_PARAMETER` の両方が `@Target` で指定されているが、`TYPE_USE` は `TYPE_PARAMETER` を内包しているはずなので、JDT Annotations For Enhanced Null Analysis と変わらない。

### NetBeans API Annotations Common、SpotBugs Annotations

残るは NetBeans API Annotations Common と SpotBugs Annotations だがどちらも `TYPE_USE` が `@Target` で指定されていないことから選択肢から外れる。

### まとめ

Java 8 で追加された `TYPE_USE`、`TYPE_PARAMETER` に対応しているのは、

- JDT Annotations For Enhanced Null Analysis
- JetBrains Java Annotations
- Checker Framework

の 3 つ。現時点 (2023/01/29) ではこれらの中から選択するのが懸命で、IBMの[ドキュメント](https://www.ibm.com/docs/ja/radfws/9.6?topic=SSRTLW_9.6.0/org.eclipse.jdt.doc.user/tasks/task-using_null_type_annotations.html)でも書かれている通り、NULL型注釈による静的検査の恩恵を最大限享受することができる。

## 静的解析ツール

### 主要なツール

2023/01/29 現在において null 検査ができる有名なツールは以下の4つだろう。

- [Checker Framework](https://checkerframework.org/)
- [SpotBugs](https://spotbugs.github.io/)
- [Nullaway](https://github.com/uber/NullAway)
- [Infer](https://fbinfer.com/)

### 静的解析ツールの `null` に対するスタンス

Nullness を静的に解析するツールでは、アノテーションが付いていない型は Nonnull 相当のアノテーションが付いている前提で解析が行われる。
これは解析にかかるコストと実用性を考えたとき、アノテーションが付いていなければ `null` になる可能性はない、と前提を置くのは自然だと思われる。
仮にアノテーションの付いてないフィールドをすべて `null` と見做すのであれば、開発初期からアノテーションを付けるように強制してないプロジェクトでは、膨大な数のエラーに見舞われることになるだろう。

しかし、一方で現実的な話をすると最初からアノテーションが付けられているコードよりも `NullPointerException` により発生する不具合に対応するための対処療法としてアノテーションによる静的解析の導入を検討する流れの方が多いのではないだろうか。
そのような場合、Checker Framework では `@DefaultQualifier` でパッケージやクラスなどの単位でアノテーションが付いていない場合の扱いを Nonnull とするのか Nullable とするのか設定することができる。

### 比較

さて、最も気になるのは各種ツールでどれが一番優れているのかという点だろう。
一言に優れているといっても静的解析ツールを評価するときにはいくつかのポイントがある。
直ぐに思いつくだけでも、

- 実行速度
- 健全性
- 導入の容易性
- 設定の柔軟性

が挙げられる。
各ツールについてここで挙げたいくつかの点で比較した論文が ASE 2021 で採択[^On the Real-World Effectiveness of Static Bug Detectors at Finding Null Pointer Exceptions]されており、どのツールを採用すればよいか検討するときに参考になる。

[^On the Real-World Effectiveness of Static Bug Detectors at Finding Null Pointer Exceptions]:[On the Real-World Effectiveness of Static Bug Detectors at Finding Null Pointer Exceptions](https://web.cs.ucdavis.edu/~rubio/includes/ase21.pdf)

各ツールの比較は、独立した記事で詳しく書くことにする。

## JSpecify

こうした状況を受けて、静的解析で利用するアノテーションの標準仕様を決める動きがある。
それが JSpecify だ。

- [公式サイト](https://jspecify.dev/)
- [GitHub](https://github.com/jspecify/jspecify)

JSpecify では、有名な静的解析ツールやIDE、JDK、ライブラリを開発している企業が参加している。
参加企業一覧は[ここ](https://jspecify.dev/about)で確認できる。

アノテーション仕様のステータスはまだ draft 段階で、各種ツールにおいて実験的に実装を進めている状況のようだ。
現状ドキュメントが存在するアノテーションは、

- `@NullMarked`
- `@Nullable`

の二つであり、`@NullMarked` アノテーションが付与されたパッケージやクラス内では、`@Nullable` が付いている型については `null` が代入される可能性があることを表し、付いていない型は `null` が代入されないことを表すことができる。
`@Nonnull` 相当のアノテーションがないが、`@Nonnull` が提供されてしまうとコードの至るところにアノテーションを付けなければならなくなるため、同様の意味を持つアノテーションは提供されないのではないだろうか。
現時点で気になるのは、`@NullMarked` が付いてないパッケージやクラスでの扱いだ。

## 動的 null チェック

ここまで静的解析による null 検査を見てきたが、動的な null 検査についても触れておく。

### Lombok

Lombok では `null` でないこと検査するためのアノテーションとして [`@NonNull`](https://projectlombok.org/features/NonNull) が提供されている。
`@NonNull` は、クラスのフィールドやコンストラクタ、メソッドの引数で利用する。
公式のサンプルを流用して説明させてもらうと、以下のように引数に `@NonNull` を付けたとき、

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

このコードは以下のコードに変換される。

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked non-null but is null");
    }
    this.name = person.getName();
  }
}
```

アノテーションが付与された変数に対してメソッドの先頭 (言語仕様から `super` よりは後) で `null` チェックをする `if` 文が追加され、`null` の場合には `NullPointerException` を明示的に発生させる。
これは、バグの早期発見を目的とした `null` チェックで Java 7 から標準ライブラリで提供されている [`Objects#requiredNonNull`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Objects.html#requireNonNull(T)) で代用できる。
Lombok を使わない場合は、以下のように書くことで Lombok によるコード生成後のコードと同じ動作をする。

```java
import java.util.Objects;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(Person person) {
    super("Hello");
    Objects.requireNonNull(person, "person is marked non-null but is null");
    this.name = person.getName();
  }
}
```

この手の `null` チェックは「実行速度に影響が出るのではないか」という懸念があるが、`null` でない値が基本的には来ることを前提すれば、気にするほどでないようだ[^Javaでのnullチェックのパフォーマンス]。

[^Javaでのnullチェックのパフォーマンス]: [Javaでのnullチェックのパフォーマンス - きしだのHatena](https://nowokay.hatenablog.com/entry/20150425/1429935605)

`NullPointerException` が発生する割合が多い場合は、速度が大幅に低下するようだが、そのようなプログラムはそもそも書くべきではない。
また、Java SE API の至るところで `Objects#requireNonNull` は使われている。

### バリデーション

`null` に関するアノテーションと言えばもう一つあるだろう。
そう、バリデーションだ。

#### Hibernate Validator

Java のバリデーションの仕様は [Jakarta Bean Validation](https://beanvalidation.org/) によって定められており、そのリファレンス実装でありデファクトスタンダードとなっている [Hibernate Validator](https://hibernate.org/validator/) を使うのが一般的だろう。

Jakarta Bean Validation でクラスのフィールドが `null` でないか検証するときには、[`@NotNull`](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/notnull) アノテーションを利用する。
`null` であること検証するための [`@Null`](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/null) アノテーションも存在しているが、どの程度使わているのだろうか。
DB などの外部ストレージに保存するとき、グループを使って特定のパターンにおいてあるフィールドは `null` であることを保証する必要がある、といった場面では使えそうではある。
しかし、これまで見てきた `@Nullable` とは意味が大きく異なることに注意しなければならない。

## 結び

静的 null 検査、Lombok、バリデーションで使うアノテーションを一通り見た。
改めてそれぞれを表にまとめる次のようになる。

| ライブラリ                                 | Nullable          | Nonnull    |
|--------------------------------------------|-------------------|------------|
| Lombok                                     | -                 | `@NonNull` |
| Hibernate  Validator                       | `@Null` :warning: | `@NotNull` |
| JDT Annotations For Enhanced Null Analysis | `@Nullable`       | `@NonNull` |
| JetBrains Java Annotations                 | `@Nullable`       | `@NotNull` |
| Checker Framework                          | `@Nullable`       | `@Nonnull` |

こうして見ると名前が衝突しているものがある。
衝突しない組合せを考えると Checker Framework 一択となってしまうが Nonnull は使わずに Nullable だけ使うという運用の方が現実的なのでそこまで気にしなくてもいいのかもしれない。

最後に結論を書くと、実も蓋もないが導入を急がないのであれば JSpecify により**真の標準**が策定されるのを待とう。
待てない場合は、この記事で触れた点に注意して

- JDT Annotations For Enhanced Null Analysis
- JetBrains Java Annotations
- Checker Framework

から選ぶのが吉。

> みんなちがってみんないい。
> -- 金子みすゞ

## 参考URL

- [The Java Version Almanac](https://javaalmanac.io/)
- [Which @NotNull Java annotation should I use? - Stack Overflow](https://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use)
- [Goodbye JSR305, Hello JSpecify - Speaker Deck](https://speakerdeck.com/eller86/goodbye-jsr305-hello-jspecify?slide=6)
- [Avoid Check for Null Statement in Java - Baeldung](https://www.baeldung.com/java-avoid-null-check)
- [モダンなJava開発ガイド (2018年版) - Qiita](https://qiita.com/yoichiwo7/items/17190cb440ab7d253cea)
- [Java EEからJakarta EEへ - Oracle Tecknology Network Japan Blog](https://blogs.oracle.com/otnjp/post/transition-from-java-ee-to-jakarta-ee-ja)
- [Java (Eclipse) で Null 安全なプログラム - Qiita](https://qiita.com/craftone/items/883cb1bcc5ed404dc669)
- [Checker FrameworkによるJavaのNullnessの静的解析 - Qiita](https://qiita.com/draftcode/items/c8d0ba6763afc7663489)
- [JSR-305: ソフトウェア欠陥検出用アノテーション](https://www.infoq.com/jp/news/2008/07/jsr-305-update/)
